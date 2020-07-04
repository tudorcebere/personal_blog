---
layout: post
title:  "Python typing features."
date:   2020-04-24 15:10:56 +0900
categories: python typing
---

[Recent](https://docs.python.org/3/whatsnew/3.8.html#typing) improvements in the typing module provided python amazing mechanisms for building healthier codebases.

We will take a look at the following features:
* annotations in Python
* [ForwardRef](https://docs.python.org/3/library/typing.html#typing.ForwardRef)
* [generics](https://docs.python.org/3/library/typing.html#generics)
* [@overload](https://docs.python.org/3/library/typing.html#typing.overload)
* [@final](https://docs.python.org/3/library/typing.html#typing.final)

## Annotations in Python


{% highlight python %}
def func(name, grades):
  return {name: sum(grades)/len(grades)}
{% endhighlight %}



{% highlight python %}
def func(name: str, grades: list) -> dict:
    return {name: sum(grades)/len(grades)}
{% endhighlight %}



{% highlight python %}
print(func.__annotations__)
{% endhighlight %}

## Generics


## ForwardRef

{% highlight python %}
def func(name: str, grades: list, database: ??) -> None:
    database.insert(name, sum(grades)/len(grades))
{% endhighlight %}


{% highlight python %}
from database1 import db1
from database2 import db2
from database3 import db3

def func(name: str, grades: list, database: Union[db1, db2, db3]) -> None:
    database.insert(name, sum(grades)/len(grades))
{% endhighlight %}


{% highlight python %}
from __future__ import annotations

def func(name: str, grades: list, database: Union["database1.db1", "database.db2", "database.db3"]) -> None:
    database.insert(name, sum(grades)/len(grades))
{% endhighlight %}


{% highlight python %}
print (func.__annotations__)
{% endhighlight %}