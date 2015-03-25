---
layout:     post
title:      "Bad weather costs"
subtitle:   "Consequences of extreme weather in the US"
date:       2015-01-13 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-extreme-weather.jpg"
---

Severe and extreme weather events in the form of storms, floods and tornados can result in fatalities, injuries and economic losses.
It is possible to explore their consequences using the
<a target="_blank" href="http://www.noaa.gov">US National Oceanic and Atmospheric Administration </a> - NOAA - detailed weather reports,
which are publicly available and cover the period from 1950 to 2011. The
<a target="_blank" href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2">data</a>
contains a great a deal of detail;
so much so that instead of a neat categorisation of weather events  - e.g. *flood*, *tornado*... - there is a plethora of 985
categories consisting of:

* **abbreviations**, like *tstm* instead of *thunderstorm*,
* **clarifications**, like *gusty wind and rain*,
* **typos**, like *torndao* instead of *tornado*
* and **vague terminology**, like *red flag criteria*


Which can, nonetheless, be sorted using regular expressions -
<a target="_blank" href="http://rpubs.com/jcarlosmayo/repdata_extreme_weather_us">see how I did it </a>-
to obtain 9 major categories:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;margin:0px auto;}
.tg td{ font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;}
.tg .tg-s6z2{text-align:center}
</style>
<table class="tg">
  <tr>
    <td class="tg-s6z2">Hot weather<br></td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Rain</td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Wind</td>
  </tr>
  <tr>
    <td class="tg-s6z2">Cold weather<br></td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Flood</td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Tornado</td>
  </tr>
  <tr>
    <td class="tg-s6z2">Thunderstorm</td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Fire</td>
    <td class="tg-031e"></td>
    <td class="tg-s6z2">Other</td>
  </tr>
</table>


After which we are ready to provide some answers on their consequences.

### Which types of events are most harmful with respect to population health?

Tornados have been by far the most dangerous weather event in the US. 60,700 tornados were registered from 1950 until 2011, injuring 91,407 and
killing 5,636 people. Tornados were also the major cause of injuries, up to 65% of the total, while the next major cause of injuries, rain,
represents only 10%.

<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/47.embed?width=460&height=345"></iframe>
<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/43.embed?width=460&height=345"></iframe>
<div align="center" id="image-credit"><b>Note</b> that the x axes do not display the same scale.</div>


### Which types of events have the greatest economic consequences?

Floods have been the major cause of economic losses from 1950 to 2011. In that period 86,136 floods were reported and while they caused
damages in the crops valued in 7,4 billion dollars, the resulted in property damages reaching 167 billion dollars.

<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/53.embed?width=460&height=345"></iframe>
<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/57.embed?width=460&height=345"></iframe>
<div align="center" id="image-credit"><b>Note</b> that the x axes do not display the same scale.</div>
<iframe width="700" height="420" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/61.embed?width=560&height=420"></iframe>


### Weather evolution
The record is only complete for all weather events from 1993 onwards. In those years the most significant characteristic is the rapid increase
in the number of thunderstorms, although floods also display some increase and interestingly there is an abrupt leap in cold weather in
recent years. The large increase of "other" events may be a result of the categorization established in the data cleaning process
but it may also reflect the fact that many more minute appreciations have been registered in recent years, such as ""red flag criteria",
"remnants of floyd" or "waterspout funnel cloud", which do not provide very specific information and fell in the category *other*.

<iframe width="700" height="480" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/105.embed?width=6400&height=480"></iframe>

---

The <a target="_blank" href="http://rpubs.com/jcarlosmayo/repdata_extreme_weather_us">original study </a>was carried out for the course
<a target="_blank" href="http://www.coursera.org/course/repdata">Reproducible Research</a>. You may also find the R code on <a target="_blank" href="http://github.com/jcarlosmayo/repdata_pa2_extreme_weather">GitHub</a>.

---

<div id="image-credit">Image: <a href="http://www.tate.org.uk/art/artworks/westall-the-commencement-of-the-deluge-n01877">TATE Gallery - The commencement of the Deluge by William Westall</a>.</div>
