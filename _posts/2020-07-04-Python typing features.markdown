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
* [@final](https://docs.python.org/3/library/typing.html#typing.final)

## Annotations in Python

Annotations are a documentation or static typechecking tool that provide the expected value of an object. Thus, it will not raise errors or provide runtime warnings
at all. Still, annotations are making the code more readable and specific behaviors can be forced regarding typechecking. Lets explore the benefits of type annotations: 

{% highlight python %}
def func(name, grades):
    ...
{% endhighlight %}

At a glance, when you are reading about `func` for the first time, you cannot be sure of the types of the arguments at all, nor about the return type. If there are no docstrings
or documentation available, we need to grind through the code to understand the possible behaviors of the function.

{% highlight python %}
def func(name: str, grades: list) -> dict:
    ...
{% endhighlight %}

By using annotations on the arguments and on the the return types, we can understand the expected behavior and the actual way to use `func`. On a more detailed level, we can look at
the func object to see how annotations are stored. Upon running `dir` on the `func` object, we observe that we have the attribute `__annotations__`. This is a dictionary with the keys
the kwargs of the function and the values the annotation value.


{% highlight python %}
print(func.__annotations__)
# {"name": <class 'str'>, "grades": <class 'list'>, "return": <class 'dict'>}.
{% endhighlight %}

As we can see, we still can't understand that much of the code, as the kwarg `grades` expects a list, but we don't know the actual typing of the elements in it. We should dig deeper to provide
support for generic subscripted typing.

## Generics

Generics are abstract data types that describe a subclass of types. We will walk through the most common generics by explaining the following snippet:

{% highlight python %}
from typing import Tuple, Mapping, Optional, List, Union, Iterable, Callable, Dict

def func(student_info: Tuple[str, str, Mapping[str, int]],
            grades: Optional[List[Union[int, float]]],
            grade_modifiers: Iterable[Callable[int, int]]) -> Dict[str, float]:
    ...
{% endhighlight %}

Lets analyse each kwarg to explain the documentation provided by the typing annotation:

{% highlight python %}
student_info: Tuple[str, str, Mapping[str, int]]
{% endhighlight %}

The `student_info` kwarg expects a tuple with three elements, a string, a string and dict-like object that maps strings to ints. Lets take a deeper look.

[Tuple](https://docs.python.org/3/library/typing.html#typing.Tuple) describes that the type should be a `tuple` that containts on each position instances of the subscripted types.

Here he observe that the kwarg `student` expects a tuple with three elements, a string, a string and a `Mapping` generic type that we will talk through soon. The `Tuple` generic is actually a gem, due to the actual immutability of tuples.

[Mapping](https://docs.python.org/3/library/typing.html#typing.Mapping) generic is the right way of describing key-value data structures like: `dict`, `ChainMap`, `TypedValue` or any custom key-value data structure, this one describers a mapping from `str` objects
to `int` objects.

 
{% highlight python %}
grades: Optional[List[Union[int, float]]]
{% endhighlight %}

The `grades` expects either None or a list that contains either ints or floats.

[Optional](https://docs.python.org/3/library/typing.html#typing.Optional) describes that the type can be the subscripted type or `None`. By using the `Optional` generic on the `grades` kwarg we mark the optional existence of an actual list.

The `Optional[Type]` is equivalent with `Union[Type, None]`. One might say that it should be `Union[Type, NoneType]`, but as PEP 484 [states](https://www.python.org/dev/peps/pep-0484/#using-none), `None` and `NoneType` are equivalent when type hinting. On a personal note, I don't like this.
 
[List](https://docs.python.org/3/library/typing.html#typing.List) describes that the type should be a `list` that contains elements that are instaces of the subscripted type. In the above example, we observe that the list `grades`, if it exists due to the `Optional` generic, should contain a Union type only, which we will cover next.

We will see why we favor the usage of `Iterable` instead of `List`.

[Union](https://docs.python.org/3/library/typing.html#typing.Union) describes that the  elements of the optional list of the kwarg `grades` can be either `str` or `int`. Union types represent a common artificial parent of the subscripted types that can be used to represent all of them to a static typechecking tool.

As a side note, there is no interpreter speedup on using Union types, as Python remains a dynamic language, compared to usage of union types in static languages like C.

{% highlight python %}
grade_modifiers: Iterable[Callable[int, int]]):
{% endhighlight %}

The `grade_modifiers` kwarg expects a list-like structure that contains function-like objects that expect an int parameter and returns an int object.

[Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable) generics are the way to describe an iterable type like: `set`, `list`, or any custom made type that is iterable, making in generic and permissive. The items generated by the iterator must be an instance of the
subscripted type of the generic. In the above example, the kwarg `grade_modifiers` is an iterable object.

[Callable](https://docs.python.org/3/library/typing.html#typing.Callable) generic is an abstraction over any object that can be called on which we can enforce further typing, for example, `my_func = lambda x: x + 1` has no possibility of further type hinting, but `my_func: Callable[int, int] = lambda x: x + 1` gives the reader a type hint on the further usage of the lambda function.
Any callable object (functions, methods, lambda function, callable objects) are an instance of the `Callable` generic.

{% highlight python %}
Dict[str, float]
{% endhighlight %}

[Dict](https://docs.python.org/3/library/typing.html#typing.Dict) describes that the return type of our function should be a `dict` with the keys of type `str` and value of type `float`.
This is not overall not recommended to use due to the lack of genericity, the corect way to represent such data structures is to use `Mapping`, as this coveres generic key-value structures, one of the most interesting such objects being `TypedDict`.


## ForwardRef and future annotations

One of the common downfalls when using explicit type hinting is that it requires to actually have the type available when annotating. This generates boilerplate code that is not useful and can generate the
infamous circular dependecy error. 

{% highlight python %}
from module.db1 import db1
from module.db2 import db2
from module.db3 import db3

def func(name: str, grades: list, database: Union[db1, db2]) -> None:
    ...
{% endhighlight %}


As of python3.8, if you are using the `annotations` from the `__future__` package, types can be stored as qualified string paths and will be resolved at runtime/when the typechecking tools needs it. This actually provided
an amazing speedup in the `typing` package.

{% highlight python %}
from __future__ import annotations
import module
from typing import Union

def func(name: str, grades: list, database: Union["module.db1", "module.db2"]) -> None:
    ...

print (func.__annotations__)
# {'name': <class 'str'>, 'grades': <class 'list'>, 'database': typing.Union['module.database1', 'module.database2']}
{% endhighlight %}

If you are not using the annotations package, the interpreter creates `ForwardRef` objects, as types are not stored as strings.

{% highlight python %}
import module
from typing import Union

def func(name: str, grades: list, database: Union["module.db1", "module.db2"]) -> None:
    ...

print (func.__annotations__)
# {'name': <class 'str'>, 'grades': <class 'list'>, 'database': typing.Union[ForwardRef('module.database1'), ForwardRef('module.database2')]}
{% endhighlight %}

## Final and @final
The `Final` type and the `@final` decorator are useful when:
* a class shouldn't be inherited.
* a method shouldn't be overriden.
* a variable shouldn't be reassigned.

{% highlight python %}
from typing import Final, final

class Interface:
    # don't reassign me
    PI: Final = 3.14

    @final
    def pi(self):
        return Interface.PI

    def compute(self)
        raise NotImplemented

@final
class Implementation
    def compute(self):
        raise 2*self.pi()
{% endhighlight %}

Breaking these constraints will result in raising an error. One interesting fact is the the `Final` is actually a generic one, being able to be subscripted. Why would this be useful? To keep a reference to a container object, like:

{% highlight python %}
from typing import Final, Dict

grades_dict: Final[Dict[str, int]] = dict()
{% endhighlight %}


In this post, we've observed how to improve a python codebase by providing better documentation and type hinting, enabling static type checking for IDEs. In further posts we will talk about building restrictions based on annotations.




