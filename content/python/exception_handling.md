# Exception Handling

## Try and Except
* Exception: a Python object representing an error
* To defend the program, suspicious code that may raise an exception can be placed in a `try` block
* After the `try` block, include an `except` block to handle the problem elegantly, i.e. print an error message

### Try to add a `string` and `int`


```python
try:
    # Do your main operations here
    'word' + 50
except Exception as e:
    # If an exception occurs, print an error message
    print('Error: ', e)
```

    Error:  must be str, not int


## Finally
* A `finally` block can be used together with `try` and `except` blocks
* It contains any code that must execute no matter what, even if an exception was raised


```python
try:
    # Do your main operations here
    'word' + 50
except Exception as e:
    # If an exception occurs, print an error message
    print('Error: ', e)
finally:
    print('Operation finished.')
```

    Error:  must be str, not int
    Operation finished.

