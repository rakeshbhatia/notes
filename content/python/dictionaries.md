# Dictionaries

## Fundamentals
* A python dictionary represents a mapping between key-value pairs
* Key-value pairs can represent any combination of valid python objects
* Dictionaries are mutable, i.e. they can be modified 

## Build a dictionary using brackets


```python
my_car = {'make': 'Nissan', 'model': 'Altima', 'year': 2010}
```

### View the dictionary


```python
my_car
```




    {'make': 'Nissan', 'model': 'Altima', 'year': 2010}



### Iterate through the dictionary
* Corresponding key and value can be retrieved simultaneously using `items()` method


```python
for key, value in my_car.items():
    print('My car\'s ' + key + ' is: ' + str(value))
```

    My car's make is: Nissan
    My car's model is: Altima
    My car's year is: 2010


## Build a dictionary using keys


```python
my_car = {}
my_car['make'] = 'Toyota'
my_car['model'] = 'Camry'
my_car['year'] = 2005
```

### View the dictionary


```python
my_car
```




    {'make': 'Toyota', 'model': 'Camry', 'year': 2005}



## Build a nested dictionary using brackets


```python
my_cars = {'make': ['Nissan', 'Toyota'], 'model': ['Altima', 'Camry'], 'year': [2005, 2010]}
```

### View the dictionary


```python
my_cars
```




    {'make': ['Nissan', 'Toyota'],
     'model': ['Altima', 'Camry'],
     'year': [2005, 2010]}



### Index the first item of the list nested in the 'year' key


```python
my_cars['year'][0]
```




    2005



### Remove a list element at a particular key


```python
del my_cars['make'][1]
```

### View the modified dictionary


```python
my_cars
```




    {'make': ['Nissan'], 'model': ['Altima', 'Camry'], 'year': [2005, 2010]}



### Remove an entire key and its value


```python
del my_cars['make']
```

### View the modified dictionary


```python
my_cars
```




    {'model': ['Altima', 'Camry'], 'year': [2005, 2010]}



### Empty all dictionary contents


```python
my_cars.clear()
```

### View the modified dictionary


```python
my_cars
```




    {}


