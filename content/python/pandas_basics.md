# Pandas Basics
15 May 2019

## Import modules


```python
import pandas as pd
import numpy as np
```

## Create an empty dataframe


```python
df = pd.DataFrame()
print(df)
```

    Empty DataFrame
    Columns: []
    Index: []


## Create a dataframe from list


```python
data = [1, 2, 3, 4]
df = pd.DataFrame(data)
print(df)
```

       0
    0  1
    1  2
    2  3
    3  4


## Create a dataframe from list of lists


```python
data = [[]]
```

## Create a dataframe from dict of ndarrays/lists


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
df = pd.DataFrame(data)
print(df)
```

      Student  Test Score
    0    Adam          95
    1    John          78
    2  Rachel          84
    3   Sarah          91


## Create a dataframe from list of dicts


```python
data = [{'a': 1, 'b': 2, 'c': 3}, {'a': 5, 'b': 6, 'c': 7, 'd': 8}]
df = pd.DataFrame(data)
print(df)
```

       a  b  c    d
    0  1  2  3  NaN
    1  5  6  7  8.0


**Note how NaN is appended in missing areas where no value is provided.**

## Selecting columns
Columns can be selected using bracket notation.


```python
data = [{'a': 1, 'b': 2, 'c': 3}, {'a': 6, 'b': 7, 'c': 8, 'd': 9}]
df = pd.DataFrame(data)
print(df['a'])
```

    0    1
    1    6
    Name: a, dtype: int64


## Adding columns
New columns can be added by passing in a Series.


```python
data = [{'a': 1, 'b': 2, 'c': 3}, {'a': 6, 'b': 7, 'c': 8, 'd': 9}]
df = pd.DataFrame(data)

# Add a new column by passing in a Series
df['e'] = pd.Series([5, 10], index=[0, 1])
print(df)
```

       a  b  c    d   e
    0  1  2  3  NaN   5
    1  6  7  8  9.0  10


## Deleting columns
Columns can be deleted using either `del` or the `pop()` function.


```python
data = [{'a': 1, 'b': 2, 'c': 3}, {'a': 6, 'b': 7, 'c': 8, 'd': 9}]
df = pd.DataFrame(data)

# Using del function
print('Deleting column \'d\' using DEL function:')
del df['d']
print(df)

# Using pop function
print('Deleting column \'c\' using POP function:')
df.pop('c')
print(df)
```

    Deleting column 'd' using DEL function:
       a  b  c
    0  1  2  3
    1  6  7  8
    Deleting column 'c' using POP function:
       a  b
    0  1  2
    1  6  7


## Row selection by label
Rows can be selected using the `loc()` function.


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
df = pd.DataFrame(data)
print(df.loc[0])
```

    Student       Adam
    Test Score      95
    Name: 0, dtype: object


## Row selection by integer location
Rows can be selected by their integer location using the `iloc()` function.


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
df = pd.DataFrame(data)
print(df.iloc[2])
```

    Student       Rachel
    Test Score        84
    Name: 2, dtype: object


## Row selection by slicing
Slice notation involves using a range, i.e. two indexes separated by a colon.


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
df = pd.DataFrame(data)
print(df[2:4])
```

      Student  Test Score
    2  Rachel          84
    3   Sarah          91


## Row addition
Rows can be added with the `append()` function.


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
data2 = {'Student': ['Michael'], 'Test Score': [97]}
df = pd.DataFrame(data)
df2 = pd.DataFrame(data2)
df = df.append(df2, ignore_index=True)
print(df)
```

       Student  Test Score
    0     Adam          95
    1     John          78
    2   Rachel          84
    3    Sarah          91
    4  Michael          97


## Row deletion
Rows can be deleted using the `drop()` function.


```python
data = {'Student': ['Adam', 'John', 'Rachel', 'Sarah'], 'Test Score': [95, 78, 84, 91]}
df = pd.DataFrame(data)
df = df.drop(0)
print(df)
```

      Student  Test Score
    1    John          78
    2  Rachel          84
    3   Sarah          91

