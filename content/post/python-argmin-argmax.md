+++
title = "Python cookbook: argmin & argmax"
date = "2015-12-01"
tags = ["python", "argmin", "argmax"]
+++
```python
def argmin(iter, function):
    return min((f(x), x) for x in iter)[1]

def argmax(iter, function):
    return max((f(x), x) for x in iter)[1]
```


```python
>>> argmin(range(-100, 100), lambda x: x * x)
0
>>> argmax(range(-100, 100), lambda x: - x * x)
0
>>> argmax([[1, 2, 4], [1], [2, 5], []], len)
[1, 2, 4]
```