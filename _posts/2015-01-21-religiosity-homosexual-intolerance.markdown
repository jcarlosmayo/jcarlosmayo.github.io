---
layout:     post
title:      "Religiosity & homosexual (in)tolerance"
subtitle:   "An independence study on European sentiment"
date:       2014-11-06 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-religiosity.jpg"
---

For the final project of [Data Analysis and Statistical Inference](https://www.coursera.org/course/statistics) I tested the existence of a relationship between religious beliefs and views towards homosexuality using the publicly available data from the [European Social Survey](http://www.europeansocialsurvey.org/) of 2012. Their data provides a fair amount of information on the beliefs and behaviour of many European nations, among which one can find the answer to the following questions:

* Howe religious are you? A categorical variable coded from 0, not religious at all, to 10, very religious.
* Gays and lesbians free to live life as they wish. A categorical variable coded from 1, strongly agree, to 5, strongly disagree.  


<p>While <a target="_blank" href="http://jcarlosmayo.shinyapps.io/ESS_2012">responses vary greatly across countries</a>, I focused on the aggregated data:</p>

<table>
	<tr>
    	<td style="background-color:white"><iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/22.embed?width=460&height=345"></iframe></td>
		<td style="background-color:white"><iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/23.embed?width=460&height=345"></iframe></td>
    </tr>
</table>

The distributions of both answers provide a good first overview on the beliefs of Europeans as a whole. The religious belief of the respondents peaks at 'not religious at all' and 'moderately religious', but they are otherwise quite evenly distributed. On the other hand, the answers of respondents to the statement 'gays and lesbians free to live life as they wish' are strongly skewed; most of the respondents agreeing with that statement.

Before running an independence test I created a contingency table to plot the percentage of respondents of one variable in terms of the other. The horizontal axis shows the degree of religiosity, from not religious at all, 1, to very religious 10. The vertical axis shows the percentage of responses to the question 'gays and lesbians free to live life as they wish' for each degree of religiosity. Visually, a pattern emerges. Respondents who stated being 'not religious at all' agree the most with that statement. Vice versa, 'very religious' respondents disagreed the most with that statement. This trend suggests that a relationship exists.

<iframe width="700" height="420" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/25.embed?width=560&height=420"></iframe>


To test the independence of the variables I ran a <a target="_blank" href="https://en.wikipedia.org/wiki/Pearson%27s_chi-squared_test">chi-squared test </a> that resulted in a **p-value of 2.2e-16 using a 1% significance level**, which indicates that **the results in the two variables are not independent**. Respondents who agree with the statement 'gays and lesbians free to live life as they wish' tend to be less religious or not religious at all, while respondents who disagree with that statement tend to be religious or very religious. **Note**, however, that no causality can be infered from this relationship.

---

See the original work in <a target="_blank" href="http://rpubs.com/jcarlosmayo/dasi_project">RPubs</a> and the R code in <a target="_blank" href="https://github.com/jcarlosmayo/dasi_project">GitHub</a>.

*Image: <a href="https://en.wikipedia.org/wiki/Rainbows_in_culture">Wikipedia / Rainbows in culture - Noah's Thank Offering by Joseph Anton Koch</a>.*
