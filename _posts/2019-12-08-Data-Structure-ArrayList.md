<!-- ---
layout:     post
title:      Data Structure -- Implement Arraylist in Python
subtitle:   
date:       2019-12-08
author:     Sudo
header-img: img/post-bg-train.jpg
catalog:    true
tags:
    - Python
    - Data Structure
--- -->

# The Array-Backed List

## 1. What is array list in python?

The list data type is simply a collection of elements which have the same data type. i.e., *String*, *Int*, or even *objects*

The **list data structure** typically support these operations:
- access values in the list by their position (index)
- append and insert new values into the list
- remove values from the list


## 2. The List API
```python
class ArrayList:        
    ### access element ###

    def __getitem__(self, idx):
        """Implements `x = self[idx]`"""
        pass

    def __setitem__(self, idx, value):
        """Implements `self[idx] = x`"""
        pass

    def __delitem__(self, idx):
        """Implements `del self[idx]`"""
        pass

    ### stringification ###

    def __repr__(self):
        """Supports inspection"""
        return '[]'

    def __str__(self):
        """Implements `str(self)`"""
        return '[]'

    ### single-element manipulation ###

    def append(self, value):
        pass

    def insert(self, idx, value):
        pass

    def pop(self, idx=-1):
        pass

    def remove(self, value):
        pass

    ### predicates (T/F queries) ###

    def __eq__(self, other):
        """Implements `self == other`"""
        return True

    def __contains__(self, value):
        """Implements `val in self`"""
        return True

    ### queries ###

    def __len__(self):
        """Implements `len(self)`"""
        return len(self.data)

    def min(self):
        pass

    def max(self):
        pass

    def index(self, value, i, j):
        pass

    def count(self, value):
        pass

    ### bulk operations ###

    def __add__(self, other):
        """Implements `self + other_array_list`"""
        return self

    def clear(self):
        pass

    def copy(self):
        pass

    def extend(self, other):
        pass

    ### iteration ###

    def __iter__(self):
        """Supports iteration (via `iter(self)`)"""
        pass
```



## 3. Implement Above API of Array List

- First of all, define a normalization function inside ArrayList class to normalize "index" in case of using entering negative index. If user enter negative index, we just assume that he wants to read element from tail to head. For example, index = -1, the last element of the list will be reached.
```python
def _normalize_idx(self, idx):
    nidx = idx
    if nidx < 0:
        nidx += len(self.data)
        if nidx < 0:
            nidx = 0
    return nidx
```

- Next, we gonna to figure out how to access element.(get, delete, set )
```Python
def __getitem__(self, idx):
    """Implements `x = self[idx]`"""
    nidx = self._normalize_idx(idx)
    if nidx >= len(self.data):
        raise IndexError
    return self.data[nidx]

def __setitem__(self, idx, value):
    """Implements `self[idx] = x`"""
    nidx = self._normalize_idx(idx)
    if nidx >= len(self.data):
        raise IndexError
    self.data[nidx] = value

def __delitem__(self, idx):
    """Implements `del self[idx]`"""
    nidx = self._normalize_idx(idx)
    if nidx >= len(self.data):
        raise IndexError
    for i in range(nidx + 1, len(self.data)):
        self.data[i - 1] = self.data[i]
    del self.data[len(self.data) - 1]
```

- Stringification
```Python
def __str__(self):
    """Implements `str(self)`. Returns '[]' if the list is empty, else
    returns `str(x)` for all values `x` in this list, separated by commas
    and enclosed by square brackets. E.g., for a list containing values
    1, 2 and 3, returns '[1, 2, 3]'."""
    if len(self.data) == 0:
        return '[]'

    else:
        string = ''
        side = '['
        string += side
        for x in range(len(self.data)):
            side = str(self.data[x])
            if x < len(self.data)-1:
                string = string + side + ', '
            else:
                string = string + side
        side = ']'
    string = string + side
    return string
```

- Single-element manipulation

```Python
def append(self, value):
    """Appends value to the end of this list."""

    if self.data:
        self.data.append(None)
        self.data[len(self.data)-1] = value
    else:
        self.data.append(None)
        self.data[0] = value


def insert(self, idx, value):
    if idx > len(self.data):
        raise IndexError
    elif idx == len(self.data):
        self.data.append(None)
        self.data[len(self.data)-1] = value


    else:
        self.data.append(None)
        for i in range(len(self.data)-1,idx,-1):
            self.data[i] = self.data[i-1]
            self.data[i-1] = None
        self.data[idx] = value

def pop(self, idx=-1):
    if idx == -1:
        ele = self.data[-1]
        del self.data[-1]
        return ele
    else:
        ele = self.data[idx]
        self.__delitem__(idx)

        return ele

def remove(self, value):
    found =0
    for i in range(0,len(self.data)):
        if self.data[i] == value:
            found +=1
            self.__delitem__(i)
            break
    if found ==0:
        raise ValueError

```


- Prediction

```python
def __eq__(self, other):
    """Returns True if this ArrayList contains the same elements (in order) as
    other. If other is not an ArrayList, returns False."""
    for i in range(len(self.data)):
        if self.data[i] != other[i]:
            return False

        else:
            return True

def __contains__(self, value):
    """Implements `val in self`. Returns true if value is found in this list."""
    found = False
    for i in range(len(self.data)):
        if self.data[i] == value:
            found = True
    return found
```


- Iteration
```Python
def __iter__(self):
    """Supports iteration (via `iter(self)`)"""
    for i in range(len(self.data)):
        yield self.data[i]
```
