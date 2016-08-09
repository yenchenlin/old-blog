---
layout: post
title:  "Workaround to use fused types class attributes"
date:   2016-08-08 18:52:21 -0500
categories: Deep Learning
comments: true
excerpt: "This post will introduce a workaround to push Cython' limitation."
---

In some of my previous blog posts , we've seen Cython fused types' ability to dramatically reduce both [memory usage](http://blog.yclin.me/gsoc/2016/06/27/KMeans/) and [code duplication]([Using Function Pointer to Maximize Code Reusability in Cython](http://blog.yclin.me/gsoc/2016/07/17/Function-Pointer/)).

However, of coure there are still some deficiencies of Cython fused types. In this blog post, we are gonna see one of the biggest inconvenience of fused types and how to address it using a hacky workaround.


## Fused Types Limitation

When you enter the [official page](http://cython.readthedocs.io/en/latest/src/userguide/fusedtypes.html) of Cython fused types, you can easily find the following warning:

> Note Fused types are not currently supported as attributes of extension types. Only variables and function/method arguments can be declared with fused types.

It means that if you have a class written in Cython such as the following:

```python
cdef class Printer:
	cdef float num;
	
	def __init__(self):
		self.num = 0
		print self.num		
```

You are not allowed to use fused types to make function `printNum` become more generalizable. For example, if we change the type of attribute `num` from `int` to `numeric` of above code snippet:

**Note**: `cython.floating` can either refer to `float` or `double`.

```python
from cython cimport floating
cdef class Printer:
	cdef floating num;

	def __init__(self):
		self.num = 0
		print x		
```

It will result in error below since fused types can't be use as extension type attributes:

```
Fused types not allowed here.
```

## Intuitive Solution

Base on my previous experiences with Cython fused types and suggestion from my mentor [Joel](https://github.com/jnothman), it is intuitive to declare the attribute we want it to be fused types as `void*`, and then further typecast it in every functions it will be accessed. 

To be more concrete, let's look at the code:

```python
from cython cimport floating
cdef class Printer:
	
	// We wish num to be fused types, so declared it as void*
	cdef void *num;

	def __init__(self):
		cdef float num = float(5)
		self.num = &num
		
		// Typecast it when we want to access its value
		cdef floating *num = <floating*>self.num
		print num[0]
```

However, above code will again result in error due to a 
unwritten rules of fused types: 

**Fused types can only be used in a function when any of its arguments is declared to be that dused types**.

Why this rules exists is because Cython fused types works by generating multiple C functions, each function's name involve the actual type it refers to. Nonetheless, once the fused types is not involved in the function's signature, it will cause error since each generated functions will have the same name.


## Workaround

Base on the above unwritten rules, here's the final workaround we can adopt:

```python
from cython cimport floating
cdef class Printer:
	
	// We wish num to be fused types, so declared it as void*
	cdef void *num;
	
	cdef bint is_float;
	
	// Used as fused type arguments
	cdef float float_sample;
	cdef long long_sample;

	def __init__(self):
		cdef float num = float(5)
		self.num = &num
		
		if type(num) == float:
			self.is_float = True
		else:
			self.is_float = False
			
		if self.is_float:
			self._print(float_sample)
		else:
			self._print(double_sample)
	
	// Underlying function
	def _print(self, floating sample):
		// Typecast it when we want to access its value
		cdef floating *num = <floating*>self.num
		print num[0]
```

As you can see, we have to also modified the functions that we want to access, keeping the original function signature as a wrapper and then introduce fused types function arguments into its underlying implementation function.

That's it, although it looks really hacky but it works!
Hope that Cython can add this functionality soon.

## Summary

Please leave any thought you have after reading, let's push Cython's limitation together!
