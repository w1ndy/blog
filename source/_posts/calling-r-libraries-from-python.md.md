---
title: Calling R Libraries from Python
tags:
  - python
  - r
id: 32
categories:
  - Python
date: 2015-10-23 14:58:47
---

A few statistical algorithms like IsolationForest, an efficient outlier detection algorithm, are implemented in R language only. R is a fantastic language for statisticians and mathematicians, but learning a totally new language would be a huge cost for just using a simple functionality in the libraries designed specifically for that language.

[rpy2](https://pypi.python.org/pypi/rpy2) is a R to python interface which allows users conveniently call R functions and methods from python, as well as exploiting existing R modules. Some transitions are required basically due to different data types used in two languages. Example given here will be demonstrated based on IsolationForest library.
<!-- more -->

**Step 1\. Import Library**

First, you should install corresponding R library in your computer as well as R itself, if you haven't yet. Some packages can be found on [r-forge](http://r-forge.r-project.org/), a central platform for the development of R packages, R-related software and further projects. As IsolationForest has been uploaded to r-forge, we can simply install the package by executing following code in a R shell:

{% codeblock lang:r %}
install.packages("IsolationForest", repos="http://R-Forge.R-project.org")
{% endcodeblock %}

Try privileged shell if installation fails.

Then we are able to import library in python with the help of rpy2, just give importr the package name you would like to import:

{% codeblock lang:python %}
from rpy2.robjects.packages import importr
importr('IsolationForest')
{% endcodeblock %}

**Step 2\. Converting data to R format**

Look up the signature of the function you want to call and decide how will you convert the data. In this example, function IsolationTrees (or class constructor? whatever, I'm not a R expert) takes a data frame, which is corresponding to OrderedDict in python, and an integer positional argument. Actually a data frame is a bit similar to CSV formatted data. It has multiple rows and each row forms a vector, which in this case, is used for assessing outlier factors.

So first convert an ordinary Python list to R float vector:

{% codeblock lang:python %}
from rpy2.robjects import FloatVector
L = [1.0, 2.0, 3.0]     # original python list
rL = FloatVector(L)     # converted R float vector
{% endcodeblock %}

To convert multiple Python lists to R vectors (As in this example):

{% codeblock lang:python %}
rlists = list(map(FloatVector, lists))
{% endcodeblock %}

List type cast is enforced to prevent evil generators in Python 3\. After that, build an OrderedDict with sequential ordering number mapped to converted R vectors and finally pack it into a data frame:

{% codeblock lang:python %}
from collections import OrderedDict
from rpy2.robjects import DataFrame
d = OrderedDict(zip(map(str, range(len(rlists))), rlists))
data = DataFrame(d)
{% endcodeblock %}

** Step 3\. Calling R functions**

Do as R programmers do. With a magical dictionary of functions.

{% codeblock lang:python %}
from rpy2.robjects import r
tr = r['IsolationTrees'](data, rFactor=0)
sc = r['AnomalyScore'](data, tr)
{% endcodeblock %}

** Step 4\. Get data out of R**

Hate to say it but here is another reference-related problem and you will have to look up the documentation of the function you are calling. In this example, the function (or whatever) AnomalyScore returns a dictionary, and by the documentation I discovered the field I need is named $outF. Therefore a conversion is performed and the data is taken away.

{% codeblock lang:python %}
list(dict(zip(sc.names, list(sc)))['outF'])
{% endcodeblock %}

Make sure you are getting a Python list, not something weird. Bye bye, R!

I've packed the code into a Python library and published at [https://github.com/w1ndy/iforest.py](https://github.com/w1ndy/iforest.py). Hope some R guy comes across this library and desperately tries to figure out how to call it in R ;)
