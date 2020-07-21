
# Tutorial Note
---

## Python
---
### iterator

``` 
for i in xrange(n)
dict_obj.iterkeys()
dict_obj.itervalues()
for k, v in dict_obj.iteritems()
for i , v in enumerate(seq_obj)
zip
open
```


### how to use iterator in costume class:


iteration protocal
```
iter()
next()
exception StopIteration:
```


### how to define a class with iterator
 define stand-alone iterator object


### closure

 No write operation
 Wrtie operation will define a new object and override the outside-scope object with the same name

### python reflection:

```
dir
__dict__: a map of (name, object) pairs
```

### Immutable and mutable object in Python

Mutable: list, set, dict, class

Immutable: int, float, str, (default function parameter)

### The difference between is and ==

**==** compares the value

a **is** b is the same as id(a) **==** id(b)

> reference **object pool** for detail


### None
is an instance of NoneType

### Bool

True/False

### import



