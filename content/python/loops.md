# **Loops**

## **For Loop**

### Create a list of numbers


```python
# Create a list of numbers
nums = [1, 2, 3, 4, 5]
print(nums)
```

    [1, 2, 3, 4, 5]


### Use a `for` loop to print each element


```python
# Loop through the list and print out each number
for i in range(len(nums)):
    print(nums[i])
```

    1
    2
    3
    4
    5


## **While Loop**

### Initialize `i`


```python
i = 0
```

### Use a `while` loop to print each element
* Manually increment `i` variable inside loop
* Loop will run until it meets a stopping condition
* Stopping condition: when `i` is equal to the length of the list


```python
while i != len(nums):
    print(nums[i])
    i += 1
```

    1
    2
    3
    4
    5


## **Break Statement**

### Initialize `i`


```python
i = 0
```

### Use `break` to exit the loop early once `i` reaches a particular value
* Loop will terminate once it encounters `break` statement


```python
while i != len(nums):
    if i == 3:
        'Exiting the loop.'
        break
    print(nums[i])
    i += 1
```

    1
    2
    3


## Continue Statement

### Initialize `i`


```python
i = 0
```

### Use `continue` to skip over a particular index
* Skips over every subsequent line of code in the current loop iteration
* The program jumps to the next iteration of the loop
* Care must be taken to avoid infinite loops


```python
while i != len(nums):
    if i == 3:
        'Skipping element at index 3.'
        i += 1
        continue
    print(nums[i])
    i += 1
```

    1
    2
    3
    5

