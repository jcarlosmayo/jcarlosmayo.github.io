---
layout:     post
title:      "Religiosity & homosexual (in)tolerance"
subtitle:   "An independence study on European sentiment"
date:       2014-11-06 12:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-religiosity.jpg"
---

**Is there a relationship between how religious a person is and his or her views towards homosexuality?** The answer may seem obvious to many, 
yet it is better to test what one may suspect. The <a target="_blank" href="http://www.europeansocialsurvey.org">European Social Survey</a> 
provides a fairly good amount of information on the beliefs and behaviour of Europeans across nations, among which one can find the answer to:

* How religious are you? A categorical variable coded from 0, not religious at all, to 10, very religious.
* Gays and lesbians free to live life as they wish. A categorical variable coded from 1, strongly agree, to 5, strongly disagree.  


While responses vary greatly across countries I focused on the aggregated data:

<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/22.embed?width=460&height=345"></iframe>
<iframe width="350" height="345" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/23.embed?width=460&height=345"></iframe>

The distributions of both answers provide a good first overview on the beliefs of Europeans as a whole. The religious belief of the respondents peaks at 'not religious at all' and 'moderately religious', but they are otherwise quite evenly distributed. On the other hand, the answers of respondents to the statement 'gays and lesbians free to live life as they wish' are strongly skewed; most of the respondents agreeing with that statement.

Before running an independence test I created a contingency table to plot the percentage of respondents of one variable in terms of the other. Visually, a pattern emerges. Respondents who stated being 'not religious at all' agree the most with that statement. Vice versa, 'very religious' respondents disagree the most with that statement. This trend already suggests that a relationship exists.

<iframe width="700" height="420" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~jcarlosmayo/25.embed?width=560&height=420"></iframe>


To test the independence of the variables I ran a <a target="_blank" href="https://en.wikipedia.org/wiki/Pearson%27s_chi-squared_test">chi-squared test </a> that resulted in a **p-value of 2.2e-16 using a 1% significance level**, which indicates that the **variables are indeed not independent**. Respondents who agree with the statement 'gays and lesbians free to live life as they wish' tend to be less religious or not religious at all, while respondents who disagree with that statement tend to be religious or very religious. **Note**, however, that no causality can be infered from this relationship.

---

The <a target="_blank" href="http://rpubs.com/jcarlosmayo/dasi_project">original study </a>was carried out for the course 
<a target="_blank" href="https://www.coursera.org/course/statistics">Data Analysis and Statistical Inference</a>. You may also find the whole R code on <a target="_blank" href="https://github.com/jcarlosmayo/dasi_project">GitHub</a>.

---


<div id="image-credit">Image: <a href="http://www.tate.org.uk/art/artworks/adams-rainbow-painting-i-t01127">TATE - Rainbow painting (I) by Norman Adams</a>.</div>
