# Implement a basic Python linked list

## Create `Node` class

### Define `init` method


```python
class Node(object):
    def __init__(self, data, next=None):
        self.data = data
        self.next = next
```

## Create `LinkedList` class

### Define `__init__` method


```python
class LinkedList(object):
    def __init__(self, head=None, tail=None):
        self.head = None
        self.tail = None
```

### Define `print_list` method


```python
class LinkedList(object):
    
    # ...
    
    def print_list(self):
        print('List Values: ')
        # Start at the head
        current_node = self.head
        # Iterate until current node is None
        while current_node != None:
            # Print node data
            print(current_node.data)
            # Move to the next node
            current_node = current_node.next
        print(None)    
    
```

### Define `append` method


```python
class LinkedList(object):
    
    # ...
    
    def append(self, data):
        node = Node(data, None)
        # Handle empty case
        if self.head is None:
            self.head = node
            self.tail = node
        else:
            # Otherwise set a new next node for the tail
            self.tail.next = node
        # Set a new tail
        self.tail = node
```

### Define `remove` method


```python
class LinkedList(object):
    
    # ...
    
    def remove(self, node_value):
        # Keep track of current and previous node
        current_node = self.head
        previous_node = None
        # Iterate through list to find value to be removed
        while current_node != None:
            if current_node.data == node_value:
                if previous_node is not None:
                    previous_node.next = current_node.next
                else:
                    # Handle edge case
                    self.head = current_node.next
            
            previous_node = current_node
            current_node = current_node.next

```

### Define `insert` method


```python
class LinkedList(object):
    
    # ...
    
    def insert(self, value, at):
        current_node = self.head
        new_node = Node(value)
        # Iterate to find value after which to insert new node
        while current_node != None:
            if current_node.data == at:
                if current_node is not None:
                    new_node.next = current_node.next
                    current_node.next = new_node
                else:
                    # Handle edge case
                    self.tail = current_node.next
            
            # Move to the next node
            current_node = current_node.next

```

## The full linked list implementation


```python
class Node(object):
    def __init__(self, data, next=None):
        self.data = data
        self.next = next

class LinkedList(object):
    def __init__(self, head=None, tail=None):
        self.head = None
        self.tail = None
    
    def print_list(self):
        print('List Values: ')
        # Start at the head
        current_node = self.head
        # Iterate until current node is None
        while current_node != None:
            # Print node data
            print(current_node.data)
            # Move to the next node
            current_node = current_node.next
    
    def append(self, data):
        node = Node(data, None)
        # Handle empty case
        if self.head is None:
            self.head = node
            self.tail = node
        else:
            # Otherwise set a new next node for the tail
            self.tail.next = node
        # Set a new tail
        self.tail = node
    
    def remove(self, node_value):
        # Keep track of current and previous node
        current_node = self.head
        previous_node = None
        # Iterate through list to find value to be removed
        while current_node != None:
            if current_node.data == node_value:
                if previous_node is not None:
                    previous_node.next = current_node.next
                else:
                    # Handle edge case
                    self.head = current_node.next
            
            previous_node = current_node
            current_node = current_node.next
    
    def insert(self, value, at):
        current_node = self.head
        new_node = Node(value)
        # Iterate to find value after which to insert new node
        while current_node != None:
            if current_node.data == at:
                if current_node is not None:
                    new_node.next = current_node.next
                    current_node.next = new_node
                else:
                    # Handle edge case
                    self.tail = current_node.next
            
            # Move to the next node
            current_node = current_node.next
            
```

## Testing the linked list

### Initialize a new linked list and append some values


```python
my_list = LinkedList()
my_list.append(1)
my_list.append(2)
my_list.append(3)
my_list.append(4)
my_list.append(5)

my_list.print_list()
```

    List Values: 
    1
    2
    3
    4
    5


### Remove some values and insert a new value


```python
my_list.remove(2)
my_list.remove(4)
my_list.insert(7, at=5)

my_list.print_list()
```

    List Values: 
    1
    3
    5
    7

