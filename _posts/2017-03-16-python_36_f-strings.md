---
layout: post
title:  "f-strings in Python 3.6"
date:   2018-02-16
categories: [python]
---

While python 3.6 has been out for over a year I recently stumbled across one feature that I wasn't aware of: f-strings!

### What are they?

f-strings (short for format-strings I assume) are a way of simplifying your string variable representation. Just prefix your string with `f` and then directly call your arguements inside curly braces.  

```python
speed='quick'
animal='fox'
number=10000
print(f'The {speed} brown {animal} jumped {number} meters over the moon')
>> 'The quick brown fox jumped 10000 meters over the moon'
```

This can be contrasted with the *old* way of doing it (Python 3.x.y where x < 6 and Python 2.7.y) using `.format()`

```python
speed='quick'
animal='fox'
number=10000
print('The {speed} brown {animal} jumped {number} meters over the moon'.format(speed=speed, animal=animal, number=number))
>> 'The quick brown fox jumped 10000 meters over the moon'
```

or the *really old* way of doing it (using older version of Python 2.x.y where x < 7) 

```python
speed='quick'
animal='fox'
number=10000
print('The %s brown %s jumped %d meters over the moon' % (speed, animal, number))
>> 'The quick brown fox jumped 10000 meters over the moon'
```

Looks like a great feature to make your strings parsing a bit more compact and from now on I will favour this over the `.format()` approach. 

You might ask whether introducing *another* way to format strings breaks the [Zen of Python](https://www.python.org/dev/peps/pep-0020/) (There should be one-- and preferably only one --obvious way to do it.) and you can find quite a few rants online from people unhappy about this but I think as form of syntactic sugar is a nice addition.
 