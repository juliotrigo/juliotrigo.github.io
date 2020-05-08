---
layout: post
title: "Unicode strings and bytestrings in Python 2"
author: Julio Trigo
date: 2015-04-21 20:24:00 +0100
last_modified_at: 2020-05-08 18:00:00 +0000
permalink: /articles/unicode-strings-and-bytestrings-in-python-2/
redirect_from:
  - /2015/04/unicode-strings-and-bytestrings-in-python-2.html
  - /posts/unicode-strings-and-bytestrings-in-python-2/
tags:
  - python
  - unicode
  - UTF-8
---

```shell
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can’t encode character u'\u03b1'
in position 0: ordinal not in range(128)
```

Does this Python exception look familiar? Most Python developers have seen this at least once.

Character sets, encodings, Unicode and all that stuff are hard to deal with until you fully understand what is going on behind the scenes.<!--more--> Please find below a few web pages that I find essential:

* [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](https://www.joelonsoftware.com/articles/Unicode.html) by Joel Spolsky

* [Unicode HOWTO — Python 2.7.9 documentation](https://docs.python.org/2/howto/unicode.html)

* [7.8. codecs — Codec registry and base classes — Python 2.7.9 documentation](https://docs.python.org/2/library/codecs.html#encodings-and-unicode)

What I am going to do is to provide some Unicode and bytestring examples using Python 2 to help understand them better.

Let's say that we want to work with some Greek characters, which cannot be represented in ASCII (0-127): α β γ ε ...

We can start creating them as Unicode characters.

```shell
>>> a = u'\u03b1'
>>> b = u'\u03b2'
>>> g = u'\u03b3'

>>> a, b, g
(u'\u03b1', u'\u03b2', u'\u03b3')

>>> print a, b, g
α β γ
```

In **Unicode**, characters are represented by code points: integer values usually denoted in base 16.

We cannot represent them using the **ASCII** character-encoding scheme (7 bits, 0-127), which is the default encoding in Python 2. So it is the one used (by default) when converting Unicode characters into bytestrings (str in Python 2).

```shell
>>> alphabet = '{}{}{}{}'.format(a, b, g, 'd')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can’t encode character u'\u03b1'
in position 0: ordinal not in range(128)
```

```shell
>>> alphabet = u'{}{}{}{}'.format(a, b, g, 'd')

>>> alphabet
u'\u03b1\u03b2\u03b3d'

>>> print alphabet
αβγd
```

A Unicode string is a sequence of code points (integers). In order to represent Unicode strings as a sequence of bytes we need to encode them using a character encoding. **UTF-8** is a character encoding capable of encoding all possible code points in Unicode.

```shell
>>> alphabet_str = alphabet.encode('utf-8')

>>> alphabet_str
'\xce\xb1\xce\xb2\xce\xb3d'

>>> print alphabet_str
αβγd
```

Now, we have a Python str (UTF-8 encoding): a sequence of bytes representing the Unicode characters.

If a character encoding is not specified, then the **encode** method will assume that we want to use ASCII and will fail because those characters cannot be represented in ASCII.

```shell
>>> alphabet_str = alphabet.encode()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can’t encode characters
in position 0-2: ordinal not in range(128)
```

We cannot represent those characters in **latin-1** (**ISO-8859-1**) either.

```shell
>>> alphabet_str = alphabet.encode('latin-1')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'latin-1' codec can’t encode characters
in position 0-2: ordinal not in range(256)
```

And, as we expect, we also get an error if we try to decode the Unicode string (which does not make sense).

```shell
>>> alphabet_str = alphabet.decode('utf-8')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/System/Library/Frameworks/Python.framework/Versions/
2.7/lib/python2.7/encodings/utf_8.py", line 16, in decode
    return codecs.utf_8_decode(input, errors, True)
UnicodeEncodeError: 'ascii' codec can’t encode characters
in position 0-2: ordinal not in range(128)
```

Once we have the bytestring, we can **decode** it using the UTF-8 character encoding, and get back the original Unicode string.

```shell
>>> alphabet_u = alphabet_str.decode('utf-8')

>>> alphabet_u
u'\u03b1\u03b2\u03b3d'

>>> print(alphabet_u)
αβγd
```

Again, we cannot encode a bytestring that was already encoded using UTF-8 (it does not make sense either).

```shell
>>> alphabet_u = alphabet_str.encode('utf-8')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can’t decode byte 0xce
in position 0: ordinal not in range(128)
```

We can see that all those variables have different types:

```shell
>>> type(alphabet)
<type 'unicode'>

>>> type(alphabet_str)
<type 'str'>

>>> type(alphabet_u)
<type 'unicode'>
```

And different lengths.

```shell
>>> len(alphabet_str)
7

>>> len(alphabet_u)
4
```

So, in order to convert types:

```python
s.decode(encoding)  # <type 'str'> to <type 'unicode'>
u.encode(encoding)  # <type 'unicode'> to <type 'str'>
```

Additionally, we need to keep in mind that we can write Unicode literals in any encoding using Python. In order to do that, we have to declare the encoding being used by including a special comment as either the first or second line of the source file.

```python
# -*- coding: latin-1 -*-

# -*- coding: utf-8 -*-
```

Unicode literals are written as strings prefixed with the `'u'` or `'U'` character.

Below, we can see some examples of how the same string could be a Unicode or a bytestring string just depending on whether we write it as a Unicode literal or not.

```shell
>>> u'ε'
u'\u03b5'

>>> 'ε'
'\xce\xb5'

>>> alphabet2 = u'{}{}{}{}'.format(a, b, g, u'ε')

>>> alphabet2
u'\u03b1\u03b2\u03b3\u03b5'

>>> alphabet2 = u'{}{}{}{}'.format(a, b, g, 'ε'.decode('utf-8'))

>>> alphabet2
u'\u03b1\u03b2\u03b3\u03b5'

>>> print alphabet2
αβγε

>>> alphabet2 = u'{}{}{}{}'.format(a, b, g, 'ε')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can’t decode byte 0xce
in position 0: ordinal not in range(128)
```

Finally, we can see how we may get different bytestrings when encoding the same Unicode string using different character encodings.

```shell
>>> u'nona'.encode('utf-8')
'nona'

>>> u'nona'.encode('latin-1')
'nona'

>>> u'ñóñà'.encode('utf-8')
'\xc3\xb1\xc3\xb3\xc3\xb1\xc3\xa0'

>>> u'ñóñà'.encode('latin-1')
'\xf1\xf3\xf1\xe0'
```
