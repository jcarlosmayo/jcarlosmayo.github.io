---
layout:     post
title:      "Send logs to Datadog from Snowflake"
subtitle:   "Using Snowflake native resources and libraries not readily available in Snowpark"
date:       2025-07-17 18:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

The [Datadog Snowflake integration](https://docs.datadoghq.com/integrations/snowflake-web/) allows you to monitor a wide range of Snowflake metrics and the documentation makes it very easy to set up. However, once you are already using Datadog and Snowflake you can create your own custom logic to follow up any other aspect of your data and its meaning by sending custom metrics to [Datadog's API](https://docs.datadoghq.com/api/latest). As an example, if you are collecting iot data in Snowflake, you might want to make sure that the sensors producing the data send data as often as they should. If each of the sensors is supposed to send data every 10 minutes you could create a query to make sure that that is the case.

Here is a basic Pyton code example to send data to Datadog using the Python library [datadog-api-client](https://pypi.org/project/datadog-api-client/). I will describe below how to run it in a Snowflake Procedure.

```python
import json
import os

from datadog_api_client import ApiClient, Configuration
from datadog_api_client.v2.api.logs_api import LogsApi
from datadog_api_client.v2.model.content_encoding import ContentEncoding
from datadog_api_client.v2.model.http_log import HTTPLog
from datadog_api_client.v2.model.http_log_item import HTTPLogItem

os.environ["DD_SITE"] = "datadoghq.eu"
os.environ["DD_API_KEY"] = "<your_datadog_api_key>"


configuration = Configuration()

# Data to be sent to logs
logs = [
    {"id": "ABC","value": 1},
    {"id": "DEF","value": 2}
]

# Convert each of the logs to a JSON-formatted string and wrap them in HTTPLogItem
log_items = [
    HTTPLogItem(
        ddsource="snowflake",
        ddtags="tag-1-key:tag-1-value,tag-1-key:tag-1-value",
        hostname="<snowflake-account-identifier>.snowflakecomputing.com",
        message=json.dumps(log),
        service="log_description",
    )
    for log in logs
]

# Pass the log_items to HTTPLog()
body = HTTPLog(log_items)

# Open the API client and send the logs
with ApiClient(configuration) as api_client:
    api_instance = LogsApi(api_client)
    response = api_instance.submit_log(content_encoding=ContentEncoding.DEFLATE, body=body)
    if response == {}:
        print(f"Successfully sent {len(log_items)} records")
```

> Bear in mind that [Datadog's API send log docs](https://docs.datadoghq.com/api/latest/logs/#send-logs) highlights the following limits. If you are likely to exceed those limitations send them in batches that maximize throughput.
>
> - Maximum content size per payload (uncompressed): 5MB
> - Maximum size for a single log: 1MB
> - Maximum array size if sending multiple logs in an array: 1000 entries

<hr>

# Snowflake Integration

These are the components needed to

- enable outgoing traffic to Datadog using a **network rule**
- store the Datadog API securely in a **secret**
- bundle the network rule and the secret in an **external access integration**
- package the [datadog-api-client](https://pypi.org/project/datadog-api-client/) library in an **internal stage**
- create a **procedure** that will run Python code making use of the external access integration and the internal stage
- create a **task** that will execute the procedure on a schedule

<p align="center">
    <img src="{{ site.baseurl }}/img/send-logs-to-datadog-from-snowflake/diagram.png" />
</p>

## Network Rule

Create a [Snowflake Network Rule](https://docs.snowflake.com/en/user-guide/network-rules) to allow outbound traffic to Datadog. Note that depending on the region your Datadog account is hosted in the address will be different. The address for a Datadog account hosted in Europe is `http-intake.logs.datadoghq.eu`.

```sql
CREATE OR REPLACE NETWORK RULE DATADOG_API
  TYPE = HOST_PORT
  VALUE_LIST = ('http-intake.logs.datadoghq.eu')
  MODE = EGRESS
  COMMENT = 'Network Rule for Datadog API.';
```

## Secret

To obtain the secret value that needs to be inserted in the [Snowflake Secret](https://docs.snowflake.com/en/sql-reference/sql/create-secret) navigate in the Datadog portal to Organization Settings >>> API Keys and create a new API Key. The value of the key is the value that needs to be passed to the secret.

```sql
CREATE OR REPLACE SECRET DATADOG_API_KEY
  TYPE = GENERIC_STRING
  SECRET_STRING = '<secret-value>'
  COMMENT = 'Datadog API key that allows sending logs to Datadog.';
```

## External Access Integration

The [Snowflake External Access Integration](https://docs.snowflake.com/en/developer-guide/external-network-access/external-network-access-overview) bundles the previously created Network Rule and Secret.

```sql
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION DATADOG_API_INTEGRATION
  ALLOWED_NETWORK_RULES = (DATADOG_API)
  ALLOWED_AUTHENTICATION_SECRETS = (DATADOG_API_KEY)
  ENABLED = true
  COMMENT = 'Access Integration to Datadog API.';
```

## Internal Stage


```SQL
CREATE STAGE LOGS_TO_DATADOG
DIRECTORY = (ENABLE = TRUE);
```

### Upload Python library to stage

The Python library [datadog-api-client](https://pypi.org/project/datadog-api-client/) is not available in Snowpark. And, as with any other Python library that is not available in [Snowflake Conda Channel](https://snowpark-python-packages.streamlit.app/) we need to upload the library to an internal stage. From where it can be made available to Snowpark functions and procedures. One way to do it is to:

1. Create a local folder where to install the Python libraries
2. Create a Python virtual environment in it
3. Activate the virtual environment
4. Install in the `datadog-api-client` library.

```
mkdir my_python_packages
cd my_python_packages
python3.12 -m venv venv
source venv/bin/activate
pip install datadog-api-client==2.35.0
```

You will notice that it installs some dependencies as well, and they are all now available under `venv/lib/python3.12/site-packages`

```
venv/lib/python3.12/site-packages
├── __pycache__
├── certifi
├── certifi-2025.7.14.dist-info
├── datadog_api_client
├── datadog_api_client-2.35.0.dist-info
├── dateutil
├── pip
├── pip-25.0.1.dist-info
├── python_dateutil-2.9.0.post0.dist-info
├── six-1.17.0.dist-info
├── six.py
├── typing_extensions-4.14.1.dist-info
├── typing_extensions.py
├── urllib3
└── urllib3-2.5.0.dist-info
```

Those are the files and folders that you need to zip and upload to an internal stage.

```
cd venv/lib/python3.12/site-packages
zip -r dependencies.zip ./*
```

Uploading them to an internal stage requires [snowsql](https://docs.snowflake.com/en/user-guide/snowsql). After connecting, the command looks like this:

```sql
PUT file:////Users/juancarlosmayo/dependencies/manual_dependencies.zip @MY_INTENAL_STAGE AUTO_COMPRESS=False;
```

Now the library can be made available to a stored procedure by referencing it in the `IMPORTS` argument, along with the arguments `EXTERNAL_ACCESS_INTEGRATIONS` and `SECRETS`, to reference the ones we created above.

```sql
USE DATABASE MY_DB;
USE SCHEMA MY_SCHEMA;

CREATE OR REPLACE PROCEDURE LOGS_TO_DATADOG()
RETURNS VARCHAR
LANGUAGE PYTHON
RUNTIME_VERSION = '3.12'
PACKAGES = ('snowflake-snowpark-python')
HANDLER = 'logs_to_datadog'
IMPORTS = ('@.MY_SCHEMA.LOGS_TO_DATADOG/dependencies.zip')
EXTERNAL_ACCESS_INTEGRATIONS = (DATADOG_API_INTEGRATION)
SECRETS = ('datadog_api_key'=DATADOG_API_KEY)
...
```

## Stored Procedure

Now, let's create the stored procedure, where we will reference all the components created so far.

```sql
CREATE OR REPLACE PROCEDURE SP_LOGS_TO_DATADOG()
RETURNS VARCHAR
LANGUAGE PYTHON
RUNTIME_VERSION = '3.11'
PACKAGES = ('snowflake-snowpark-python')
HANDLER = 'logs_to_datadog'
IMPORTS = ('@MY_DB.MY_SCHEMA.LOGS_TO_DATADOG/manual_dependecies.zip')
EXTERNAL_ACCESS_INTEGRATIONS = (DATADOG_API_INTEGRATION)
SECRETS = ('datadog_api_key'=DATADOG_API_KEY)
EXECUTE AS CALLER
AS $$
import json
import os
import _snowflake
from datadog_api_client.api_client import ApiClient
from datadog_api_client.configuration import Configuration
from datadog_api_client.v2.api.logs_api import LogsApi
from datadog_api_client.v2.model.content_encoding import ContentEncoding
from datadog_api_client.v2.model.http_log import HTTPLog
from datadog_api_client.v2.model.http_log_item import HTTPLogItem


def logs_to_datadog(session):

    # Create a query
    query = "SELECT * FROM MY_DB.MY_SCHEMA.MY_TABLE LIMIT 5;"
            
    query_results = session.sql(query).collect()

    ##########################
    # INITIALIZE DATADOG API #
    ##########################
    os.environ["DD_SITE"] = "datadoghq.eu"
    os.environ["DD_API_KEY"] = _snowflake.get_generic_secret_string("datadog_api_key")

    # Configure the Datadog API client
    configuration = Configuration()

    # Split the data in lists of 900 items max
    # Datadog supports up to 1,000
    with ApiClient(configuration) as api_client:
    
        logs = []
        for item in query_results:
            logs.append(
                {
                    "id": item[0],
                    "latest_ingested_diff_min": item[1],
                    "latest_device_diff_min": item[2],
                }
            )

        # Convert each dict to a JSON-formatted string and wrap in HTTPLogItem
        log_items = [
            HTTPLogItem(
                ddsource="snowflake",
                ddtags="account:TY25799,env:production",
                hostname="QQMTHNM-TY25799.snowflakecomputing.com",
                message=json.dumps(log),
                service="asset_id_last_available_at",
            )
            for log in logs
        ]

        body = HTTPLog(log_items)

        api_instance = LogsApi(api_client)
        response = api_instance.submit_log(content_encoding=ContentEncoding.DEFLATE, body=body)
        if response == {}:
            return f"{len(query_results)} logs sent to Datadog"
        else:
            # Implement logic to catch errors
            return "Error"

    
$$;
```

You can check that your function works correctly by executing `CALL LOGS_TO_DATADOG();`.

## Create Task

Now, you could create a task to execute the procedure as often you need. For example, every 5 minutes:

```sql
CREATE OR REPLACE TASK MY_DB.MY_SCHEMA.LOGS_TO_DATAGOG_TASK
	warehouse=MY_WH
	schedule='USING CRON */5 * * * * UTC'
	COMMENT='Send logs to Datadog.'
	as CALL SP_LOGS_TO_DATADOG();
```
