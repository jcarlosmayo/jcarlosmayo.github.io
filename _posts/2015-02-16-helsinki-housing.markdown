---
layout:     post
title:      "Helsinki rentals"
subtitle:   "Extracting and exploring housing market data"
date:       2015-02-16 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-helsinki-housing/post-helsinki-housing.jpg"
---

Housing information is a popular environment where to play with data. In Helsinki - where I live - and taking into account the cities of Espoo and 
Vantaa too, which constitute <a target ="_blank" href="https://en.wikipedia.org/wiki/Greater_Helsinki"> Finland's capital region</a>, 
there are, at any given time, a couple of thousand <a target="_blank" href="http://www.vuokraovi.com/?locale=en">rentals available</a>.

To extract their major characteristics <a target="_blank" href="https://www.kimonolabs.com/">Kimono labs</a> provides a great, 
easy to use and free tool- <a target="_blank" href="https://import.io/">import.io</a> is another good alternative. I chose to extract 
the fields price, size, description and address. The resulting data is not as clean as one would like it to be, and that is where 
some data munging with R is required - you can see how I did it 
<a target="_blank" href="https://github.com/jcarlosmayo/helsinki_housing/blob/master/clear_kimono.R">here</a>.

One of the major features missing in the original data are geo-coordinates, which are essential to explore and understand the data. 
Fortunately, the can be easily retrieved running the address values in the *geocode* function within the 
<a target="_blank" href="https://github.com/dkahle/ggmap">ggmap package</a>. The same package allows also to plot several types of maps 
in combination with latitude and longitude coordinates, however, a more interactive and flexible alternative are
<a target="_blank" href="http://leafletjs.com/">leaflet maps</a>, which can be used in R via the 
<a target="_blank" href="https://github.com/rstudio/leaflet">leaflet package</a>.

 
After retrieving and cleaning the data available on February 3rd and 12th, I put it all together in the shiny app that you 
can see below. It could be easily developed to provide live results.

<a target="_blank" href="https://jcarlosmayo.shinyapps.io/helsinki_housing_leaflet/">
<img src="{{ site.baseurl }}/img/post-helsinki-housing/post-helsinki-housing-teaser.png" />
</a>

Even though the data captures the rentals offered by several real state agents, it has been extracted from a single portal at 
only two different points in time so it definitely does not provide an overall picture of the housing market in the area. 
I have restricted the values to the central 95th percentile to remove the distortions introduce by extreme outliers, which, granted, 
might represent interesting studies in themselves. 
Bearing that in mind it is still interesting to have a look at the distribution of price and size. The vast majority of rentals correspond to 
apartments in the range of 50m<sup>2</sup> and there seems to be a clear division of prices at 1,000 EUR per month.

<img src="{{ site.baseurl }}/img/post-helsinki-housing/hist_size.png" />
<img src="{{ site.baseurl }}/img/post-helsinki-housing/hist_price.png" />

Among a good amount of competition SATO, offering about 25% of the available rentals available, seems to be a dominant player in the market - 
at this point in time at least. It is also interesting that a fair percentage of the rentals available, about 13%, is composed by small 
associations or societies and private individuals, which I packed together under the category *Other*.

<div>
    <a href="https://plot.ly/~jcarlosmayo/175/" target="_blank" title="Amount of rentals by real state agency" style="display: block; text-align: center;"><img src="https://plot.ly/~jcarlosmayo/175.png" alt="Amount of rentals by real state agency" style="max-width: 100%;width: 800px;"  width="800" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="jcarlosmayo:175" src="https://plot.ly/embed.js" async></script>
</div>

---

Find the whole R code to produce the Shinyapp and plots on <a target="_blank" href="http://github.com/jcarlosmayo/helsinki_housing/">GitHub</a>.

---

<div id="image-credit">Image: <a href="https://commons.wikimedia.org/wiki/File%3A1900_Plan_af_Helsingfors_stad.tif">Wikimedia commons - Plan af Helsingfors stad</a>.</div>
