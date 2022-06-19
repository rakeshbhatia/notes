# Sets

## Basics
* A set is an unordered collection with no duplicate elements
* Uses: membership testing, eliminating duplicate entries
* A set can be initialized using curly braces or the `set()` function
    * Duplicate values will automatically be filtered out
* Note: to create an empty set, use set() as opposed to {}
    * {} will create an empty dictionary instead

### Create a set using curly braces


```python
pets = {'dog', 'cat', 'parrot', 'cat', 'dog', 'rabbit', 'hamster', 'dog'}

# Confirm that duplicates are removed
print(pets)
```

    {'parrot', 'hamster', 'dog', 'cat', 'rabbit'}


## Set Operations
### Create sets from two words


```python
a = set('calcariferous')
b = set('demagogue')
```

### Unique letters in a


```python
a
```




    {'a', 'c', 'e', 'f', 'i', 'l', 'o', 'r', 's', 'u'}



### Unique letters in b


```python
b
```




    {'a', 'd', 'e', 'g', 'm', 'o', 'u'}



### Letters in a but not in b


```python
a - b
```




    {'c', 'f', 'i', 'l', 'r', 's'}



### Letters in a or b or both


```python
a | b
```




    {'a', 'c', 'd', 'e', 'f', 'g', 'i', 'l', 'm', 'o', 'r', 's', 'u'}



### Letters in both a and b


```python
a & b
```




    {'a', 'e', 'o', 'u'}



### Letters in a or b but not both


```python
a ^ b
```




    {'c', 'd', 'f', 'g', 'i', 'l', 'm', 'r', 's'}


