---
layout: post
title: "How to Obtain the Last Element from a Python Iterator"
description: ""
category: quick tips
tags: [python]
---
{% include JB/setup %}

## Use for loop

### Python 3.x

```python
*_, last = iterator
```

### Python 2.x

```python
last = next(iterator)
for last in iterator:
    pass
```


## Use a deque of size 1

```python
from collections import deque

iter_deque = deque(iterator, maxlen)
last = iter_deque.pop()
```


## pytest examples

```python
def test_get_last_element():
    itr = iter([1, 2, 3, 4, 5])
    itr_deque = deque(itr, maxlen=1)
    assert itr_deque.pop() == 5

    itr = iter([100, 200, 300, 400, 500])
    *_, last = itr
    assert last == 500

    itr = iter(['a', 'b', 'd', 'd', 'e'])
    last = next(itr)
    for last in itr:
        pass
    assert last == 'e'
```


## Reference

* [Cleanest way to get last item from Python iterator](https://stackoverflow.com/q/2138873)
