
Let's demonstrate the difference between for loops and list comprehensions in Python. A list comprehension can be used to generate lists in a single line of code, without the need for loops (although it technically does use loops "under the hood." First, we will create an empty list and add all even numbers from 0 to 20 using a for loop.


```python
# Create empty list
my_list = []

# Add all even numbers from 0 to 100 to the list
for i in range(0, 20):
    if i % 2 == 0:
        my_list.append(i)

print(my_list)
```

    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]


Now, let's do it using a list comprehension. Notice how everything aligns between the loop syntax above and the more compact list comprehension below.


```python
# Now create the same list in one line
my_list = [i for i in range(0, 20) if i % 2 == 0]

print(my_list)
```

    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

