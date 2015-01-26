---
layout:     post
title:      "Bad weather costs"
subtitle:   "Consequences of extreme weather in the US"
date:       2015-01-13 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-extreme-weather.jpg"
---

Severe and extreme weather events in the form of storms, floods and tornados can result in fatalities, injuries and economic losses. 
Using data from the <a target="_blank" href="http://www.noaa.gov">US National Oceanic and Atmospheric Administration </a> - NOAA - 
the course <a target="_blank" href="http://www.coursera.org/course/repdata">Reproducible Research</a> required students to
answer two questions:

* Which types of events are most harmful with respect to population health?

* Which types of events have the greatest economic consequences?


The <a target="_blank" href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2">data</a> available from NOAA contains 
information on weather reports in the United States from 1950 to 2011. A tidy categorisation would make answering these 
questions fairly easy, instead there is a plethora of abbreviations, clarifications, typos and vague terminology that results in 985 
categories of weather events. Ranging from specific and clear values like *thunderstorm* or *flood* to vague terms like *funnel cloud* or 
*gradient wind*. For example thunderstorms can be referred to as *thunderstorms*,  *gusty thunderstorm wind* and *tstm*. 
Nevertheless, after applying regular expressions - <a target="_blank" href="http://rpubs.com/jcarlosmayo/repdata_extreme_weather_us"> 
see how I did it </a>- it is possible to reduce it to just 9 major categories, which gets us ready to answer the questions.

### Which types of events are most harmful with respect to population health?

Tornados are by far the most dangerous weather event in the US. 60,700 tornados were registered from 1950 until 2011, injuring 91,407 and 
killing 5,636 people. Tornados are also the major cause of injuries, up to 65% of the total, while the next major cause of injuries, rain, 
represents only 10%.

<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/47.embed?width=460&height=345"></iframe>
<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/43.embed?width=460&height=345"></iframe>
<div align="center" id="image-credit"><b>Note</b> that the x axes do not display the same scale.</div>


### Which types of events have the greatest economic consequences?

Floods have caused the major economic losses from 1950 to 2011. In that period 86,136 floods have been reported and while they have caused 
damages in the crops valued in 7,4 billion dollars, the result in property damages reaches 167 billion dollars.

<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/53.embed?width=460&height=345"></iframe>
<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/57.embed?width=460&height=345"></iframe>
<div align="center" id="image-credit"><b>Note</b> that the x axes do not display the same scale.</div> 
<iframe width="700" height="420" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/61.embed?width=560&height=420"></iframe>


### Weather evolution
The record is only complete for all weather events from 1993 onwards. In those years the most significant characteristic is the rapid increase 
in the number of thunderstorms, although floods also display some increase and interestingly there is an abrupt leap in cold weather in 
recent years. The large increase of "other" events may be a result of the categorization established in the data cleaning process 
but it may also reflect the fact that many more minute appreciations have been registered in recent years, such as ""red flag criteria", 
"remnants of floyd" or "waterspout funnel cloud", which fall in the category "other".

<iframe width="700" height="480" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/84.embed?width=640&height=480"></iframe>

---

See the original work on <a target="_blank" href="http://rpubs.com/jcarlosmayo/repdata_extreme_weather_us">RPubs</a> and the R code on <a target="_blank" href="http://github.com/jcarlosmayo/repdata_pa2_extreme_weather">GitHub</a>.

---

<div id="image-credit">Image: <a href="http://www.tate.org.uk/art/artworks/westall-the-commencement-of-the-deluge-n01877">TATE Gallery - The commencement of the Deluge by William Westall</a>.</div>
