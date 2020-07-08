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

Generics are abstract data types that describe a group of types. We will walk through the most common generics by explaining the following snippets:


{% highlight python %}
def func(student_info: Tuple[str, str, Dict[str, int]],
            grades: Optional[List[Union[int, float]]],
            grade_modifiers: Iterable[Callable[int, int]]) -> Mapping[str, float]:
    if grades is None:
        return {name: 0}
    return {name: sum(grades)/len(grades)}
{% endhighlight %}


[Optional](https://docs.python.org/3/library/typing.html#typing.Optional) describes that the type can be the subscripted type or `None`. By using the `Optional` generic on the `grades` kwarg we mark the optional existence of an actual list.

The `Optional[Type]` is equivalent with `Union[Type, None]`. One might say that it should be `Union[Type, NoneType]`, but as PEP 484 [states](https://www.python.org/dev/peps/pep-0484/#using-none), `None` and `NoneType` are equivalent when type hinting. On a personal note, I don't like this.
 
[List](https://docs.python.org/3/library/typing.html#typing.List) describes that the type should be a `list` that contains elements that are instaces of the subscripted type. In the above example, we observe that the list `grades`, if it exists due to the `Optional` generic, should contain a Union type only, which we will cover next.

We will see why we favor the usage of `Iterable` instead of `List` and compare the genericity of the two generics.

[Union](https://docs.python.org/3/library/typing.html#typing.Union) describes that the  elements of the optional list of the kwarg `grades` can be either `str` or `int`. Union types represent a common artificial parent of the subscripted types that can be used to represent all of them to a static typechecking tool.

As a side note, there is no interpreter speedup on using Union types, as Python remains a dynamic language, compared to usage of union types in static languages like C.
 
[Tuple](https://docs.python.org/3/library/typing.html#typing.Tuple) describes that the type should be a `tuple` that containts on each position instances of the subscripted types.

Here he observe that the kwarg `student` expects a tuple with three elements, a string, a string and a `Dict` generic type that we will talk through soon. The `Tuple` generic is actually a gem, due to the actual immutability of tuples.

[Dict](https://docs.python.org/3/library/typing.html#typing.Dict) describes that the 3rd argument of the `Tuple` type that should be received by the `student_info` kwarg should a `dict` object with the keys of type `str` and the values of type `int`.
This is not overall not recommended to use due to the lack of genericity, the corect way to represent such data structures is to use `Mapping`, as this coveres generic key-value structures, one of the most interesting such objects being `TypedDict`.

[Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable) generics are the way to describe an iterable type like: `set`, `list`, or any custom made type that is iterable, making in generic and permissive. The items generated by the iterator must be an instance of the
subscripted type of the generic. In the above example, the kwarg `grade_modifiers` is an iterable object.

[Mapping](https://docs.python.org/3/library/typing.html#typing.Mapping) generic is the right way of describing key-value data structures like: `dict`, `ChainMap`, `TypedValue`

[Callable](https://docs.python.org/3/library/typing.html#typing.Callable)


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
