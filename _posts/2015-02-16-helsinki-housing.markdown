---
layout:     post
title:      "Helsinki rentals"
subtitle:   "Extracting and exploring housing market data"
date:       2015-02-16 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-helsinki-housing/post-helsinki-housing.jpg"
---

Housing information is a popular environment where to play with data. In Helsinki - where I live - there are about a thousand rentals available 
at any given time. And if one takes into account the neighbouring cities of Espoo and Vantaa, which together constitute 
<a target ="_blank" href="https://en.wikipedia.org/wiki/Greater_Helsinki"> Finland's capital region</a>, the number adds up to 
a couple thousands. These data can be easily crawled over and scraped using <a target="_blank" href="https://www.kimonolabs.com/">Kimono labs</a> 
tools.

I extracted the information related to price, size, description and address. I was especially curious to see how they are distributed 
geographically, since the original portal does not provide any map visualisation, but the geo-coordinates are missing in the original data. 
To generate them one has to first clean the data, which does not come out as neat as one would like it to be but, although slightly tedious, 
it can be easily done with R - you can see how I did it 
<a target="_blank" href="https://github.com/jcarlosmayo/helsinki_housing/blob/master/clear_kimono.R">here</a>. 
Once the address field has been cleaned, it can be used to generate the coordinates of each rental location using the *geocode* function 
within the <a target="_blank" href="https://github.com/dkahle/ggmap">ggmap package</a>. The same package allows also to plot several types of maps 
using latitude and longitude coordinates; however, a more interactive and flexible alternative are
<a target="_blank" href="http://leafletjs.com/">leaflet maps</a>, which can be used straight in R via the 
<a target="_blank" href="https://github.com/rstudio/leaflet">leaflet package</a>.

 
After extracting and cleaning the data available on February 3rd and 12th, I put it all together in the shiny app that you 
can see below, which displays static content but could be easily developed to provide live results.

<a target="_blank" href="https://jcarlosmayo.shinyapps.io/helsinki_housing_leaflet/">
<img src="{{ site.baseurl }}/img/post-helsinki-housing/post-helsinki-housing-teaser.png" />
</a>

# A brief analysis

The data captures the rentals offered by several real estate agents but it has been extracted from a single portal at 
only two different points in time so it definitely does not provide an overall picture of the housing market in the area. 
Additionally, I have restricted the values to the central 95th percentile to remove the distortions introduced by extreme outliers. 
Bearing that in mind it is still interesting to have a look at the distribution of price and size. The vast majority of rentals correspond to 
apartments in the range of 50 to 60m<sup>2</sup> and there seems to be an interesting price *barrier* at 1,000 EUR per month.

<img src="{{ site.baseurl }}/img/post-helsinki-housing/hist_size.png" />
<img src="{{ site.baseurl }}/img/post-helsinki-housing/hist_price.png" />

Regarding real estate firms, SATO seems to be a dominant player since it deals about 25% of all available rentals - 
at least at the moment when these data were scraped. It is also interesting to notice the fact that a fair percentage of the rentals available, 
about 13%, is composed by small firms, associations and societies not directly involved in real estate and private individuals, 
which I packed together under the category *Other*.

<div>
    <a href="https://plot.ly/~jcarlosmayo/175/" target="_blank" title="Amount of rentals by real estate agency" style="display: block; text-align: center;"><img src="https://plot.ly/~jcarlosmayo/175.png" alt="Amount of rentals by real estate agency" style="max-width: 100%;width: 800px;"  width="800" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="jcarlosmayo:175" src="https://plot.ly/embed.js" async></script>
</div>


This is a static view of the market. Nevertheless, using this same approach and tools it would be easy to follow up the market development by 
periodically collecting scrape results.

---

Find the whole R code on <a target="_blank" href="http://github.com/jcarlosmayo/helsinki_housing/">GitHub</a>.

---

<div id="image-credit">Image: <a href="https://commons.wikimedia.org/wiki/File%3A1900_Plan_af_Helsingfors_stad.tif">Wikimedia commons - Plan af Helsingfors stad</a>.</div>
