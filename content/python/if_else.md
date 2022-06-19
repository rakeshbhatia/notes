### Create a boolean variable that represents a current status.
* True if current status is 'active'
* False if current status is 'inactive'


```python
status = True
```

### If the status is active, print a statement.


```python
if status == True:
    print('The current status is active.')
```

    The current status is active.


### Change the value of status to inactive.


```python
status = False
```

### If the status is active, print a statement. If it is inactive, print a different statement.


```python
if status == True:
    print('The current status is active.')
else:
    print('The current status is inactive.')
```

    The current status is inactive.


### Change the value of status to unknown.


```python
status = None
```

### If the status is active, print a statement. If it is inactive, print a different statement. Otherwise, if the status is unknown, print a third statement.


```python
if status == True:
    print('The current status is active.')
elif status == False:
    print('The current status is inactive.')
elif status == None:
    print('The current status is unknown.')
```

    The current status is unknown.

