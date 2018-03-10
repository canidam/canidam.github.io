---
title: "Why NumPy library is so powerful?"
tags: [python]
---

NumPy is the fundamental package for scientific computing with Python. NumPy is also a very efficient multi-dimensional container of generic data. Arbitrary data-types can be defined. This allows NumPy to seamlessly and speedily integrate with a wide variety of databases.

It behaves different than a list when executing mathematical instructions on the object.
Consider the examples below

```Python
import numpy as np

seq = [2, 4, 6, 8]
np_array = np.array([s for s in seq])

print seq * 2  # [2, 4, 6, 8, 2, 4, 6, 8]
print np_array * 2  #[ 4 8 12 16]
````

When multiplying lists, it actually *extends* the list. With numpy, the elements are iterated
and the multiplication happens on each one.

Another example is if you want to increase the value of elements by 1. With lists,
you would have to do something such

```Python
new_seq = [s + 1 for s in seq]
```

With numpy, this is much more intutive
```python
print np_array + 2  # [4 6 8 10]
```

Numpy provides universal functions that works on arrays. These are replacements for similar
functions from the math module. For example,

```Python
np.sqrt(np_array)  # ([ 1.41421356,  2.        ,  2.44948974,  2.82842712])
```

Numpy arrays are implemented in the same way C arrays are. This causes them to scale very well,
and much better than the regular list objects.

'''python
grid = np.zeros(shape=(3,3), dtype=float)
print grid

[[ 0.  0.  0.]
 [ 0.  0.  0.]
 [ 0.  0.  0.]]
 ```
