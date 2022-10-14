---
layout:     post
title:      "Generate and load large volumes of test data fast"
subtitle:   "Python's multiprocessing library Vs generating data directly on PostgreSQL"
date:       2022-10-14 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

Generating test data is an interesting task that poses at least two immediate questions:

**How realistic should the data be?**
Generating detailed data and distributing it in a way that resembles reality might require some effort so one might need to break the process in steps. First, generate the data, perhaps using a script to mimic complex real-life behaviour, and second load it in the database. On the other hand, if there is no need for realistic data, generation and load can be done much faster by working directly on the database.


**How much data is needed?**
This question relates directly to time constraints - i.e. *how much data do you want and have much time do you have?* Obviously, generating a few data points is much faster than generating lots of it.

---

In this case, we had a clear reporting scenario in mind that required a large volume of fairly-realistic time-series data. What we actually wanted to know was how plain [PostgreSQL](https://www.postgresql.org/) compared against [Timescale](https://www.timescale.com/) when it came to building reports based on SQL queries. I started by generating pretty realistic data in Python. Using a simple for loop at first, but soon after I realised that to generate the volume we had in mind - around 300 million records - I needed to speed up the process. Applying the [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) library the process we could generate data 5 times faster. That meant that generating around 300 million records could be done in about 30 minutes.


| Number or <br/> records |  | | | | for <br/> loop | | | | | for loop +<br/> multiprocessing |
|------------------------:|--|-|-|-|---------------:|-|-|-|-|--------------------------------:|
|1 million                |  | | | |       21.186 s | | | | |                         4.201 s |
|10 million               |  | | | |      206.069 s | | | | |                        42.458 s |
|100 million              |  | | | |    1,891.245 s | | | | |                       593.367 s |

The second step was to load the data, in this case on a PostgreSQL instance running locally on Docker, and these were the results:

| Number of records | | | | | Seconds to load |
|------------------:|-|-|-|-|----------------:|
|10, 000            | | | | |           0.182 |
|100, 000           | | | | |           1.691 |
|1,000, 000         | | | | |          29.102 |
|10,000,000*        | | | | |       3,559.772 |

\* *The data was loaded in batches of CSV files containing 1 million records each. There seems to be a sweet spot in the batch size that enhances performance and allows to load data faster. Files containing 1 million records are way over already, I suspect something in the 100K rows is probably closer to it. That is something worth looking into to streamline the load.*

So, the speed gained by introducing the *multiprocessing* library is wasted by the speed at which we can load the data into PostgreSQL. On my local test setup it seems that it can pretty much keep up with the speed at which the simple *for loop* can generate it. Extrapolating from the results above it would take 30 hours of simultaneous data generation and  load to get to 300 million records.

So, I dropped the idea of fairly-realistic data and moved onto, kind-of realistic data. The kind-of realistic data that I could generate directly on the database. Combining the `generate_series` function with a `CROSS JOIN` you can generate and insert data in a table extremely fast, and since I was interested in volume this is the option I went for.


| Number of records | | | | | Seconds to load |
|------------------:|-|-|-|-|----------------:|
|10,158            | | | | |            0.633 |
|101,646           | | | | |            6.343 |
|1,015,570         | | | | |           53.634 |
|11,748,066        | | | | |              229 |

# Python multiprocessing

This is a simplified - and crude - version of the Python code I used to generate data. The function below - `generate_row_data` - takes a number from 1 to 31 and generates data that populates a tuple with randomly generated:
- event-id
- event-timestamp
- product
- product-type

As you can see part of the data is hardcoded - May 2022 - the reason is that we wanted to stick to a single month.

```python
import uuid
from random import randint
from random import choices


def generate_row_data(day: int) -> tuple:

    # event-id

    event_id = str(uuid.uuid4())

    # event-timestamp

    day = randint(1, 30)
    if day < 10:
        day = f'0{day}'

    else:
        day = str(day)

    hour = randint(0, 23)
    if hour < 10:
        hour = f'0{hour}'

    minute = randint(0, 59)
    if minute < 10:
        minute = f'0{minute}'

    second = randint(0, 59)
    if second < 10:
        second = f'0{second}'

    millisecond = randint(0, 999)

    if millisecond < 10:
        millisecond = f'00{millisecond}'
    elif millisecond < 100:
        millisecond = f'0{millisecond}'

    event_timestamp = f'2022-05-{day} {hour}:{minute}:{second}.{millisecond}'

    # product

    #   - product-a

    #   - product-b

    #   - product-c

    products_list = ['product-a', 'product-b', 'product-c']
    chosen_product = choices(products_list,
                             weights=[1, 1, 2],
                             k=1)
    product = chosen_product[0]

    # product-type

    # - red

    # - blue

    # - yellow

    product_type_list = ['red', 'blue', 'yellow']
    chosen_product_type = choices(product_type_list,
                                  weights=[1, 2, 2],
                                  k=1)
    product_type = chosen_product_type[0]

    # Row to insert

    row_values = (event_id,
                  event_timestamp,
                  product,
                  product_type)

    return row_values
```

You can then wrap the previous function around the one below - `generate_daily_data` - which is basically a for loop generate rows according to the *number_of_records* parameter and which the function will write into a separate CSV file for each date under the `test-data/` folder:


```python
def generate_daily_data(number_of_records: int, day_number: int):

    # Initialise list where to append tuples

    insert_list = []

    # Run loop x amount of times to generate x data rows for insertion

    for _ in range(0, number_of_records, 1):
        row = generate_row_data(day_number)
        insert_list.append(row)

    # Write list to csv file

    if day_number < 10:
        day_number = f'0{day_number}'

    else:
        day_number = str(day_number)
    file_name = f'202205{day_number}-test-data.csv'
    with open(f'test-data/{file_name}', 'w') as f:
        writer = csv.writer(f, delimiter=',', lineterminator='\n')
        writer.writerows(insert_list)
```

To go through a series of dates now all you have to do is create a for loop and place in it the `generate_daily_data` function. For example, below you can see how to create ten days worth of data with 1 million records each:
```python
for i in range(1, 11):
    generate_daily_data(1000000, i)
```

To speed up the process using the `multiprocessing` library you need to introduce a couple of loops that split process into tasks that are fed separately to each of your machine processors and which will be joined together once they are done.
```python
import multiprocessing

if __name__ == '__main__':

    processes = []

    for i in range(1, 11):
        p = multiprocessing.Process(target=generate_daily_data, args=(10000000, i))
        p.start()
        processes.append(p)

    for process in processes:
        process.join()
```
## Load the data into PostgreSQL

The chunk below loops over the CSV files generated above, available on the `test-data/` folder, and uses the `psycopg2` library to load them into the database.

```python
import psycopg2
import os

try:
    db_conn = psycopg2.connect(
            dbname='<your-database-name>',
            host='<your-database-host>',
            port='<your-database-port>',
            user='<your-username>',
            password='<your-password>',
    )
except Exception as e:
    print('Connection to postgreSQL failed')
    raise e

all_files = os.listdir("test-data")
csv_files = list(filter(lambda f: f.endswith('.csv'), all_files))

for file in csv_files:
    with db_conn.cursor() as db_cursor:
        try:
            with open(f'test-data/{file}') as f:
                db_cursor.copy_from(f, 'test_table',
                                    columns=('event_id', 'event_timestamp', 'product', 'product_type'),
                                    sep=',')

        except (Exception, psycopg2.DatabaseError) as error:
            db_conn.rollback()
            db_cursor.close()
            print("Rolled backed error: {0}".format(error))
            raise

    db_conn.commit()

db_conn.close()
```

# Working directly on PostgreSQL

Generating data directly on the database is a simple process that just requires a couple of steps, and if you are a SQL expert I'm sure it is possible to introduce some pretty smart logic to make the data pretty realistic as well.

First, let's create a table and insert into it the minimum amount of permutations we want to have in the data.
```sql
CREATE TABLE if not exists stg_table (
	product text,
	product_type text
);

insert into stg_table (
	product,
	product_type)
values
	('product-a', 'yellow'),
	('product-a', 'red'),
	('product-a', 'blue'),
	('product-b', 'yellow'),
	('product-b', 'red'),
	('product-c', 'blue');
```

Second, let's create the final table that will hold the test data. Note that I'm calling the `uuid-ossp` extension in order to have uuids on the *event_id* field, which will be autogenerated.
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE if not exists test_table (
	event_timestamp timestamp,
	event_id uuid default uuid_generate_v4(),
	product text,
	product_type text,
	primary key (event_id, event_timestamp)
);
```

Last, we can combine the SQL function `generate_series` with a `CROSS JOIN` to our `stg_table` to generate and insert data on our `test_table`. The code below will generate 518,046 records - in roughly 22 seconds - dated on the first of May 2022, one record every second. Modifying the date range or the time between events you can generate different loads.
```sql
insert into test_table (
	event_timestamp,
	product,
	product_type
)
select
	event_timestamp,
	product,
	product_type
from (
	select *
	FROM generate_series(
	    '2022-05-01 00:00'::timestamp,
    	'2022-05-01 23:59',
    	'1 second') event_timestamp
	cross join stg_table
) as t;
```
