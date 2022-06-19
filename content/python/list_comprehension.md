# **List Comprehension**

## **One-Dimensional List Comprehension**

Let's demonstrate the difference between for loops and list comprehensions in Python. A list comprehension can be used to generate lists in a single line of code, without the need for explicit loops (although it technically still uses loops "under the hood." First, we will create an empty list and create a simple list of all even integers between 0 and 20.


```python
# Create empty list
my_list = []
```


```python
# Add all even numbers from 0 to 20 to the list
for i in range(0, 20):
    if i % 2 == 0:
        my_list.append(i)

print(my_list)
```

    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]


Now, let's use a list comprehension to do the same thing. Noting the differences between the two methods will give you an idea of how to use simple list comprehensions. 


```python
# Now create the same list in one line
my_list = [i for i in range(0, 20) if i % 2 == 0]

print(my_list)
```

    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]


We eliminated three lines of code using the list comprehension.

## **Two-Dimensional List Comprehension**

Let's try a slightly more complicated list comprehension to create a matrix, or 2D array. This will essentially achieve the same result as using the double-nested for loop below.


```python
# Create empty list
my_list = []
```


```python
# Add all even numbers from 0 to 10 to the list
for i in range(0, 10):
    temp_list = []
    for j in range(0, 10):
        temp_list.append(j)
    my_list.append(temp_list)

print(my_list)
```

    [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]]


We can see that our result is a matrix, or 2-dimensional array, i.e. list of lists.
