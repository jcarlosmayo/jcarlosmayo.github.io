---
layout:     post
title:      "Spanish demography"
subtitle:   "A view on geographic distribution"
date:       2015-08-30 12:37:00
author:     "Juan Carlos Mayo"
header-img: "img/post-spanish-demography/bol_kirmes.png"
---


Coming from a small rural town in northwest Spain the issue of regional development has interested me for a long time, and one primary and immediate way to approach it is from the perspective of population distribution.

After the <a href="https://en.wikipedia.org/wiki/Rural_flight" target="_blank">rural flight</a> that took place during the 20<sup>th</sup> century the 21<sup>st</sup> has seen a vast majority of the population settled in urban areas and a rural landscape largely depopulated. The population divide between urban and rural can be clearly appreciated in the plot below, containing all the municipalities in peninsular Spain, the Balearic Islands and the autonomous cities of Ceuta and Melilla, in the northern coast of Africa.


<iframe width="750" height="350" frameborder="0" seamless="seamless" scrolling="no"
src="{{ site.baseurl }}/img/post-spanish-demography/leaf_map.html"></iframe>
<div id="image-credit">Spanish population by municipality provided by the <a href="http://www.ine.es/" target="_blank">Spanish Institute of Statistics</a> - Retrieved June 1<sup>st</sup> 2015.  
</div>
<div id="image-credit">
Nathan Yau provides excellent <a href="http://flowingdata.com/category/tutorials/" target="_blank">tutorials</a> to work with maps.
</div>

The map reveals that by large, most of the surface area corresponds to municipalities very scarcely populated, less than 2,500 inhabitants, while at the same time, a north-south trend becomes evident. As one moves south the municipalities tend to grow in size, although not necessarily in population density.  
An interesting unobservable feature is the fact that municipalities in the south tend to be larger and comprise a single town, while in the north a single municipality may correspond to a collection of a small number of villages, parishes and hamlets.


<div>
    <a href="https://plot.ly/~jcarlosmayo/294/" target="_blank" title="variable vs value" style="display: block; text-align: center;"><img src="https://plot.ly/~jcarlosmayo/294.png" alt="variable vs value" style="max-width: 100%;width: 932px;"  width="932" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="jcarlosmayo:294"  src="https://plot.ly/embed.js" async></script>
</div>


Given the data available it is not possible to examine the evolution over a longer period of time. Nevertheless, it appears that from 1998 to 2014 the percentage of population living in municipalities with less than 5,000 or more than 500,000 inhabitants has slowly but consistently decreased. At the same time the percentage of population living in municipalities between 5,000 and 10,000 inhabitants has only dropped slightly, while the percentage of people living in municipalities between 100,000 and 500,000 inhabitants has remained almost the same.

The rising rates appear, interestingly, at the center of the range. Municipalities between 10,000 and 100,000 inhabitants have seen increased the share of population living in them by about 2 percentage points each. By 2014 about 39% of the population of Spain lived in these middle-sized municipalities. This relocation is interesting in light of what has been termed <a href="https://en.wikipedia.org/wiki/Quality_of_life" target="_blank">quality of life</a> and its relation to population size. Small communities seem to boost the quality of life in many respects but tend to lack in others, especially services and leisure activities. Conversely, large cities can provide a wider, and perhaps better, assortment of services and a more lively environment but be detrimental in other aspects. One reason that might make middle-sized cities attractive to move in is the fact that they may provide an appealing balance in quality of life.

If you are wondering about the abrupt variation in 2007, it is likely to have been caused by the large number of missing entries in the data set that year, corresponding to <a href="https://www.dropbox.com/s/g778ylguwk95is9/pop_distribution_2007.png?dl=0" target="_blank">682 municipalities missing from the records</a>.

These changes in population distribution have not disturbed, however, the percentage of surface area covered by the different types of municipalities, which has remained almost constant through the period 1998 - 2014. It is particularly noticeable that 59% of the surface area of the country is occupied by municipalities with less than 2,500 inhabitants,  and about 71% by municipalities of less than 5,000 inhabitants. This fact signals the very low population density in most of Spain.

<div>
    <a href="https://plot.ly/~jcarlosmayo/350/" target="_blank" title="Municipality area distribution by population level in percentage points" style="display: block; text-align: center;"><img src="https://plot.ly/~jcarlosmayo/350.png" alt="Municipality area distribution by population level in percentage points" style="max-width: 100%;width: 932px;"  width="932" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="jcarlosmayo:350"  src="https://plot.ly/embed.js" async></script>
</div>


---

Find the R code on <a target="_blank" href="http://github.com/jcarlosmayo/spanish_demography">GitHub</a>.

---

<div id="image-credit">Image: <a href="https://commons.wikimedia.org/wiki/File:Hans_Bol_001.jpg">Wikimedia Commons - Bäuerliche Kirmes by Hans Bol</a>.</div>
