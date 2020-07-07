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

Annotations are a documentation or static typechecking tool that provide the expected value of an object. Thus, it will not raise errors or provide runtime warnings
at all. Still, annotations are making the code more readable and specific behaviors can be forced regarding typechecking. Lets explore the benefits of type annotations: 

{% highlight python %}
def func(name, grades):
  return {name: sum(grades)/len(grades)}
{% endhighlight %}

At a glance, when you are reading about `func` for the first time, you cannot be sure of the types of the arguments at all, nor about the return type. If there are no docstrings
or documentation available, we need to grind through the code to understand the possible behaviors of the function.

{% highlight python %}
def func(name: str, grades: list) -> dict:
    return {name: sum(grades)/len(grades)}
{% endhighlight %}

By using annotations on the arguments and on the the return types, we can understand the expect behavior and the actual way to use `func`. On a more detailed level, we can look at
the func object to see how annotations are stored. Upon running `dir` on the `func` object, we observe that we have the attribute `__annotations__`. This is a dictionary with the keys
the kwargs of the function and the values the annotation value.


{% highlight python %}
print(func.__annotations__)
# {"name": <class 'str'>, "grades": <class 'list'>, "return": <class 'dict'>}.
{% endhighlight %}

As we can see, we still can't understand that much of the code, as the kwarg `grades` expects a list, but we don't know the actual typing of the elements in it. We should dig deeper to provide
support for generic subscripted typing.

## Generics

Generics are abstract data types that describes a generic behavior, not actually describing the implementation of a type or a data structure.

The most common generics are:
* (Optional)[https://docs.python.org/3/library/typing.html#typing.Optional]
* (Union)[https://docs.python.org/3/library/typing.html#typing.Union]
* (List)[https://docs.python.org/3/library/typing.html#typing.List]
* (Tuple)[https://docs.python.org/3/library/typing.html#typing.Tuple]
* (Dict)[https://docs.python.org/3/library/typing.html#typing.Dict]
* (Iterable)[https://docs.python.org/3/library/typing.html#typing.Iterable]
* (Mapping)[https://docs.python.org/3/library/typing.html#typing.Mapping]
* (Callable)[https://docs.python.org/3/library/typing.html#typing.Callable]


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
