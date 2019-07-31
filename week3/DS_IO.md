

```python
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
```


```python
import expectexception
```

# Importing and Exporting Data

<!-- requirement: data/sample.txt -->
<!-- requirement: data/csv_sample.txt -->
<!-- requirement: data/bad_csv.csv -->

So far we've only dealt with data that we have created within Python. Generating random data is helpful for testing out ideas, but we want to work with real data. Most often that data will be stored in a file, either locally on the computer or online. In this notebook we'll learn how to read and write data to files.

## Python file handles (`open`)

In Python we interact with files on disk using the commands `open` and `close`. We've included a file in the `data` folder called `sample.txt`. Let's open it and read its contents


```python
f = open('./data/sample.txt', 'r')
data = f.read()
f.close()

print(data)
print(f)
```

    Hello!
    Congratulations!
    You've read in data from a file.
    <_io.TextIOWrapper name='./data/sample.txt' mode='r' encoding='UTF-8'>


Notice that we `open` the file  and assign it to `f`, `read` the data from `f`, and then close `f`. What is `f`? It's called a **file handle**. It's an object that connects Python to the file we `open`. We `read` the data using this connection, and then once we're done with `close` the connection. It's a good habit to `close` a file handle once we're done with it, so usually we will do it automatically using Python's `with` keyword. 


```python
# f is automatically closed
# at the end of the body of the with statement
with open('./data/sample.txt', 'r') as f:
    print(f.read())

print(f)
```

    Hello!
    Congratulations!
    You've read in data from a file.
    <_io.TextIOWrapper name='./data/sample.txt' mode='r' encoding='UTF-8'>


We can also read individual lines of a file.


```python
with open('./data/sample.txt', 'r') as f:
    print(f.readline())
```

    Hello!
    



```python
with open('./data/sample.txt', 'r') as f:
    print(f.readlines())
```

    ['Hello!\n', 'Congratulations!\n', "You've read in data from a file."]


Writing data to files is very similar. The main difference is when we `open` the file, we will use the `'w'` flag instead of `'r'`.


```python
with open('./data/my_data.txt', 'w') as f:
    f.write('This is a new file.')
    f.write('I am practicing writing data to disk.')

with open('./data/my_data.txt', 'r') as f:
    my_data = f.read()

print(my_data)
```

    This is a new file.I am practicing writing data to disk.


No matter how often I execute the above cell, the same output gets printed. Opening the file with the `'w'` flag will overwrite the contents of the file. If we want to add to what is already in the file, we have to open the file with the `'a'` flag (`'a'` stands for _append_).


```python
with open('./data/my_data.txt', 'a') as f:
    f.write('\nAdding a new line to the file.')

with open('./data/my_data.txt', 'r') as f:
    my_data = f.read()

print(my_data)
```

    This is a new file.I am practicing writing data to disk.
    Adding a new line to the file.


We always need to be careful when writing to disk, because we could overwrite or alter data by accident. It is also easy to encounter errors when working with files, because we might not know ahead of time if the file we're trying to access exists, or we might mix up the `'r'`, `'w'`, and `'a'` flags.


```python
%%expect_exception IOError

# if a file doesn't exist
# we can't open it for reading
# (but we can open it for writing)

with open('./data/fail.txt', 'r') as f:
    f.read()
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mFileNotFoundError[0m                         Traceback (most recent call last)
    [0;32m<ipython-input-9-1f1268ec381a>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      4[0m [0;31m# (but we can open it for writing)[0m[0;34m[0m[0;34m[0m[0m
    [1;32m      5[0m [0;34m[0m[0m
    [0;32m----> 6[0;31m [0;32mwith[0m [0mopen[0m[0;34m([0m[0;34m'./data/fail.txt'[0m[0;34m,[0m [0;34m'r'[0m[0;34m)[0m [0;32mas[0m [0mf[0m[0;34m:[0m[0;34m[0m[0m
    [0m[1;32m      7[0m     [0mf[0m[0;34m.[0m[0mread[0m[0;34m([0m[0;34m)[0m[0;34m[0m[0m
    
    [0;31mFileNotFoundError[0m: [Errno 2] No such file or directory: './data/fail.txt'



```python
%%expect_exception IOError

# we can't read a file open for writing

with open('./data/fail.txt', 'w') as f:
    f.read()
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mUnsupportedOperation[0m                      Traceback (most recent call last)
    [0;32m<ipython-input-10-2396ab194fe8>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      3[0m [0;34m[0m[0m
    [1;32m      4[0m [0;32mwith[0m [0mopen[0m[0;34m([0m[0;34m'./data/fail.txt'[0m[0;34m,[0m [0;34m'w'[0m[0;34m)[0m [0;32mas[0m [0mf[0m[0;34m:[0m[0;34m[0m[0m
    [0;32m----> 5[0;31m     [0mf[0m[0;34m.[0m[0mread[0m[0;34m([0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;31mUnsupportedOperation[0m: not readable



```python
%%expect_exception IOError

# and we can't write to a file open for reading

with open('./data/sample.txt', 'r') as f:
    f.write('This will fail')
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mUnsupportedOperation[0m                      Traceback (most recent call last)
    [0;32m<ipython-input-11-415e83617ea3>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      3[0m [0;34m[0m[0m
    [1;32m      4[0m [0;32mwith[0m [0mopen[0m[0;34m([0m[0;34m'./data/sample.txt'[0m[0;34m,[0m [0;34m'r'[0m[0;34m)[0m [0;32mas[0m [0mf[0m[0;34m:[0m[0;34m[0m[0m
    [0;32m----> 5[0;31m     [0mf[0m[0;34m.[0m[0mwrite[0m[0;34m([0m[0;34m'This will fail'[0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;31mUnsupportedOperation[0m: not writable


Can we prevent some of these errors? How do we find out what files are on disk?

## `os` module

Python has a module for navigating the computer's file system called `os`. There are many useful tools in the `os` module, but there are two functions that are most useful for finding files.


```python
import os

# list the contents of the current directory
# ('.' refers to the current directory)
os.listdir('.')
```




    ['data',
     'DS_IO.ipynb',
     'miniprojects',
     'DS_Pandas.ipynb',
     '.ipynb_checkpoints',
     '.done',
     'DS_IO-Copy1.ipynb',
     'DS_Data_Munging.ipynb',
     'DS_Intro_Statistics.ipynb',
     'DS_SQL.ipynb',
     'DS_Basic_DS_Modules.ipynb',
     'DS_Classes_and_ORM.ipynb']



The command `listdir` is the simpler of the two functions we'll cover. It simply lists the contents of the directory path we specify. When we pass `'.'` as the argument, `listdir` will look in the current directory. It lists all the Jupyter notebooks we're using for the course, as well as the `data` subdirectory. We could find out what's in the `data` subdirectory by looking in `'./data'`.


```python
os.listdir('./data')
```




    ['customers.csv',
     'csv_sample.txt',
     'sample.txt',
     'orders.csv',
     'my_data.txt',
     'fail.txt',
     'yelp.json.gz',
     'bad_csv.csv',
     'products.csv',
     'PEP_2016_PEPANNRES.csv']



What if we wanted to find all the files and subdirectories below a directory somewhere on our computer? With `listdir` we only see the files and subdirectories under the particular directory we're looking in. We cannot use `listdir` to automatically search through subdirectories. For this we need to use `walk`, which "walks" through all the subdirectories below our chosen directory. We won't cover `walk` in this course, but it's one of the very useful tools (along with the `os.path` sub-module) for working with files in Python, particularly if you are working with many different data files at once.

## CSV files

One of the simplest and most common formats for saving data is as comma-separated values (CSV).


```python
with open('./data/csv_sample.txt', 'r') as f:
    csv = f.read()

print(csv)
```

    index,name,age
    0,Dylan,28
    1,Terrence,54
    2,Mya,31
    


This format is often used to represent tables of data. Usually a CSV will have rows (separated by newline characters, `'\n'`) and columns (separated by commas). Otherwise they are no different from any other text file. We can use the special formatting of a CSV to create a list of lists representing the table.


```python
list_table = []
with open('./data/csv_sample.txt', 'r') as f:
    for line in f.readlines():
        list_table.append(line.strip().split(','))

list_table
```




    [['index', 'name', 'age'],
     ['0', 'Dylan', '28'],
     ['1', 'Terrence', '54'],
     ['2', 'Mya', '31']]



However, we can work with tabular data much more easily in a Pandas DataFrame. Pandas provides a `read_csv` method to read the data directly into a DataFrame.


```python
import pandas as pd

df = pd.read_csv('./data/csv_sample.txt', index_col=0)
df
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
      <th>name</th>
      <th>age</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dylan</td>
      <td>28</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Terrence</td>
      <td>54</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mya</td>
      <td>31</td>
    </tr>
  </tbody>
</table>
</div>



The `read_csv` method is very flexible to deal with the formatting of different data sets. Some data sets will include column headers while others may not. Some data sets will include an index while others may not. Some data sets may have values separated by tabs, semicolons, or other characters instead of commas. There are options in the `read_csv` method for dealing with all of these. You can read about them in the [Pandas documentation on `read_csv`](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_csv.html). We'll also discuss it further in the [Pandas notebook](DS_Pandas.ipynb).


```python
# an example of downloading
# and importing real data using `read_csv`

if 'factbook.csv' not in os.listdir('./data/'):
    !wget -P ./data/ https://perso.telecom-paristech.fr/eagan/class/igr204/data/factbook.csv

countries = pd.read_csv('./data/factbook.csv', delimiter=';', skiprows=[1])
countries.head()
```

    --2019-07-23 23:38:31--  https://perso.telecom-paristech.fr/eagan/class/igr204/data/factbook.csv
    Resolving perso.telecom-paristech.fr (perso.telecom-paristech.fr)... 137.194.2.165, 2001:660:330f:2::a5
    Connecting to perso.telecom-paristech.fr (perso.telecom-paristech.fr)|137.194.2.165|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 64856 (63K) [text/csv]
    Saving to: ‘./data/factbook.csv’
    
    factbook.csv        100%[===================>]  63.34K  --.-KB/s    in 0.08s   
    
    2019-07-23 23:38:33 (796 KB/s) - ‘./data/factbook.csv’ saved [64856/64856]
    





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
      <th>Country</th>
      <th>Area(sq km)</th>
      <th>Birth rate(births/1000 population)</th>
      <th>Current account balance</th>
      <th>Death rate(deaths/1000 population)</th>
      <th>Debt - external</th>
      <th>Electricity - consumption(kWh)</th>
      <th>Electricity - production(kWh)</th>
      <th>Exports</th>
      <th>GDP</th>
      <th>...</th>
      <th>Oil - production(bbl/day)</th>
      <th>Oil - proved reserves(bbl)</th>
      <th>Population</th>
      <th>Public debt(% of GDP)</th>
      <th>Railways(km)</th>
      <th>Reserves of foreign exchange &amp; gold</th>
      <th>Telephones - main lines in use</th>
      <th>Telephones - mobile cellular</th>
      <th>Total fertility rate(children born/woman)</th>
      <th>Unemployment rate(%)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>647500</td>
      <td>47.02</td>
      <td>NaN</td>
      <td>20.75</td>
      <td>8.000000e+09</td>
      <td>6.522000e+08</td>
      <td>5.400000e+08</td>
      <td>4.460000e+08</td>
      <td>2.150000e+10</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>29928987.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>33100.0</td>
      <td>15000.0</td>
      <td>6.75</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Akrotiri</td>
      <td>123</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Albania</td>
      <td>28748</td>
      <td>15.08</td>
      <td>-5.040000e+08</td>
      <td>5.12</td>
      <td>1.410000e+09</td>
      <td>6.760000e+09</td>
      <td>5.680000e+09</td>
      <td>5.524000e+08</td>
      <td>1.746000e+10</td>
      <td>...</td>
      <td>2000.0</td>
      <td>1.855000e+08</td>
      <td>3563112.0</td>
      <td>NaN</td>
      <td>447.0</td>
      <td>1.206000e+09</td>
      <td>255000.0</td>
      <td>1100000.0</td>
      <td>2.04</td>
      <td>14.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Algeria</td>
      <td>2381740</td>
      <td>17.13</td>
      <td>1.190000e+10</td>
      <td>4.60</td>
      <td>2.190000e+10</td>
      <td>2.361000e+10</td>
      <td>2.576000e+10</td>
      <td>3.216000e+10</td>
      <td>2.123000e+11</td>
      <td>...</td>
      <td>1200000.0</td>
      <td>1.187000e+10</td>
      <td>32531853.0</td>
      <td>37.4</td>
      <td>3973.0</td>
      <td>4.355000e+10</td>
      <td>2199600.0</td>
      <td>1447310.0</td>
      <td>1.92</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>American Samoa</td>
      <td>199</td>
      <td>23.13</td>
      <td>NaN</td>
      <td>3.33</td>
      <td>NaN</td>
      <td>1.209000e+08</td>
      <td>1.300000e+08</td>
      <td>3.000000e+07</td>
      <td>5.000000e+08</td>
      <td>...</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>57881.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>15000.0</td>
      <td>2377.0</td>
      <td>3.25</td>
      <td>6.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 45 columns</p>
</div>




```python
# we can also use pandas to write CSV
# using the DataFrame's to_csv method

pd.DataFrame({'a': [0, 3, 10], 'b': [True, True, False]}).to_csv('./data/pd_write.csv')

pd.read_csv('./data/pd_write.csv', index_col=0)
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
      <th>a</th>
      <th>b</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Sometimes a CSV won't be perfect. For example, maybe different rows have different numbers of commas. This makes it difficult to interpret the contents of the file as a table.


```python
# the 3rd line only has 2 "columns"

!cat ./data/bad_csv.csv
```

    index,name,age
    0,Dylan,27
    1,54
    2,Mya,31


```python
# what happens if we try to read this
# into a DataFrame using read_csv?

pd.read_csv('./data/bad_csv.csv', index_col = 0)
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
      <th>name</th>
      <th>age</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dylan</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>54</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mya</td>
      <td>31.0</td>
    </tr>
  </tbody>
</table>
</div>



Pandas' `read_csv` method will do its best to construct a table out of a poorly formatted CSV, but it may make mistakes. For example, 54 was interpreted as a name instead of as an age, because there were only 2 columns in that line of the file. Data sets will often contain mistakes like bad formatting, missing data, or typos.

**Question:** How could we fix the badly formatted CSV so it would work with `read_csv`?

## JSON

JSON stands for JavaScript Object Notation. JavaScript is a common language for creating web applications, and JSON files are used to collect and transmit information between JavaScript applications. As a result, a lot of data on the internet exists in the JSON file format. For example, Twitter and Google Maps use JSON.

A JSON file is essentially a data structure built out of nested dictionaries and lists. Let's make our own example and then we'll examine an example downloaded from the internet.


```python
book1 = {'title': 'The Prophet',
         'author': 'Khalil Gibran',
         'genre': 'poetry',
         'tags': ['religion', 'spirituality', 'philosophy', 'Lebanon', 'Arabic', 'Middle East'],
         'book_id': '811.19',
         'copies': [{'edition_year': 1996,
                     'checkouts': 486,
                     'borrowed': False},
                    {'edition_year': 1996,
                     'checkouts': 443,
                     'borrowed': False}]
         }
         
book2 = {'title': 'The Little Prince',
         'author': 'Antoine de Saint-Exupery',
         'genre': 'children',
         'tags': ['fantasy', 'France', 'philosophy', 'illustrated', 'fable'],
         'id': '843.912',
         'copies': [{'edition_year': 1983,
                     'checkouts': 634,
                     'borrowed': True,
                     'due_date': '2017/02/02'},
                    {'edition_year': 2015,
                     'checkouts': 41,
                     'borrowed': False}]
         }

library = [book1, book2]
library
```




    [{'title': 'The Prophet',
      'author': 'Khalil Gibran',
      'genre': 'poetry',
      'tags': ['religion',
       'spirituality',
       'philosophy',
       'Lebanon',
       'Arabic',
       'Middle East'],
      'book_id': '811.19',
      'copies': [{'edition_year': 1996, 'checkouts': 486, 'borrowed': False},
       {'edition_year': 1996, 'checkouts': 443, 'borrowed': False}]},
     {'title': 'The Little Prince',
      'author': 'Antoine de Saint-Exupery',
      'genre': 'children',
      'tags': ['fantasy', 'France', 'philosophy', 'illustrated', 'fable'],
      'id': '843.912',
      'copies': [{'edition_year': 1983,
        'checkouts': 634,
        'borrowed': True,
        'due_date': '2017/02/02'},
       {'edition_year': 2015, 'checkouts': 41, 'borrowed': False}]}]



We have two books in our `library`. Both books have some common properties: a title, an author, an id, and tags. Each book can have several tags, so we store that data as a list. Additionally, there can be multiple copies of each book, and each copy also has some unique information like the year it was printed and how many times it's been checked out. Notice that if a book is checked out, it also has a due date. It's convenient to store the information about the multiple copies as a list of dictionaries within the dictionary about the book, because every copy shares the same title, author, etc.

This structure is typical of JSON files. It has the advantage of reducing redundancy of data. We only store the author and title once, even though there are multiple copies of the book. Also, we don't store a due date for copies that aren't checked out.

If we were to put this data in a table, we would have to duplicate a lot of information. Also, since only one copy in our library is checked out, we also have a column with a lot of missing data.

|index|title|author|id|genre|tags|edition_year|checkouts|borrowed|due_date|
|:---:|:---:|:----:|::|:---:|:--:|:----------:|:-------:|:------:|:------:|
|0|The Prophet|Khalil Gibran|811.19|poetry|religion, spirituality, philosophy, Lebanon, Arabic, Middle East|1996|486|False|Null|
|1|The Prophet|Khalil Gibran|811.19|poetry|religion, spirituality, philosophy, Lebanon, Arabic, Middle East|1996|443|False|Null|
|2|The Little Prince|Antoine de Saint-Exupery|843.912|children|fantasy, France, philosophy, illustrated, fable|1983|634|True|2017/02/02|
|3|The Little Prince|Antoine de Saint-Exupery|843.912|children|fantasy, France, philosophy, illustrated, fable|2015|41|False|Null|

This is very wasteful. Since JSON files are meant to be shared quickly over the internet, it is important that they are small to reduce the amount of resources needed to store and transmit them.

We can write our `library` to disk using the `json` module.


```python
import json

with open('./data/library.json', 'w') as f:
    json.dump(library, f, indent=2)
```


```python
!cat ./data/library.json
```

    [
      {
        "title": "The Prophet",
        "author": "Khalil Gibran",
        "genre": "poetry",
        "tags": [
          "religion",
          "spirituality",
          "philosophy",
          "Lebanon",
          "Arabic",
          "Middle East"
        ],
        "book_id": "811.19",
        "copies": [
          {
            "edition_year": 1996,
            "checkouts": 486,
            "borrowed": false
          },
          {
            "edition_year": 1996,
            "checkouts": 443,
            "borrowed": false
          }
        ]
      },
      {
        "title": "The Little Prince",
        "author": "Antoine de Saint-Exupery",
        "genre": "children",
        "tags": [
          "fantasy",
          "France",
          "philosophy",
          "illustrated",
          "fable"
        ],
        "id": "843.912",
        "copies": [
          {
            "edition_year": 1983,
            "checkouts": 634,
            "borrowed": true,
            "due_date": "2017/02/02"
          },
          {
            "edition_year": 2015,
            "checkouts": 41,
            "borrowed": false
          }
        ]
      }
    ]


```python
with open('./data/library.json', 'r') as f:
    reloaded_library = json.load(f)

reloaded_library
```




    [{'title': 'The Prophet',
      'author': 'Khalil Gibran',
      'genre': 'poetry',
      'tags': ['religion',
       'spirituality',
       'philosophy',
       'Lebanon',
       'Arabic',
       'Middle East'],
      'book_id': '811.19',
      'copies': [{'edition_year': 1996, 'checkouts': 486, 'borrowed': False},
       {'edition_year': 1996, 'checkouts': 443, 'borrowed': False}]},
     {'title': 'The Little Prince',
      'author': 'Antoine de Saint-Exupery',
      'genre': 'children',
      'tags': ['fantasy', 'France', 'philosophy', 'illustrated', 'fable'],
      'id': '843.912',
      'copies': [{'edition_year': 1983,
        'checkouts': 634,
        'borrowed': True,
        'due_date': '2017/02/02'},
       {'edition_year': 2015, 'checkouts': 41, 'borrowed': False}]}]




```python
# note that if we loaded it in without JSON
# the file would be interpreted as plain text

with open('./data/library.json', 'r') as f:
    library_string = f.read()

# this isn't what we want
library_string
```




    '[\n  {\n    "title": "The Prophet",\n    "author": "Khalil Gibran",\n    "genre": "poetry",\n    "tags": [\n      "religion",\n      "spirituality",\n      "philosophy",\n      "Lebanon",\n      "Arabic",\n      "Middle East"\n    ],\n    "book_id": "811.19",\n    "copies": [\n      {\n        "edition_year": 1996,\n        "checkouts": 486,\n        "borrowed": false\n      },\n      {\n        "edition_year": 1996,\n        "checkouts": 443,\n        "borrowed": false\n      }\n    ]\n  },\n  {\n    "title": "The Little Prince",\n    "author": "Antoine de Saint-Exupery",\n    "genre": "children",\n    "tags": [\n      "fantasy",\n      "France",\n      "philosophy",\n      "illustrated",\n      "fable"\n    ],\n    "id": "843.912",\n    "copies": [\n      {\n        "edition_year": 1983,\n        "checkouts": 634,\n        "borrowed": true,\n        "due_date": "2017/02/02"\n      },\n      {\n        "edition_year": 2015,\n        "checkouts": 41,\n        "borrowed": false\n      }\n    ]\n  }\n]'




```python
# Pandas can also read_json
# notice how it constructs the table
# does it represent the data well?

pd.read_json('./data/library.json')
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
      <th>author</th>
      <th>book_id</th>
      <th>copies</th>
      <th>genre</th>
      <th>id</th>
      <th>tags</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Khalil Gibran</td>
      <td>811.19</td>
      <td>[{'edition_year': 1996, 'checkouts': 486, 'bor...</td>
      <td>poetry</td>
      <td>NaN</td>
      <td>[religion, spirituality, philosophy, Lebanon, ...</td>
      <td>The Prophet</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Antoine de Saint-Exupery</td>
      <td>NaN</td>
      <td>[{'edition_year': 1983, 'checkouts': 634, 'bor...</td>
      <td>children</td>
      <td>843.912</td>
      <td>[fantasy, France, philosophy, illustrated, fable]</td>
      <td>The Little Prince</td>
    </tr>
  </tbody>
</table>
</div>




```python
# and to_json
df.to_json('./data/example_df.json')

!head ./data/example_df.json
```

    {"name":{"0":"Dylan","1":"Terrence","2":"Mya"},"age":{"0":28,"1":54,"2":31}}

We can download JSON files many ways. Sometimes we will download it manually, but we can also use `wget` like we did for the CSV example. Often we'll connect to a website's API which will respond using JSON.

Panda's `read_json` method is capable of connecting directly to a URL (whether it's the address of a JSON file or an API connection) and reading the JSON without saving the file to our computer.


```python
pd.read_json('https://api.github.com/repos/pydata/pandas/issues?per_page=5')
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
      <th>assignee</th>
      <th>assignees</th>
      <th>author_association</th>
      <th>body</th>
      <th>closed_at</th>
      <th>comments</th>
      <th>comments_url</th>
      <th>created_at</th>
      <th>events_url</th>
      <th>html_url</th>
      <th>...</th>
      <th>milestone</th>
      <th>node_id</th>
      <th>number</th>
      <th>pull_request</th>
      <th>repository_url</th>
      <th>state</th>
      <th>title</th>
      <th>updated_at</th>
      <th>url</th>
      <th>user</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>[]</td>
      <td>MEMBER</td>
      <td>- [X] closes #27546\r\n- [ ] tests added / pas...</td>
      <td>NaT</td>
      <td>0</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>2019-07-23 23:03:08</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>https://github.com/pandas-dev/pandas/pull/27550</td>
      <td>...</td>
      <td>None</td>
      <td>MDExOlB1bGxSZXF1ZXN0MzAwNTAyMDg2</td>
      <td>27550</td>
      <td>{'url': 'https://api.github.com/repos/pandas-d...</td>
      <td>https://api.github.com/repos/pandas-dev/pandas</td>
      <td>open</td>
      <td>Revert CI changes from 27542</td>
      <td>2019-07-23 23:04:12</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>{'login': 'WillAyd', 'id': 609873, 'node_id': ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>[]</td>
      <td>CONTRIBUTOR</td>
      <td>\r\n- [ x ] closes #27548 \r\n- [ x ] tests ad...</td>
      <td>NaT</td>
      <td>1</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>2019-07-23 21:33:15</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>https://github.com/pandas-dev/pandas/pull/27549</td>
      <td>...</td>
      <td>{'url': 'https://api.github.com/repos/pandas-d...</td>
      <td>MDExOlB1bGxSZXF1ZXN0MzAwNDc5OTU3</td>
      <td>27549</td>
      <td>{'url': 'https://api.github.com/repos/pandas-d...</td>
      <td>https://api.github.com/repos/pandas-dev/pandas</td>
      <td>open</td>
      <td>BUG: Fix interpolate ValueError for datetime64...</td>
      <td>2019-07-23 22:35:50</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>{'login': 'alorenzo175', 'id': 4625457, 'node_...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>[]</td>
      <td>CONTRIBUTOR</td>
      <td>#### Code Sample, a copy-pastable example if p...</td>
      <td>NaT</td>
      <td>0</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>2019-07-23 21:05:57</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>https://github.com/pandas-dev/pandas/issues/27548</td>
      <td>...</td>
      <td>{'url': 'https://api.github.com/repos/pandas-d...</td>
      <td>MDU6SXNzdWU0NzE5NDgwMzU=</td>
      <td>27548</td>
      <td>NaN</td>
      <td>https://api.github.com/repos/pandas-dev/pandas</td>
      <td>open</td>
      <td>pd.Series.interpolate() ValueError for some me...</td>
      <td>2019-07-23 22:24:23</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>{'login': 'alorenzo175', 'id': 4625457, 'node_...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>[]</td>
      <td>CONTRIBUTOR</td>
      <td>#### Code Sample\r\n\r\n```python\r\nimport pa...</td>
      <td>NaT</td>
      <td>0</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>2019-07-23 20:47:40</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>https://github.com/pandas-dev/pandas/issues/27547</td>
      <td>...</td>
      <td>None</td>
      <td>MDU6SXNzdWU0NzE5MzY1NTA=</td>
      <td>27547</td>
      <td>NaN</td>
      <td>https://api.github.com/repos/pandas-dev/pandas</td>
      <td>open</td>
      <td>read.sql fails with adodbapi connection</td>
      <td>2019-07-23 20:49:24</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>{'login': 'ParfaitG', 'id': 13602663, 'node_id...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>[]</td>
      <td>CONTRIBUTOR</td>
      <td>https://github.com/pandas-dev/pandas/pull/2754...</td>
      <td>NaT</td>
      <td>2</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>2019-07-23 19:02:31</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>https://github.com/pandas-dev/pandas/issues/27546</td>
      <td>...</td>
      <td>None</td>
      <td>MDU6SXNzdWU0NzE4Nzc0NDU=</td>
      <td>27546</td>
      <td>NaN</td>
      <td>https://api.github.com/repos/pandas-dev/pandas</td>
      <td>open</td>
      <td>Cleanup CI environment files after #27542</td>
      <td>2019-07-23 23:06:09</td>
      <td>https://api.github.com/repos/pandas-dev/pandas...</td>
      <td>{'login': 'TomAugspurger', 'id': 1312546, 'nod...</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>
</div>



## Compressed files (Gzip)

Another way we save storage and network resources is by using **compression**. Many times data sets will contain patterns that can be used to reduce the amount of space needed to store the information.

A simple example is the following list of numbers: 10, 10, 10, 2, 3, 3, 3, 3, 3, 50, 50, 1, 1, 50, 10, 10, 10, 10

Rather than writing out the full list of numbers (18 integers), we can represent the same information with only 14 numbers: (3, 10), (1, 2), (5, 3), (2, 50), (2, 1), (1, 50), (4, 10)

Here the first number in each pair is the number of repetitions, and the second number in the pair is the actual value. We've successfully reduced the amount of numbers we need to represent the same data. Most forms of compression use a similar idea, although actual implementations are usually more complex.

In the world of data science, the most common compression is Gzip (which uses the [deflate algorithm](http://www.infinitepartitions.com/art001.html)). Gzip files end with the extension `.gz`.


```python
!wget -P ./data/ https://archive.org/stream/TheEpicofGilgamesh_201606/eog_djvu.txt
```

    --2019-07-23 23:39:35--  https://archive.org/stream/TheEpicofGilgamesh_201606/eog_djvu.txt
    Resolving archive.org (archive.org)... 207.241.224.2
    Connecting to archive.org (archive.org)|207.241.224.2|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [text/html]
    Saving to: ‘./data/eog_djvu.txt’
    
    eog_djvu.txt            [ <=>                ] 159.76K  --.-KB/s    in 0.1s    
    
    2019-07-23 23:39:35 (1.20 MB/s) - ‘./data/eog_djvu.txt’ saved [163595]
    



```python
import gzip

with open('./data/eog_djvu.txt', 'r') as f:
    text = f.read()

with gzip.open('./data/eog_djvu.txt.gz', 'wb') as f:
    f.write(text.encode('utf-8'))

!ls -lh ./data/eog*
```

    -rw-r--r-- 1 jovyan users 160K Jul 23 23:39 ./data/eog_djvu.txt
    -rw-r--r-- 1 jovyan users  46K Jul 23 23:39 ./data/eog_djvu.txt.gz


We were able to compress the text of The Epic of Gilgamesh to a third of its original size! Remember that compression depends on patterns in the data. Language has a lot of patterns, but what would happen if we scrambled all the letters in the text?


```python
import numpy as np

with gzip.open('./data/eog_djvu_scrambled.txt.gz', 'wb') as f:
    f.write(np.random.permutation(list(text)))

!ls -lh ./data/eog*
```

    -rw-r--r-- 1 jovyan users 160K Jul 23 23:39 ./data/eog_djvu.txt
    -rw-r--r-- 1 jovyan users  46K Jul 23 23:39 ./data/eog_djvu.txt.gz
    -rw-r--r-- 1 jovyan users 136K Jul 23 23:39 ./data/eog_djvu_scrambled.txt.gz


The scrambled version only compressed to two-thirds the size of the original. Compression won't perform very well on random data. Compression also doesn't work very well on data that is already small.


```python
short_text = 'Hello'

with open('./data/short_text.txt', 'w') as f:
    f.write(short_text)

with gzip.open('./data/short_text.txt.gz', 'wb') as f:
    f.write(short_text.encode('utf-8'))

!ls -lh ./data/short_text*
```

    -rw-r--r-- 1 jovyan users  5 Jul 23 23:39 ./data/short_text.txt
    -rw-r--r-- 1 jovyan users 40 Jul 23 23:39 ./data/short_text.txt.gz


The compressed file is bigger than the plain text! That's because the compressed file includes a header, which takes up a small amount of extra space. Also, since the text is so short, it's not possible to use patterns to represent the text more efficiently. Therefore we usually save compression for large files.

You may have noticed that when we write Gzip files, we have been using a `'wb'` flag instead of a plain `'w'` flag. This is because Gzip is not plain text. When compressing the file we write _binary_ files. The files are not readable as plain text.


```python
# we have to uncompress the file
# before we can read it

!cat ./data/short_text.txt.gz
```

    �Ț7]�short_text.txt �H��� ����   

We should only use `'w'` for plain text files (which includes CSV and JSON). Using `'w'` instead of `'wb'` for Gzip files, or other files which are not plain text (e.g. images), could damage the file.

## Serialization (`pickle`)

Often we will want to save our work in Python and come back to it later. However, that work might be a machine learning model or some other complex object in Python. How do we save complex Python objects? Python has a module for this purpose called `pickle`. We can use `pickle` to write a binary file that contains all the information about a Python object. Later we can load that pickle file and reconstruct the object in Python.


```python
pickle_example = ['hello', {'a': 23, 'b': True}, (1, 2, 3), [['dogs', 'cats'], None]]
```


```python
%%expect_exception TypeError

# we can't save this as text
with open('./data/pickle_example.txt', 'w') as f:
    f.write(pickle_example)
```

    [0;31m---------------------------------------------------------------------------[0m
    [0;31mTypeError[0m                                 Traceback (most recent call last)
    [0;32m<ipython-input-35-dc175613edd9>[0m in [0;36m<module>[0;34m()[0m
    [1;32m      2[0m [0;31m# we can't save this as text[0m[0;34m[0m[0;34m[0m[0m
    [1;32m      3[0m [0;32mwith[0m [0mopen[0m[0;34m([0m[0;34m'./data/pickle_example.txt'[0m[0;34m,[0m [0;34m'w'[0m[0;34m)[0m [0;32mas[0m [0mf[0m[0;34m:[0m[0;34m[0m[0m
    [0;32m----> 4[0;31m     [0mf[0m[0;34m.[0m[0mwrite[0m[0;34m([0m[0mpickle_example[0m[0;34m)[0m[0;34m[0m[0m
    [0m
    [0;31mTypeError[0m: write() argument must be str, not list



```python
import pickle

# we can save it as a pickle
with open('./data/pickle_example.pkl', 'wb') as f:
    pickle.dump(pickle_example, f)

with open('./data/pickle_example.pkl', 'rb') as f:
    reloaded_example = pickle.load(f)

reloaded_example
```




    ['hello', {'a': 23, 'b': True}, (1, 2, 3), [['dogs', 'cats'], None]]




```python
# the reloaded example is the same as the original

reloaded_example == pickle_example
```




    True



Pickle is an important tool for data scientists. Data processing and training machine learning models can take a long time, and it is useful to save checkpoints.

Pandas also has `to_pickle` and `read_pickle` methods.

## NumPy file formats

NumPy also has methods for saving and loading data. They are straightforward to use. You may encounter these when working with certain machine learning libraries that require data be stored in NumPy arrays. NumPy arrays are also often used when working with image data.


```python
sample_array = np.random.random((4, 4))
print(sample_array)
```

    [[1.15101439e-01 7.56843401e-02 3.27703247e-02 5.10796008e-01]
     [9.08183143e-01 4.13487506e-01 6.63167331e-02 8.43464983e-01]
     [4.43216987e-01 6.38575731e-01 1.25723732e-03 4.48233785e-01]
     [7.95726251e-04 4.10639564e-01 3.60672338e-01 8.13265871e-01]]



```python
# to save as plain text
np.savetxt('./data/sample_array.txt', sample_array)
```


```python
!cat ./data/sample_array.txt
```

    1.151014387735269651e-01 7.568434007615110204e-02 3.277032473522956124e-02 5.107960075302095948e-01
    9.081831434398075498e-01 4.134875062305792826e-01 6.631673310787011832e-02 8.434649826487297108e-01
    4.432169873585417585e-01 6.385757309924426917e-01 1.257237315254067234e-03 4.482337849146016406e-01
    7.957262511946172623e-04 4.106395640743909503e-01 3.606723376271812054e-01 8.132658714747813544e-01



```python
print(np.loadtxt('./data/sample_array.txt'))
```

    [[1.15101439e-01 7.56843401e-02 3.27703247e-02 5.10796008e-01]
     [9.08183143e-01 4.13487506e-01 6.63167331e-02 8.43464983e-01]
     [4.43216987e-01 6.38575731e-01 1.25723732e-03 4.48233785e-01]
     [7.95726251e-04 4.10639564e-01 3.60672338e-01 8.13265871e-01]]



```python
# to save as compressed binary
np.save('./data/sample_array.npy', sample_array)
```


```python
!cat ./data/sample_array.npy
```

    �NUMPY v {'descr': '<f8', 'fortran_order': False, 'shape': (4, 4), }                                                          
    �=A�Iw�? Hr�`�?���EǠ?��h�pX�?FG{��?�'$M�v�?�8�'"��?"�yF���?*$oȪ]�?�7_6o�? ��I<�T?����ܯ�? ��	J?H*�*�G�?,��mA�?��&F�?


```python
print(np.load('./data/sample_array.npy'))
```

    [[1.15101439e-01 7.56843401e-02 3.27703247e-02 5.10796008e-01]
     [9.08183143e-01 4.13487506e-01 6.63167331e-02 8.43464983e-01]
     [4.43216987e-01 6.38575731e-01 1.25723732e-03 4.48233785e-01]
     [7.95726251e-04 4.10639564e-01 3.60672338e-01 8.13265871e-01]]


## Topics used by not discussed:
- BASH commands (!)
- `wget`
- `str.split()`
- APIs

*Copyright &copy; 2017 The Data Incubator.  All rights reserved.*
