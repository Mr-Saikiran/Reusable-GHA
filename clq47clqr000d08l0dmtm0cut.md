---
title: "Beginner's Python: Unveiling Quirky and Useful Functions You Didn't know Existed"
datePublished: Mon Oct 14 2019 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clq47clqr000d08l0dmtm0cut
slug: beginners-python-unveiling-quirky-and-useful-functions-you-didnt-know-existed
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702497748432/c566cf16-0ace-4c05-aca9-4d41127f3921.jpeg
tags: python, python3, py, python-tips, python-beginner, python-tricks

---

### 1\. Function to make a noun (adjective) from a verb or an adverb from a noun (adjective)

```python
def verbing(s):
    returns iflen(s) < 3 elses + ('ing', 'ly')['ing' ins]
>>> verbing('help')
helping
>>> verbing('helping')
helpingly
```

### 2\. use a bare “\*” asterisk in function parameter lists to force the caller to use keyword arguments for certain parameters

```python
>>> def f(a, b, *, c='x', d='y', e='z'):
...     return 'Hello'
```

To pass the value for c, d, and e you will need to explicitly pass it as “key=value” named arguments:

```python
>>> f(1, 2, 'p', 'q', 'v')
TypeError: f() takes 2 positional arguments but 5 were given
>>> f(1, 2, c='p', d='q',e='v')
'Hello'
```

### 3\. getpass module provides a platform-independent way to enter a password in a command-line program without echoing

```python
>>> import getpass
>>> password = getpass.getpass()
Password:
>>> password
'alohomora'
```

`getuser()` gets the current username:

```python
>>> getpass.getuser()
'sirius_black'
```

### 4\. if the fractional component of the number is halfway between two integers, one of which is even and the other odd, then `round()` returns the even. This kind of rounding is called rounding to even (or banker’s rounding)

```python
>>> round(0.5)
0
>>> round(2.4)
2
>>> round(2.5)
2
>>> round(2.6)
3
>>> round(3.5)
4
>>> round(4.5)
4
```

### 5\. The built-in function `sys.getsizeof()` returns the size of an object in bytes. The object can be any type of object

```python
>>> sys.getsizeof(1)
28
>>> sys.getsizeof([1, 2, 3])
88
>>> sys.getsizeof('')
49
>>> sys.getsizeof('P')
50
>>> sys.getsizeof('Py')
51
>>> sys.getsizeof('Python')
55
```