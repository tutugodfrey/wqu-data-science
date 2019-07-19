

```python
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
```


```python
import expectexception
```

# Object Oriented Programming

Over the last lectures, I have sometimes referred to **objects** or **Python objects**. I've also mentioned **methods** of objects (e.g. the `get` method of `dict`). What do these terms mean?

For now we can think of an object as anything we can store in a variable. We can have objects with different `type`. We might also call an object's `type` its **class**. We'll come back to class later.


```python
x = 42
print('%d is an object of %s' % (x, type(x)))

x = 'Hello world!'
print('%s is an object of %s' % (x, type(x)))

x = {'name': 'Dylan', 'age': 26}
print('%s is an object of %s' % (x, type(x)))
```

    42 is an object of <class 'int'>
    Hello world! is an object of <class 'str'>
    {'name': 'Dylan', 'age': 26} is an object of <class 'dict'>


We already know that integers, strings, and dictionaries behave differently. They have different properties and different capabilities. In the language of programming, we say they have different **attributes** and **methods**.

An object's attributes are its internal variables that are used to store information about the object.


```python
# a complex number has real and imaginary parts
x = complex(5, 3)
print(x.real)
print(x.imag)
```

    5.0
    3.0


An object's methods are its internal functions that implement different capabilities.


```python
x = 'Dylan'
print(x.lower())
print(x.upper())
```

    dylan
    DYLAN


We'll interact with an object's methods more often than its attributes. The attributes represent the _state_ of an object. We usually prefer to mutate the state of an object via its methods, since the methods represent the actions one can take safely without breaking the object. Often the attributes of an object will be immutable.


```python
%%expect_exception AttributeError

x = complex(5, 3)
x.real = 6
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mAttributeError[0m                            Traceback (most recent call last)
    [0;32m<ipython-input-6-947651b88b0e>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [1;32m      2[0m [0mx[0m [0;34m=[0m [0mcomplex[0m[0;34m([0m[0;36m5[0m[0;34m,[0m [0;36m3[0m[0;34m)[0m[0;34m[0m[0m
    [0;32m----> 3[0;31m [0mx[0m[0;34m.[0m[0mreal[0m [0;34m=[0m [0;36m6[0m[0;34m[0m[0m
    [0m
    [0;31mAttributeError[0m: readonly attribute


An example of a method that mutates an object is the `append` method of a `list`.


```python
x = [35, 'example', 348.1]
x.append(True)
print(x)
```

    [35, 'example', 348.1, True]


How do we know what the attributes and methods of an object are? We can use Python's `dir` function. We can use `dir` on an object or on a class.


```python
# dir on an object
x = 42
print(dir(x)[-6:]) # I've truncated the results for clarity

# dir on a class
print(dir(int)[-6:])
```

    ['denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']
    ['denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']


We can also look up documentation on the class. For example, [here's Python's documentation on the built-in Python types](https://docs.python.org/2/library/stdtypes.html). We'll use documentation more and more as we incorporate third-party libraries and tools into Python.

## Classes

But this isn't the whole story. The methods and attributes of a `dict` don't tell us anything about key-value pairs or hashing. The full definition of an object is an object's class. We can define our own classes to create objects that carry out a variety of related tasks or represent information in a convenient way. Some examples we'll deal with later in the course are classes for making plots and graphs, classes for creating and analyzing tables of data, and classes for doing statistics and regression.

For now, let's implement a class called `Rational` for working with fractional numbers (e.g. 5/15). The first thing we'll need `Rational` to do is to be able to create a `Rational` object. We define how this should work with a special (hidden) method called `__init__`. We'll also define another special method called `__repr__` that tells Python how to print out the object.


```python
class Rational(object):

    def __init__(self, numerator, denominator):
        self.numerator = numerator
        self.denominator = denominator

    def __repr__(self):
        return '%d/%d' % (self.numerator, self.denominator)
```


```python
fraction = Rational(4, 3)
print(fraction)
```

    4/3


You might have noticed that both of the methods took as a first argument the keyword `self`.  The first argument to any method in a class is the instance of the class upon which the method is being called.  Think of a class like a blueprint from which possibly many objects are built. The `self` argument is the mechanism Python uses so that the method can know which instance of the class it is being called upon.  When the method is actually called, we can call it in two ways.  Lets say we create a class `MyClass` with method `.do_it(self)`, if we instantiate an object from this class, we can call the method in two ways:


```python
class MyClass(object):
    def __init__(self, num):
        self.num = num
        
    def do_it(self):
        print(self.num)
        
myclass = MyClass(2)
myclass.do_it()
MyClass.do_it(myclass)
```

    2
    2


In one way `myclass.do_it()` the `self` argument is understood because `myclass` is an instance of `MyClass`.  This is the almost universal way to do call a method.  The other possibility is `MyClass.do_it(myclass)` where we are passing in the object `myclass` as the `self` argument, this syntax is much less common.  

Like all Python arguments, there is no need for `self` to be named `self`, we could also call it `this` or `apple` or `wizard`.  However, the use of `self` is a very strong Python convention which is rarely broken.  You should use this convention so that your code is understood by other people.

Lets get back to our `Rational` class.  So far, we can make a `Rational` object and `print` it out, but it can't do much else. We might also want a `reduce` method that will divide the numerator and denominator by their greatest common divisor. We will therefore need to write a function that computes the greatest common divisor. We'll add these to our class definition.


```python
class Rational(object):

    def __init__(self, numerator, denominator):
        self.numerator = numerator
        self.denominator = denominator

    def __repr__(self):
        return '%d/%d' % (self.numerator, self.denominator)

    def _gcd(self):
        smaller = min(self.numerator, self.denominator)
        small_divisors = {i for i in range(1, smaller + 1) if smaller % i == 0}
        print(small_divisors, 'small divisor')
        larger = max(self.numerator, self.denominator)
        common_divisors = {i for i in small_divisors if larger % i == 0}
        print(common_divisors, 'common divisor')
        return max(common_divisors)

    def reduce(self):
        gcd = self._gcd()
        print(gcd, 'gcd')
        self.numerator = self.numerator / gcd
        self.denominator = self.denominator / gcd
        print(self.numerator, self.denominator, 'values')
        print(self, 'self')
        return self
```


```python
fraction = Rational(16, 32)
fraction.reduce()
print(fraction)
```

    {1, 2, 4, 8, 16} small divisor
    {1, 2, 4, 8, 16} common divisor
    16 gcd
    1.0 2.0 values
    1/2 self
    1/2


We're gradually building up the functionality of our `Rational` class, but it has a huge problem: we can't do math with it!


```python
# Notice that even though we perform operation like the ones shown below
# we are unable to do the same with the result of the fraction object
print(1/2 +4)
print(6 * 3/5)
```

    4.5
    3.6



```python
%%expect_exception TypeError

print(4 * fraction)
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-15-cc5282b78078>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [0;32m----> 2[0;31m [0mprint[0m[0;34m([0m[0;36m4[0m [0;34m*[0m [0mfraction[0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;31mTypeError[0m: unsupported operand type(s) for *: 'int' and 'Rational'


We have to tell Python how to implement mathematical operators (`+`, `-`, `*`, `/`) for our class.


```python
print(dir(int))
```

    ['__abs__', '__add__', '__and__', '__bool__', '__ceil__', '__class__', '__delattr__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floor__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getnewargs__', '__gt__', '__hash__', '__index__', '__init__', '__init_subclass__', '__int__', '__invert__', '__le__', '__lshift__', '__lt__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__round__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'bit_length', 'conjugate', 'denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']


If we look at `dir(int)` we see it has hidden methods like `__add__`, `__div__`, `__mul__`, `__sub__`, etc. Just like `__repr__` tells Python how to `print` our object, these hidden methods tell Python how to handle mathematical operators.

Let's add the methods implementing mathematical operations to our class definition. To perform addition or subtraction, we'll have to find a common denominator with the number we're adding. For simplicity, we'll only implement multiplication. We won't be able to add, subtract, or divide. Even implementing only multiplication will require quite a bit of logic.


```python
class Rational(object):

    def __init__(self, numerator, denominator):
        self.numerator = numerator
        self.denominator = denominator

    def __repr__(self):
        return '%d/%d' % (self.numerator, self.denominator)

    def __mul__(self, number):
        if isinstance(number, int):
            return Rational(self.numerator * number, self.denominator)
        elif isinstance(number, Rational):
            return Rational(self.numerator * number.numerator, self.denominator * number.denominator)
        else:
            raise TypeError('Expected number to be int or Rational. Got %s' % type(number))
        
    def _gcd(self):
        smaller = min(self.numerator, self.denominator)
        small_divisors = {i for i in range(1, smaller + 1) if smaller % i == 0}
        larger = max(self.numerator, self.denominator)
        common_divisors = {i for i in small_divisors if larger % i == 0}
        return max(common_divisors)

    def reduce(self):
        gcd = self._gcd()
        self.numerator = self.numerator / gcd
        self.denominator = self.denominator / gcd
        return self
```


```python
print(Rational(4, 6) * 3)
print(Rational(5, 9) * Rational(2, 3))
```

    12/6
    10/27



```python
%%expect_exception TypeError

# remember, no support for float
print(Rational(4, 6) * 2.3)
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-19-e585b376123d>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [1;32m      2[0m [0;31m# remember, no support for float[0m[0;34m[0m[0;34m[0m[0m
    [0;32m----> 3[0;31m [0mprint[0m[0;34m([0m[0mRational[0m[0;34m([0m[0;36m4[0m[0;34m,[0m [0;36m6[0m[0;34m)[0m [0;34m*[0m [0;36m2.3[0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;32m<ipython-input-17-bfab24d3b796>[0m in [0;36m__mul__[0;34m(self, number)[0m
    [1;32m     14[0m             [0;32mreturn[0m [0mRational[0m[0;34m([0m[0mself[0m[0;34m.[0m[0mnumerator[0m [0;34m*[0m [0mnumber[0m[0;34m.[0m[0mnumerator[0m[0;34m,[0m [0mself[0m[0;34m.[0m[0mdenominator[0m [0;34m*[0m [0mnumber[0m[0;34m.[0m[0mdenominator[0m[0;34m)[0m[0;34m[0m[0m
    [1;32m     15[0m         [0;32melse[0m[0;34m:[0m[0;34m[0m[0m
    [0;32m---> 16[0;31m             [0;32mraise[0m [0mTypeError[0m[0;34m([0m[0;34m'Expected number to be int or Rational. Got %s'[0m [0;34m%[0m [0mtype[0m[0;34m([0m[0mnumber[0m[0;34m)[0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m     17[0m [0;34m[0m[0m
    [1;32m     18[0m     [0;32mdef[0m [0m_gcd[0m[0;34m([0m[0mself[0m[0;34m)[0m[0;34m:[0m[0;34m[0m[0m
    
    [0;31mTypeError[0m: Expected number to be int or Rational. Got <class 'float'>



```python
%%expect_exception TypeError

# also, no addition, subtraction, etc.
print(Rational(4, 6) + Rational(2, 3))
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-20-3e2914851db1>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [1;32m      2[0m [0;31m# also, no addition, subtraction, etc.[0m[0;34m[0m[0;34m[0m[0m
    [0;32m----> 3[0;31m [0mprint[0m[0;34m([0m[0mRational[0m[0;34m([0m[0;36m4[0m[0;34m,[0m [0;36m6[0m[0;34m)[0m [0;34m+[0m [0mRational[0m[0;34m([0m[0;36m2[0m[0;34m,[0m [0;36m3[0m[0;34m)[0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;31mTypeError[0m: unsupported operand type(s) for +: 'Rational' and 'Rational'


Defining classes can be a lot of work. We have to imagine all the ways we might want to use an object, and where we might run into trouble. This is also true of defining functions, but classes will typically handle many tasks while a function might only do one.

## Private Methods in Python

You might have noticed we have used some methods which start with `_` such as `_gcd`.  This has a conventional meaning in Python which is formally implemented in other languages, the notion of a private function.  Classes are used to encapsulate functionality and data while providing an interface to the outside world of other objects.  Think of a program as a company, each worker has their own responsibilities and they know that other people the company perform certain tasks, but they don't necessary know how those people perform those tasks.  

In order to make this possible, Classes have both public and private methods.  Public methods are methods which are exposed to other objects or user interaction.  Private methods are used internally to the object, often in a "helper" sense.  In some languages this notion of public and private methods is enforced and the programmer will have to specify every method as either public or private.  In Python every method is public, but to distinguish which methods we mean to be private, we add an underscore to the front of the method, hence `_gcd`.  This is a note to someone using the class that this method should only be called inside the object and can be subject to change with new versions, whereas the public methods will hopefully not change their interface.

Another Python convention dealing with underscores are the so called `dunder` methods which have double underscores before and after the method names.  There are a bunch of these in Python `__init__, __name__, __add__`, etc and they have special meaning.  Note that they are generally considered private methods as well except in special circumstances.  In the case of methods like `__add__`, they are what allow the programmer to specify the `+` operation.  Since these method have special meaning to Python they should only be used with care.  Additionally, even though overloading things like the `+` operator might make sense to you as you program it, it can very confusing to someone reading your code as Python's dynamic type system usually does not allow determination of types until runtime, usually defining an `.add` method is much more clear.

## When do we want Classes?

When we want to perform a set of related tasks, especially in repetition, we will usually want to define a new class. We will see that in most of the third-party libraries we will use, the major tools they introduce to Python are new classes. For example, later in the course we'll learn about the Pandas library, whose main feature is the `DataFrame` class.


```python
import pandas as pd

df = pd.DataFrame({'a': [1, 2, 5], 'b': [True, False, True]})

print(type(df))
df.head()
```

    <class 'pandas.core.frame.DataFrame'>





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



Here's the (abridged) beginning of the DataFrame class definition:

```python
class DataFrame(NDFrame):

    def __init__(self, data=None, index=None, columns=None, dtype=None,
                 copy=False):
        if data is None:
            data = {}
        if dtype is not None:
            dtype = self._validate_dtype(dtype)

        if isinstance(data, DataFrame):
            data = data._data

        if isinstance(data, BlockManager):
            mgr = self._init_mgr(data, axes=dict(index=index, columns=columns),
                                 dtype=dtype, copy=copy)
        elif isinstance(data, dict):
            mgr = self._init_dict(data, index, columns, dtype=dtype)
        elif isinstance(data, ma.MaskedArray):
            import numpy.ma.mrecords as mrecords
            # masked recarray
            if isinstance(data, mrecords.MaskedRecords):
                mgr = _masked_rec_array_to_mgr(data, index, columns, dtype,
                                               copy)

            # a masked array
            else:
                mask = ma.getmaskarray(data)
                if mask.any():
                    data, fill_value = maybe_upcast(data, copy=True)
                    data[mask] = fill_value
                else:
                    data = data.copy()
                mgr = self._init_ndarray(data, index, columns, dtype=dtype,
                                         copy=copy)

        elif isinstance(data, (np.ndarray, Series, Index)):
            if data.dtype.names:
                data_columns = list(data.dtype.names)
                data = dict((k, data[k]) for k in data_columns)
                if columns is None:
                    columns = data_columns
                mgr = self._init_dict(data, index, columns, dtype=dtype)
            elif getattr(data, 'name', None) is not None:
                mgr = self._init_dict({data.name: data}, index, columns,
                                      dtype=dtype)
            else:
                mgr = self._init_ndarray(data, index, columns, dtype=dtype,
                                         copy=copy)
        elif isinstance(data, (list, types.GeneratorType)):
            if isinstance(data, types.GeneratorType):
                data = list(data)
            if len(data) > 0:
                if is_list_like(data[0]) and getattr(data[0], 'ndim', 1) == 1:
                    if is_named_tuple(data[0]) and columns is None:
                        columns = data[0]._fields
                    arrays, columns = _to_arrays(data, columns, dtype=dtype)
                    columns = _ensure_index(columns)

                    # set the index
                    if index is None:
                        if isinstance(data[0], Series):
                            index = _get_names_from_index(data)
                        elif isinstance(data[0], Categorical):
                            index = _default_index(len(data[0]))
                        else:
                            index = _default_index(len(data))

                    mgr = _arrays_to_mgr(arrays, columns, index, columns,
                                         dtype=dtype)
                else:
                    mgr = self._init_ndarray(data, index, columns, dtype=dtype,
                                             copy=copy)
            else:
                mgr = self._init_dict({}, index, columns, dtype=dtype)
        elif isinstance(data, collections.Iterator):
            raise TypeError("data argument can't be an iterator")
        else:
            try:
                arr = np.array(data, dtype=dtype, copy=copy)
            except (ValueError, TypeError) as e:
                exc = TypeError('DataFrame constructor called with '
                                'incompatible data and dtype: %s' % e)
                raise_with_traceback(exc)

            if arr.ndim == 0 and index is not None and columns is not None:
                values = cast_scalar_to_array((len(index), len(columns)),
                                              data, dtype=dtype)
                mgr = self._init_ndarray(values, index, columns,
                                         dtype=values.dtype, copy=False)
            else:
                raise ValueError('DataFrame constructor not properly called!')

        NDFrame.__init__(self, mgr, fastpath=True)
```

That's a lot of code just for `__init__`!

Often we'll use the relationship between a new class and existing classes to _inherit_ functionality, saving us from writing some code.

## Inheritance

Often the classes we define in Python will build off of existing ideas in other classes. For example, our `Rational` class is a number, so it should behave like other numbers. We could write an implementation of `Rational` that uses `float` arithmetic and simply converts between floating point and rational representations during input and output. This would save us complexity in implementing the arithmetic, but might complicate object creation and representation. Even if you never write a class, it's useful to understand the idea of inheritance and the relationship between classes.

Lets write a general class called `Rectangle`, it will have two attributes, a length and a width, as well as a few methods.


```python
class Rectangle(object):
    def __init__(self, height, length):
        self.height = height
        self.length = length
    
    def area(self):
        return self.height * self.length
    
    def perimeter(self):
        return 2 * (self.height + self.length)
```

Now a square is also a rectangle, but its somewhat more restricted in that it has the same height as length, so we can subclass `Rectangle` and enforce this in code.


```python
class Square(Rectangle):
    def __init__(self, length):
        super(Square, self).__init__(length, length)
```


```python
s = Square(5)
s.area(), s.perimeter()
```




    (25, 20)



Sometimes (although not often) we want to actually check the type of a python object (what class it is from).  There are two ways of doing this, lets first look at a few examples to get a sense of the difference.


```python
type(s) == Square
```




    True




```python
type(s) == Rectangle
```




    False




```python
isinstance(s, Rectangle)
```




    True



As you might have noticed checking type quality only checks the exact class to which an object belongs, whereas `isinstance(c, Class)` checks if `c` is either a member of class `Class` or a member of a subclass of `Class`.  Almost always `isinstance` is the proper way to check this, because if a class implements some sort of functionality, its subclasses usually implement the same functionality (they just might have some extra bonus functionality!).

## Object Oriented Programming

Now that we understand objects and classes, let's return to the idea of _object oriented programming_. Object oriented programming (`OOP`) is a perspective that programs are essentially about the creation of objects and the interaction between them. In `OOP`, almost every piece of code either describes an object, an object's attributes, or an object's methods. Keeping this perspective in mind can help us understand what's happening in a program.

## Questions:
- What are some built-in Python objects that might inherit from the same parent class?

*Copyright &copy; 2017 The Data Incubator.  All rights reserved.*
