

```python
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
```


```python
import expectexception
```

# Pythonisms

Much of what we covered in the previous notebook can be fairly generally applicable.  Even the Python syntax is quite similar to other languages in the C family.  But there are a few things that every language chooses how to do beyond just syntax (although many new languages do take some inspiration from the Python way of doing things).  The things we will go over here

* What is Pythonic?
* Float Division
* Python `import` system
* Exceptions
* How to debug Python


Lets start by what we mean by the Python way of doing things.

## `Pythonic`

When learning Python, you will probably browse blogs and other web resources that claim certain things are `Pythonic`.  Python has an opinionated way of doing things, mostly captured in the Zen of Python


```python
import this
```

    The Zen of Python, by Tim Peters
    
    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!


`Pythonic` practices are those which the general Python community has agreed are preferable, sometimes this is purely a stylistic consideration and other times it may be related to the way the Python runs.

Making your code `Pythonic` can also be useful when other Python programmers need to interact with it as they will be familiar with the idioms and paradigms you use.  

## Imports

In the cells above you might have noticed we used the `import <package>` syntax.  This construct allows us to include code from other python files or more generally modules (collections of files) and packages (collection of modules) into the current code we are working with.  For the purposes of this course, we have installed all the packages you will need on your machine, but for working with packages, some recommended tools are 

- conda
- pip

With installed packages (usually installed with one of those two "package managers"), we can import the package with the `import` command.  We can also import only parts of the package.  For example, one package we will use in the course is called `pandas`.  We can import `pandas`


```python
import pandas
pandas
```




    <module 'pandas' from '/opt/conda/lib/python3.6/site-packages/pandas/__init__.py'>



We can also import pandas, but call it something else (saves a bit of typing and is conventional for some of the main packages in the Python scientific stack).


```python
import pandas as pd
pd
```




    <module 'pandas' from '/opt/conda/lib/python3.6/site-packages/pandas/__init__.py'>



Now when we want to use a function or class from pandas, we need to call it with the syntax `pd.function` or `pd.class`.  For example, the `DataFrame` object


```python
pd.DataFrame
```




    pandas.core.frame.DataFrame



Note that this DataFrame does not exist in the main namespace.


```python
%%expect_exception NameError

DataFrame
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mNameError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-9-06a8df4645de>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [0;32m----> 2[0;31m [0mDataFrame[0m[0;34m[0m[0m
    [0m
    [0;31mNameError[0m: name 'DataFrame' is not defined


We can also just import parts of a package, we can even import them and give them another name!


```python
from pandas import DataFrame as dframe
dframe
```




    pandas.core.frame.DataFrame



Another thing we can do is to import everything into the main namespace using the syntax

```python

from pandas import *
```

This is highly discouraged because it can cause problems when multiple packages have a function or class with the same name (not uncommon, think about a function like `.info`).

We have covered the basic mechanics of the import system, but what does it allow us to do?  Having a sane packaging system allows Python users to package bundles of functionality into modules and packages which can be imported into other bits of codes.  If well written, these packages operate mostly like black boxes, where the user understands _what_ the package is doing, but not necessarily _how_ it is performing its functionality.  

While it may seem like this is giving up too much control, most of us don't understand exactly how our computer processor works, or even the keyboard, yet we are perfectly comfortable using them to serve their purpose. Packages are similar and when written well can be invaluable tools that allows us incorporate well written tested code that does powerful things into our applications with very little difficulty.

## Standard Library

One useful thing we can do with `import` statements is import packages in the Python standard library.  These are packages which are packaged with the interpreter and available on (almost) any Python installation.  These packages server a wide variety of purposes, here we have listed just a few along with their description.  For the rest, checkout the [documentation](https://docs.python.org/2/library/).

- `collections` - containers
- `re` - regular expressions
- `datetime` - date and time handling
- `heapq` - the heap queue algorithm
- `itertools` - functions for help with iteration
- `functools` - function to assist with functional programming
- `os` - operating system interfaces
- `sys` - system functions
- `pickle` - serialize Python objects
- `gzip` - work with Gzipped files
- `time` - time access
- `argparse` - command line argument handling
- `threading` - threading interface
- `multiprocessing` - process based "threading"
- `subprocess` - subprocess management
- `unittest` - testing tools
- `pdb` - debugger




These packages are optimized, reliable, and available anywhere there is a Python installation, so use them when you can!

## Exceptions

An exception is something that deviates from the norm.  In Python its no different, exceptions are when your program deviates from expected behavior.  The Python interpreter will attempt to execute any code that it's given and when it can't, it will raise an `Exception`.  In our notebooks you will note the `%%expect_exception` magic.  This is just a sign that we know there will be an exception in that cell.  For example, lets try to add a number to a string.


```python
%%expect_exception TypeError

2 + '3'
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-11-f5fbcd61b14a>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [0;32m----> 2[0;31m [0;36m2[0m [0;34m+[0m [0;34m'3'[0m[0;34m[0m[0m
    [0m
    [0;31mTypeError[0m: unsupported operand type(s) for +: 'int' and 'str'


We can see that this raises a `TypeError` because Python doesn't know how to add a string and an integer together (Python will not coerce one of the values into a different type; remember the Zen of Python: 'In the face of ambiguity, refuse the temptation to guess').  Exceptions are often very readable and helpful to debug code, however, we can also write code to handle exceptions when they occur.  Lets write a function which adds to things together (basically just another version of the add function) except it will catch the `TypeError` and do some conversion.


```python
def add(x, y):
    try:
        return x + y
    except TypeError:
        return float(x) + float(y)
```

Now lets run something similar to the previous example


```python
add(2, '3')
```




    5.0



As seen above, the way to handle Exceptions is with the `try` and `except` keywords.  The `try` block specifies a bit of code to try to run and the `except` block handles all exceptions that are specifically enumerated.  One can also catch all exceptions by doing

```python

try:
    func()
except:
    handle_exception()
    
```

But this is not generally a good idea since Python uses Exceptions for all sorts of things (sometimes even exiting programs) and you don't want to catch Exceptions which Python is using for a different purpose.  Think of `Exception` handling as handling the small probability things that will happen in your code, not as a tool to anticipate anything.

We have seen exceptions, but what are the alternative?  One option, used by other languages is to test ahead of time that conditions necessary to proceed are met.  We can rewrite the add function in a different way.


```python
def add_2(x,y):
    if not isinstance(x, (float, int)):
        x = float(x)
    if not isinstance(y, (float, int)):
        y = float(y)
    return x + y
add_2(2, '3')
```




    5.0



This also works, but its not Pythonic.  The Pythonic way of thinking about this is roughly analogous to "its easier to ask for forgiveness than permission".  Throwing exceptions actually has other positive benefits, such as the ability to handle errors at higher level code instead of in low level functions.  

What we mean by this is if we have a series of functions `f_a,f_b,f_c` and `f_a` calls `f_b` which calls `f_c`, we can choose to handle an exception in `f_c` in any of these functions!

## Python Debugging

We have seen how to handle errors with `Exceptions`, but how do we figure out whats wrong when we have errors that we haven't handled?

Lets look again at our previous example.


```python
%%expect_exception TypeError

2 + '3'
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-15-f5fbcd61b14a>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [0;32m----> 2[0;31m [0;36m2[0m [0;34m+[0m [0;34m'3'[0m[0;34m[0m[0m
    [0m
    [0;31mTypeError[0m: unsupported operand type(s) for +: 'int' and 'str'


If we look at the returned text, referred to as a `Traceback`, we can see much useful information.  Tracebacks should be read starting from the bottom and working up.  In this case the Traceback tells us exactly what happened, we tried to add an `int` and a `str` and there is no way to do this.  It even points to the exact line of code where this error occurs.  

Lets take a look at a more complicated Traceback.  We will create a pandas `DataFrame` with illegal arguments.


```python
%%expect_exception ValueError

pd.DataFrame(['one','two','three'],['test'])
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mValueError[0m                                Traceback (most recent call last)
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36mcreate_block_manager_from_blocks[0;34m(blocks, axes)[0m
    [1;32m   4295[0m [0;34m[0m[0m
    [0;32m-> 4296[0;31m         [0mmgr[0m [0;34m=[0m [0mBlockManager[0m[0;34m([0m[0mblocks[0m[0;34m,[0m [0maxes[0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m   4297[0m         [0mmgr[0m[0;34m.[0m[0m_consolidate_inplace[0m[0;34m([0m[0;34m)[0m[0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36m__init__[0;34m(self, blocks, axes, do_integrity_check, fastpath)[0m
    [1;32m   2794[0m         [0;32mif[0m [0mdo_integrity_check[0m[0;34m:[0m[0;34m[0m[0m
    [0;32m-> 2795[0;31m             [0mself[0m[0;34m.[0m[0m_verify_integrity[0m[0;34m([0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m   2796[0m [0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36m_verify_integrity[0;34m(self)[0m
    [1;32m   3005[0m             [0;32mif[0m [0mblock[0m[0;34m.[0m[0m_verify_integrity[0m [0;32mand[0m [0mblock[0m[0;34m.[0m[0mshape[0m[0;34m[[0m[0;36m1[0m[0;34m:[0m[0;34m][0m [0;34m!=[0m [0mmgr_shape[0m[0;34m[[0m[0;36m1[0m[0;34m:[0m[0;34m][0m[0;34m:[0m[0;34m[0m[0m
    [0;32m-> 3006[0;31m                 [0mconstruction_error[0m[0;34m([0m[0mtot_items[0m[0;34m,[0m [0mblock[0m[0;34m.[0m[0mshape[0m[0;34m[[0m[0;36m1[0m[0;34m:[0m[0;34m][0m[0;34m,[0m [0mself[0m[0;34m.[0m[0maxes[0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m   3007[0m         [0;32mif[0m [0mlen[0m[0;34m([0m[0mself[0m[0;34m.[0m[0mitems[0m[0;34m)[0m [0;34m!=[0m [0mtot_items[0m[0;34m:[0m[0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36mconstruction_error[0;34m(tot_items, block_shape, axes, e)[0m
    [1;32m   4279[0m     raise ValueError("Shape of passed values is {0}, indices imply {1}".format(
    [0;32m-> 4280[0;31m         passed, implied))
    [0m[1;32m   4281[0m [0;34m[0m[0m
    
    [0;31mValueError[0m: Shape of passed values is (1, 3), indices imply (1, 1)
    
    During handling of the above exception, another exception occurred:
    
    [0;31mValueError[0m                                Traceback (most recent call last)
    [0;32m<ipython-input-16-f56d47fdabdf>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      1[0m [0;34m[0m[0m
    [0;32m----> 2[0;31m [0mpd[0m[0;34m.[0m[0mDataFrame[0m[0;34m([0m[0;34m[[0m[0;34m'one'[0m[0;34m,[0m[0;34m'two'[0m[0;34m,[0m[0;34m'three'[0m[0;34m][0m[0;34m,[0m[0;34m[[0m[0;34m'test'[0m[0;34m][0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/frame.py[0m in [0;36m__init__[0;34m(self, data, index, columns, dtype, copy)[0m
    [1;32m    328[0m                 [0;32melse[0m[0;34m:[0m[0;34m[0m[0m
    [1;32m    329[0m                     mgr = self._init_ndarray(data, index, columns, dtype=dtype,
    [0;32m--> 330[0;31m                                              copy=copy)
    [0m[1;32m    331[0m             [0;32melse[0m[0;34m:[0m[0;34m[0m[0m
    [1;32m    332[0m                 [0mmgr[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0m_init_dict[0m[0;34m([0m[0;34m{[0m[0;34m}[0m[0;34m,[0m [0mindex[0m[0;34m,[0m [0mcolumns[0m[0;34m,[0m [0mdtype[0m[0;34m=[0m[0mdtype[0m[0;34m)[0m[0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/frame.py[0m in [0;36m_init_ndarray[0;34m(self, values, index, columns, dtype, copy)[0m
    [1;32m    481[0m             [0mvalues[0m [0;34m=[0m [0mmaybe_infer_to_datetimelike[0m[0;34m([0m[0mvalues[0m[0;34m)[0m[0;34m[0m[0m
    [1;32m    482[0m [0;34m[0m[0m
    [0;32m--> 483[0;31m         [0;32mreturn[0m [0mcreate_block_manager_from_blocks[0m[0;34m([0m[0;34m[[0m[0mvalues[0m[0;34m][0m[0;34m,[0m [0;34m[[0m[0mcolumns[0m[0;34m,[0m [0mindex[0m[0;34m][0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m    484[0m [0;34m[0m[0m
    [1;32m    485[0m     [0;34m@[0m[0mproperty[0m[0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36mcreate_block_manager_from_blocks[0;34m(blocks, axes)[0m
    [1;32m   4301[0m         [0mblocks[0m [0;34m=[0m [0;34m[[0m[0mgetattr[0m[0;34m([0m[0mb[0m[0;34m,[0m [0;34m'values'[0m[0;34m,[0m [0mb[0m[0;34m)[0m [0;32mfor[0m [0mb[0m [0;32min[0m [0mblocks[0m[0;34m][0m[0;34m[0m[0m
    [1;32m   4302[0m         [0mtot_items[0m [0;34m=[0m [0msum[0m[0;34m([0m[0mb[0m[0;34m.[0m[0mshape[0m[0;34m[[0m[0;36m0[0m[0;34m][0m [0;32mfor[0m [0mb[0m [0;32min[0m [0mblocks[0m[0;34m)[0m[0;34m[0m[0m
    [0;32m-> 4303[0;31m         [0mconstruction_error[0m[0;34m([0m[0mtot_items[0m[0;34m,[0m [0mblocks[0m[0;34m[[0m[0;36m0[0m[0;34m][0m[0;34m.[0m[0mshape[0m[0;34m[[0m[0;36m1[0m[0;34m:[0m[0;34m][0m[0;34m,[0m [0maxes[0m[0;34m,[0m [0me[0m[0;34m)[0m[0;34m[0m[0m
    [0m[1;32m   4304[0m [0;34m[0m[0m
    [1;32m   4305[0m [0;34m[0m[0m
    
    [0;32m/opt/conda/lib/python3.6/site-packages/pandas/core/internals.py[0m in [0;36mconstruction_error[0;34m(tot_items, block_shape, axes, e)[0m
    [1;32m   4278[0m         [0;32mraise[0m [0mValueError[0m[0;34m([0m[0;34m"Empty data passed with indices specified."[0m[0;34m)[0m[0;34m[0m[0m
    [1;32m   4279[0m     raise ValueError("Shape of passed values is {0}, indices imply {1}".format(
    [0;32m-> 4280[0;31m         passed, implied))
    [0m[1;32m   4281[0m [0;34m[0m[0m
    [1;32m   4282[0m [0;34m[0m[0m
    
    [0;31mValueError[0m: Shape of passed values is (1, 3), indices imply (1, 1)


If we look to the bottom, we can see that this is caused by an improper shape of the arrays we have passed into the `DataFrame` function.  We can trace our way back up through the code to see all the functions which were called in order to get to this error.  In this case, there were four called, `DataFrame, _init_ndarray, create_block_manager_from_blocks, construction_error`.  

Learning how to read Tracebacks and especially to figure out why simple bits of code are failing is an important part to becoming a good Python programmer.

### Exercise

Run the following bits of code in new cells and determine the error, fix the errors in a sensible way.


```python
# Example 1
float([1])

# Example 2
a = []
a[1]

# Example 3
pd.DataFrame(['one','two','three'],['test'])
```


```python
float([1])
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-17-b9231af3fe11> in <module>()
    ----> 1 float([1])
    

    TypeError: float() argument must be a string or a number, not 'list'



```python
float([1][0])
```




    1.0




```python
# Example 2
a = []
a[1]
```


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-19-b192eb7e0e62> in <module>()
          1 # Example 2
          2 a = []
    ----> 3 a[1]
    

    IndexError: list index out of range



```python
a = []
a
```




    []




```python
# Example 3
pd.DataFrame(['one','two','three'],['test'])
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in create_block_manager_from_blocks(blocks, axes)
       4295 
    -> 4296         mgr = BlockManager(blocks, axes)
       4297         mgr._consolidate_inplace()


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in __init__(self, blocks, axes, do_integrity_check, fastpath)
       2794         if do_integrity_check:
    -> 2795             self._verify_integrity()
       2796 


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in _verify_integrity(self)
       3005             if block._verify_integrity and block.shape[1:] != mgr_shape[1:]:
    -> 3006                 construction_error(tot_items, block.shape[1:], self.axes)
       3007         if len(self.items) != tot_items:


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in construction_error(tot_items, block_shape, axes, e)
       4279     raise ValueError("Shape of passed values is {0}, indices imply {1}".format(
    -> 4280         passed, implied))
       4281 


    ValueError: Shape of passed values is (1, 3), indices imply (1, 1)

    
    During handling of the above exception, another exception occurred:


    ValueError                                Traceback (most recent call last)

    <ipython-input-24-3a8cdd4c93bc> in <module>()
          1 # Example 3
    ----> 2 pd.DataFrame(['one','two','three'],['test'])
    

    /opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in __init__(self, data, index, columns, dtype, copy)
        328                 else:
        329                     mgr = self._init_ndarray(data, index, columns, dtype=dtype,
    --> 330                                              copy=copy)
        331             else:
        332                 mgr = self._init_dict({}, index, columns, dtype=dtype)


    /opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in _init_ndarray(self, values, index, columns, dtype, copy)
        481             values = maybe_infer_to_datetimelike(values)
        482 
    --> 483         return create_block_manager_from_blocks([values], [columns, index])
        484 
        485     @property


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in create_block_manager_from_blocks(blocks, axes)
       4301         blocks = [getattr(b, 'values', b) for b in blocks]
       4302         tot_items = sum(b.shape[0] for b in blocks)
    -> 4303         construction_error(tot_items, blocks[0].shape[1:], axes, e)
       4304 
       4305 


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in construction_error(tot_items, block_shape, axes, e)
       4278         raise ValueError("Empty data passed with indices specified.")
       4279     raise ValueError("Shape of passed values is {0}, indices imply {1}".format(
    -> 4280         passed, implied))
       4281 
       4282 


    ValueError: Shape of passed values is (1, 3), indices imply (1, 1)



```python
# Example 3
pd.DataFrame(['one','two','three'],[1, 2, 3])
```




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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>one</td>
    </tr>
    <tr>
      <th>2</th>
      <td>two</td>
    </tr>
    <tr>
      <th>3</th>
      <td>three</td>
    </tr>
  </tbody>
</table>
</div>



*Copyright &copy; 2018 The Data Incubator.  All rights reserved.*
