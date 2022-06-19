# Strings

## Basics

* Strings represent a sequence of text characters
* They are created by enclosing characters in quotes
    * Single quotes and double quotes are treated the same in Python

### Create a string with single quotes and assign it to a variable


```python
my_str = 'Hello World'

print(my_str)
```

    Hello World


### Create a string with double quotes and assign it to a variable


```python
my_str = "Hello World"

print(my_str)
```

    Hello World


## Accessing String Elements

* Square brackets and the index (or indices) are used together to access string elements
* A single character is also treated as a substring, or just a string of length one

### Access substrings of a string


```python
my_str = "Hello World"

print(my_str[0])

print(my_str[0:5])
```

    H
    Hello


## Modifying Strings
* A string can be modified by reassigning a string variable to a different string

### Create a new string using a substring of another string


```python
my_str = "Hello World"

new_str = my_str[:6] + "Universe"

print(new_str)
```

    Hello Universe


## String Formatting Operator
* The string formatting operator "%" can be used to set strings dynamically
* This operator is used to format a set of variables enclosed in a tuple
* The format variable is used in place of the variable element in the string
    * This variable can take on different types
        * "%s" is used for a string
        * "%d" is used for an integer

### Simple string formatting example


```python
print("My name is %s and I am %d years old." % ("John", 30))
```

    My name is John and I am 30 years old.



```python

```
