---
layout:     post
title:      "Automating Snowflake Network Policies for Azure Services"
subtitle:   "A step‑by‑step guide to fetch Azure’s weekly IP list, load it into Snowflake, and keep network rules in sync automatically"
date:       2025-10-01 18:00:00 +00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

This article extends the approach described in [Automate Power BI allowlisting in Snowflake](https://medium.com/@daria.rostovtseva/how-to-automate-power-bi-ip-allowlisting-in-snowflake-4cfa8e1769e9) and shows you how to:


1. download the latest IP ranges used by Azure's services and make them available through a view.

2. create a generic functionality that uses the above to create or update Snowflake network rules for any (public) Azure service.

# Download the latest IP ranges used by Azure's services

Each week Azure publishes a JSON file listing the [public IP ranges for all its services]((https://www.microsoft.com/en-us/download/details.aspx?id=56519)). By ingesting this file you can identify which Azure services need access to Snowflake and grant that access via Snowflake network rules. If you use Azure in combination with Snowflake you must keep the file up-to-date to avoid processes losing access to data or data pipelines breaking. One example is Snowpipe, which require Azure's Eventgrid access to Snowflake to notify about the arrival of new files. If Eventgrid loses access to Snowflake the automatic data ingestion pipeline breaks and manual intervention is needed to trace the missing data and load it manually.

The approach to keep up with this file can be sketched out like this:

<p align="center">
    <img src="{{ site.baseurl }}/img/2025/10/download-azure-services-ip-ranges-file.png" />
</p>

## 1. Allow Snowflake to access Microsoft's download page

In order to download the file containing the latest IP ranges used by Azure's services, you need to allow Snowflake access to [Microsoft's download page](https://www.microsoft.com/en-us/download/details.aspx?id=56519).

To do that, first, you need to create an egress [network rule](https://docs.snowflake.com/en/sql-reference/sql/create-network-rule) pointing to the Microsoft's website addresses needed to reach and download the file. And, second, create an [external access integration](https://docs.snowflake.com/en/sql-reference/sql/create-external-access-integration) that will use those network rules to allow Snowflake outgoing traffic.

```sql
CREATE NETWORK RULE MY_DB.MY_SCHEMA.NETWORK_RULE_MICROSOFT
MODE = EGRESS
TYPE = HOST_PORT
VALUE_LIST = (
  'www.microsoft.com',
  'download.microsoft.com'
);

CREATE EXTERNAL ACCESS INTEGRATION MICROSOFT_ACCESS
ALLOWED_NETWORK_RULES = ('NETWORK_RULE_MICROSOFT')
ENABLED = true;
```

## 2. File Format and External Stage to store the JSON file

Let's now [create a File Format](https://docs.snowflake.com/en/sql-reference/sql/create-file-format) and an [External Stage linked to an Azure blob storage container](https://docs.snowflake.com/en/user-guide/data-load-azure-create-stage). **Note**: `MY_INT` must be a Snowflake storage integration that points to an Azure Blob container with write permissions. The File Format will allow Snowflake to read in the contents of the JSON file, and the stage will store the file itself.

```sql
CREATE OR REPLACE FILE FORMAT JSON_FMT
TYPE = JSON
COMPRESSION = GZIP
BINARY_FORMAT = UTF8;

CREATE STAGE MY_STAGE
STORAGE_INTEGRATION = MY_INT
URL = 'azure://<myaccount>.blob.core.windows.net/<mycontainer>/azure-ip-ranges-and-service-tags/'
FILE_FORMAT = JSON_FMT;
```

## 3. Stored Procedure to download the file

```sql
CREATE OR REPLACE PROCEDURE DOWNLOAD_AZURE_PUBLIC_IP_RANGES()
RETURNS VARCHAR
LANGUAGE PYTHON
RUNTIME_VERSION = '3.12'
PACKAGES = ('snowflake-snowpark-python', 'requests', 'urllib3')
HANDLER = 'main'
EXTERNAL_ACCESS_INTEGRATIONS = (MICROSOFT_ACCESS)
EXECUTE AS CALLER
AS $$
import re
from io import BytesIO

import requests
import snowflake.snowpark as snowpark
import urllib3


def main(session: snowpark.Session) -> str:
    """
    Main entry point for downloading and uploading the Azure Service Tags JSON file.

    Args:
        session (snowpark.Session): Active Snowflake session used for stage operations.

    Returns:
        str: A confirmation message with the filename and stage it was uploaded to.

    Raises:
        ValueError: If the JSON file URL cannot be extracted from the Microsoft page.
        RuntimeError: If the upload to Snowflake fails.
    """

    # Step 1: Request the Microsoft Service Tags download page
    url = "https://www.microsoft.com/en-us/download/details.aspx?id=56519"
    http = urllib3.PoolManager()
    response = http.request("GET", url)
    data = response.data.decode("utf-8")

    # Step 2: Find the actual Service Tags JSON download link inside the page
    start = "https://download.microsoft.com/"
    end = ".json"
    parts = data.split(start)[1:]
    for part in parts:
        if end in part:
            furl = start + part.split(end)[0] + end
            break
    else:
        # If no JSON link is found, raise an error
        raise ValueError(f"URL {url} does not contain a part ending with '{end}'")

    # Step 3: Download the Service Tags JSON file
    response = requests.get(furl, stream=True, timeout=30)
    response.raise_for_status()

    # Step 4: Construct the Snowflake stage name
    stage_name = f"MY_DB.MY_SCHEMA.MY_STAGE"

    # Step 5: Extract JSON filename from the URL (last part of the URL path)
    match = re.search(r"[^/]+\.json$", furl)
    if match:
        file_name = match.group(0)
    else:
        raise ValueError("No JSON filename found in the url")

    # Step 6: Upload the file content to the Snowflake stage
    try:
        # Convert file content into a byte stream for uploading
        stream = BytesIO(response.content)
        session.file.put_stream(stream, f"@{stage_name}/{file_name}")

    except Exception as e:
        # Raise error with more context if upload fails
        raise RuntimeError(f"Failed to upload {file_name} to stage {stage_name}: {e}") from e

    # Step 7: Return success confirmation
    return f"Successfully downloaded file {file_name} into stage {stage_name}"
$$;
```

## 4. Task to trigger the Stored Procedure every week

Microsoft does not provide details about when the file is updated, but it seems to happen weekly every Monday. The website also mentions that '_New ranges appearing in the file will not be used in Azure for at least one week_'. So, it seems safe to create a Task that will download the file every Tuesday.

```sql
CREATE TASK DOWNLOAD_AZURE_PUBLIC_IP_RANGES_WEEKLY
  WAREHOUSE = MY_WH
  SCHEDULE = "USING CRON 0 1 * * TUE UTC"   -- 01:00 UTC every Tuesday
  AS
    CALL DOWNLOAD_AZURE_PUBLIC_IP_RANGES();
```

## 5. Table to store the file contents

Next let's create the table that will store the contents of the file. The reason why it is named `_RAW` is because it will contain the contents of the file as ingested by the Snowpipe - that we will create in the next step - and we are not able to apply a lateral flatten on the contents of the field `values`, which contains the actual list of Azure services and their public IPs. The lateral flatten that will expose its contents in an structured manner will be done in the last step, the creation of the view that exposes the latest values.

```sql
CREATE TABLE AZURE_PUBLIC_IP_RANGES_RAW (
    SOURCE VARCHAR(16777216),
    PUBLICATION_DATE DATE,
    SOURCE_FILE_CHANGE_NUMBER NUMBER(38,0),
    PAYLOAD VARIANT,
    INGESTED_AT TIMESTAMP_NTZ(9)
);
```

## 6. Snowpipe to copy the file contents to the table

The following Snowpipe will receive a notification from Eventgrid whenever a new file lands in the stage and that will trigger the command to copy the contents of the file to the table. _**Note**, that as with the creation of the External Stage, creating a Snowpipe requires a [storage integration](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)._

```sql
CREATE PIPE AZURE_PUBLIC_IP_RANGES_PIPE
AUTO_INGEST = TRUE
INTEGRATION = 'MY_INT'
AS
COPY INTO MY_DB.MY_SCHEMA.AZURE_PUBLIC_IP_RANGES_RAW
    FROM (
    SELECT
        metadata$filename,
        TO_DATE(REGEXP_SUBSTR(metadata$filename, '\\d{8}'), 'YYYYMMDD'),
        $1:changeNumber::NUMBER,
        $1:values,
        current_timestamp()::timestamp_ntz(9) AS INGESTED_AT
    FROM @MY_DB.MY_SCHEMA.MY_STAGE
    (FILE_FORMAT => 'MY_DB.MY_SCHEMA.JSON_FMT'));
```


## 7. View to display the latest values

Lastly, we create a view that will expose the values available in the latest available file, and display them in a structured manner thanks to the `LATERAL FLATTEN` that unpacks the JSON array stored in the column `PAYLOAD`.

```sql
CREATE VIEW AZURE_PUBLIC_IP_RANGES_LATEST_V
  AS SELECT
    t.SOURCE AS SOURCE,
    t.PUBLICATION_DATE AS PUBLICATION_DATE,
    t.SOURCE_FILE_CHANGE_NUMBER AS SOURCE_FILE_CHANGE_NUMBER,
    f.value:id::VARCHAR AS ID,
    f.value:name::VARCHAR AS NAME,
    f.value:properties.addressPrefixes::ARRAY AS ADDRESS_PREFIXES,
    f.value:properties.changeNumber::NUMBER AS CHANGE_NUMBER,
    f.value:properties.networkFeatures::ARRAY AS NETWORK_FEATURES,
    f.value:properties.platform::VARCHAR AS PLATFORM,
    f.value:properties.region::VARCHAR AS REGION,
    f.value:properties.regionId::NUMBER AS REGION_ID,
    f.value:properties.systemService::VARCHAR AS SYSTEM_SERVICE,
    t.INGESTED_AT AS INGESTED_AT
FROM AZURE_PUBLIC_IP_RANGES_RAW AS t,
    LATERAL FLATTEN(input => t.PAYLOAD) f
WHERE PUBLICATION_DATE = (
    SELECT MAX(PUBLICATION_DATE)
    FROM AZURE_PUBLIC_IP_RANGES_RAW
    );
```

# Create and update Snowflake network rules to allow access to the Azure services

## Stored Procedure to create and update network rules

Now, we can create a Stored Procedure to create network rules for any of the Azure public IP services. The Stored Procedure is defined in a Python function that will query the view we defined earlier and create or update a network rule for the service we pass to it.

```sql
CREATE OR REPLACE PROCEDURE UPDATE_AZURE_SERVICES_NETWORK_RULES(service_name VARCHAR)
RETURNS VARCHAR
LANGUAGE PYTHON
RUNTIME_VERSION = '3.12'
PACKAGES = ('snowflake-snowpark-python')
HANDLER = 'main'
EXECUTE AS CALLER
AS $$
import ipaddress
import json
from typing import List

import snowflake.snowpark as snowpark


def main(session: snowpark.Session, service_name: str) -> str:
    """
    Creates or updates a Snowflake network rule for a given Azure service.

    Parameters
    ----------
    session : snowpark.Session
        Active Snowflake Snowpark session used to execute SQL queries.
    service_name : str
        The Azure service name whose IP prefixes will be used
        to create/update the network rule.

    Returns
    -------
    str
        Success message indicating that the network rule was created or updated.

    Raises
    ------
    ValueError
        If no results are found for the given service name,
        or if no valid IPv4 addresses are parsed.
    RuntimeError
        If any SQL execution or JSON parsing fails.
    """

    try:
        # Query the view to get the raw JSON list of address prefixes
        query_address_prefixes = f"""
        SELECT ADDRESS_PREFIXES
        FROM AZURE_PUBLIC_IP_RANGES_LATEST_V
        WHERE NAME = '{service_name}';
        """

        df = session.sql(query_address_prefixes).collect()

        if not df:
            raise ValueError(f"No results found for service name: {service_name}")

        # Explicit cast to str
        raw_value = str(df[0][0])

        try:
            address_prefixes_list: List[str] = json.loads(raw_value)
        except json.JSONDecodeError as e:
            raise RuntimeError(f"Failed to parse JSON for {service_name}: {e}") from e

        # Filter for valid IPv4 CIDR ranges only
        ipv4_address_prefixes = []
        for cidr in address_prefixes_list:
            try:
                net = ipaddress.ip_network(cidr, strict=False)
                if isinstance(net, ipaddress.IPv4Network):
                    ipv4_address_prefixes.append(cidr)
            except ValueError:
                # Skip invalid CIDR values
                continue

        if not ipv4_address_prefixes:
            raise ValueError(f"No valid IPv4 address prefixes found for {service_name}")

        # Format as a Snowflake value list string
        ipv4_address_prefixes_str = (
            "(" + ",".join(f'"{item}"' for item in ipv4_address_prefixes) + ")"
        )

        # Network rule name (Snowflake identifiers cannot contain ".")
        rule_name = service_name.upper().replace(".", "_")

        # Build SQL queries
        create_nr_query = f"""
        CREATE NETWORK RULE IF NOT EXISTS {rule_name}
        MODE=INGRESS TYPE=IPV4
        VALUE_LIST = {ipv4_address_prefixes_str};
        """

        alter_nr_query = f"""
        ALTER NETWORK RULE {rule_name}
        SET VALUE_LIST = {ipv4_address_prefixes_str};
        """

        # Execute both create and update queries
        try:
            session.sql(create_nr_query).collect()
            session.sql(alter_nr_query).collect()
        except Exception as e:
            raise RuntimeError(f"Failed to create/update network rule {rule_name}: {e}") from e

        return f"Successfully created/updated network rule {rule_name}"

    except Exception as e:
        # Catch-all for unexpected errors
        raise RuntimeError(f"Error processing network rule for {service_name}: {e}") from e
$$;
```

### Calling the Stored Procedure

It is important to note that when calling the Stored Procedure we must pass the Azure service name as it appears in the JSON file we have downloaded. You can inspect the file itself or the column `NAME` in the view. For example, if you would like to create a network rule for Eventgrid regardless of the region, you would run:

```sql
CALL UPDATE_AZURE_SERVICES_NETWORK_RULES('AzureEventGrid');
```

This will create the network rule `AZUREEVENTGRID`.

It is possible to also narrow it down to a specific region because the JSON file also contains separate instances where the name includes the region it refers to. So you could create a network rule for the Eventgrid addresses located in Western Europe by executing:

```sql
CALL UPDATE_AZURE_SERVICES_NETWORK_RULES('AzureEventGrid.WestEurope');
```

This will create the network rule `AZUREEVENTGRID_WESTEUROPE`.

## Task to update the network rules every week

Since the file is published every week, and we defined a Task to retrieve it every Tuesday at 01:00 UTC, as well as create a Task to update the network rule following that. It is not advisable to chain the Task to the execution of the one downloading the file because there might be a delay in the process of downloading the file and then Snowpipe ingesting it. We can be generous and give it a few minutes. Alternatively, you could expand the logic that we have gone through so far to include checking if the IP addresses for a given service have changed or not. This easily verified by looking at the field `changeNumber` in the JSON file. Some, like `AzureEventGrid.WestEurope` have been updated only twice.



```sql
CREATE TASK UPDATE_AZURE_NETWORK_RULES_WEEKLY
  WAREHOUSE = MY_WH
  SCHEDULE = "USING CRON 5 1 * * TUE UTC"   -- 01:05 UTC every Tuesday
  AS
    CALL UPDATE_AZURE_SERVICES_NETWORK_RULES('AzureEventGrid.WestEurope');
```

## Enforce the network rules through a network policy

After you’ve created the necessary network rules, you can define one or more network policies that grant Snowflake access to the corresponding Azure services. Following the previous example, we can allow Eventgrid notifications from Western Europe by:

```sql
CREATE NETWORK POLICY ALLOW_AZURE_SERVICES
  ALLOWED_NETWORK_RULE_LIST = ('AZUREEVENTGRID_WESTEUROPE');
```

Because the policy references the rule names, any future updates to the rules are automatically reflected; you only need a new Stored Procedure if you want to modify the policy itself.


### Activate the network policy

The final step is to activate the network policy so it takes effect.  
Snowflake offers three ways to attach a policy:

1. **Account‑level** – `ALTER ACCOUNT SET NETWORK_POLICY = <policy_name>;`  
2. **User‑level** – `ALTER USER <user> SET NETWORK_POLICY = <policy_name>;`  
3. **Role‑level** – `ALTER ROLE <role> SET NETWORK_POLICY = <policy_name>;`

Refer to Snowflake’s documentation on [activating a network policy](https://docs.snowflake.com/en/user-guide/network-policies#activating-a-network-policy) to choose the method that best fits your requirements.