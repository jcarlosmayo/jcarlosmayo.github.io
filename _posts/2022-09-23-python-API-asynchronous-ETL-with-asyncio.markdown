---
layout:     post
title:      "API ETL process with asyncio"
subtitle:   "Optimize API ETL processes in Python using the isyncio asynchronous library"
date:       2022-09-20 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

A few months ago I was approached by a company for a position related to data analysis and data engineering, and after a couple of preliminary interviews I was asked to complete an exercise with the following tasks:

1. Extract data from a public API
2. Load it into a database
3. Make a forecast
4. Create an application providing an interface to the forecast

The amount of time available for the exercise was limited and I did not manage to complete it in full but they were nevertheless happy with my approach - *it was a pity that in the end we could not agree on the salary!*

Using Python to extract data from JSON and to load it on a database is something I am familiar with. However, I am used to an event driven approach where I do not have to worry about making calls to retrieve data. Now what I realised - and what worried me the most - was how my process to iterate over a series of API calls was so slow! To make a few hundred calls was taking minutes! I didn't have time to fix it at the time but a few days later I went back and worked on some enhancements applying an asynchronous approach using the <a target="_blank" href="https://docs.python.org/3/library/asyncio.html">asyncio</a> library. I managed to reduce the time to make the same amount of API calls **from 147.115 to 2.556 seconds!** Largely thanks to the explanations provided by <a target="_blank" href="https://www.youtube.com/watch?v=nFn4_nA_yk8&t=102s">Patrick Collins' Youtube tutorial</a>.

Additionally, although loading data into the database was not a major bottleneck, it could definitely be improved too. So, instead of looping over the data and inserting it row by row as I as making the API calls - silly me - I loaded it in bulk using the psycopg2 function `execute_values`. The time spent loading data went down **from 1.438 to 0.0632 seconds!** In this case I learned a lot reading <a target="_blank" href="https://hakibenita.com/fast-load-data-python-postgresql">Haki Benita's excellent blog post</a>. *Note* that I used this approach because I could easily hold the data in memory, otherwise possibly the best way to do it would be to dump it to a file first and load it afterwards using the `copy_from` function available also in the <a target="_blank" href="https://www.psycopg.org/docs/index.html">psycopg2</a> library.

## API Call Approaches

It took me three iterations to be satisfied with the result. The table below summarises the time spent - in seconds - making the same number of calls. As you can see, the end result is more than an order of magnitude better than the initial approach:

|                    |365 API calls |
|--------------------|-------------:|
|  for loop          | 147.115 s    |
| asyncio w/o tasks  | 41.922 s     |
| asyncio with tasks | 2.556 s      |

You can find below the basic code for the three different approaches. Since the API URL requires a date the code contains also a function to generate a series of dates that is iterated over through each API call.

### for loop
**147.115 seconds**

```python
import asyncio
import aiohttp
from datetime import date, timedelta
import time


def date_list(date1, date2):
    dates = []
    for n in range(int((date2 - date1).days) + 1):
        result_date = date1 + timedelta(n)
        dates.append(result_date.strftime("%Y-%m-%d"))
    return dates


async def retrieve_data(dates):
    async with aiohttp.ClientSession() as session:
        for day in dates:
            request_url = f"https://rata.digitraffic.fi/api/v1/trains/{day}/27"
            response = await session.get(request_url)


start_date = date(2021, 3, 23)
end_date = date(2022, 3, 23)
date_list = date_list(start_date, end_date)

asyncio.run(retrieve_data(date_list))
```

### asyncio without tasks
**41.922 seconds**

```python
import asyncio
import aiohttp
from datetime import date, timedelta


def date_list(date1, date2):
    dates = []
    for n in range(int((date2 - date1).days) + 1):
        result_date = date1 + timedelta(n)
        dates.append(result_date.strftime("%Y-%m-%d"))
    return dates


async def retrieve_data(dates):
    async with aiohttp.ClientSession() as session:
        for day in dates:
            request_url = f"https://rata.digitraffic.fi/api/v1/trains/{day}/27"
            response = await session.get(request_url)


start_date = date(2021, 3, 23)
end_date = date(2022, 3, 23)
date_list = date_list(start_date, end_date)

asyncio.run(retrieve_data(date_list))
```

### asyncio with tasks
**2.556 seconds**

```python
import asyncio
import aiohttp
from datetime import date, timedelta


def date_list(date1, date2):
    dates = []
    for n in range(int((date2 - date1).days) + 1):
        result_date = date1 + timedelta(n)
        dates.append(result_date.strftime("%Y-%m-%d"))
    return dates


def request_tasks(session, dates):
    tasks = []
    for day in dates:
        request_url = f"https://rata.digitraffic.fi/api/v1/trains/{day}/27"
        tasks.append(session.get(request_url))
    return tasks


async def retrieve_data(dates):
    async with aiohttp.ClientSession() as session:
        tasks = request_tasks(session, dates)
        responses = await asyncio.gather(*tasks)
        for response in responses:
            pass


start_date = date(2021, 3, 23)
end_date = date(2022, 3, 23)
date_list = date_list(start_date, end_date)

asyncio.run(retrieve_data(date_list))
```
