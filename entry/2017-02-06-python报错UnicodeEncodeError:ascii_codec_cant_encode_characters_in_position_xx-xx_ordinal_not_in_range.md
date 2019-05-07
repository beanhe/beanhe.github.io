---
layout: post
title: python报错UnicodeEncodeError:ascii codec_can't_encode_characters_in_position_xx-xx_ordinal_not_in_range(128)
category: python
tags: [python,error]
---

- **解决方案**：`PYTHONIOENCODING="UTF-8"`

> 摘自：[Stackoverflow](http://stackoverflow.com/questions/3828723/why-should-we-not-use-sys-setdefaultencodingutf-8-in-a-py-script)

- As per the documentation: This allows you to switch from the default ASCII to other encodings such as UTF-8, which the Python runtime will use whenever it has to decode a string buffer to unicode.

- This function is only available at Python start-up time, when Python scans the environment. It has to be called in a system-wide module, sitecustomize.py, After this module has been evaluated, the setdefaultencoding() function is removed from the sys module.

- The only way to actually use it is with a reload hack that brings the attribute back.

- Also, the use of sys.setdefaultencoding() has always been discouraged, and it has become a no-op in py3k. The encoding of py3k is hard-wired to "utf-8" and changing it raises an error.

- I suggest some pointers for reading:

- [http://blog.ianbicking.org/illusive-setdefaultencoding.html](http://blog.ianbicking.org/illusive-setdefaultencoding.html)
- [http://nedbatchelder.com/blog/200401/printing_unicode_from_python.html](http://nedbatchelder.com/blog/200401/printing_unicode_from_python.html)
- [http://www.diveintopython3.net/strings.html#one-ring-to-rule-them-all](http://www.diveintopython3.net/strings.html#one-ring-to-rule-them-all)
- [http://boodebr.org/main/python/all-about-python-and-unicode](http://boodebr.org/main/python/all-about-python-and-unicode)
- [http://blog.notdot.net/2010/07/Getting-unicode-right-in-Python](http://blog.notdot.net/2010/07/Getting-unicode-right-in-Python)

- [http://blog.notdot.net/2010/07/Getting-unicode-right-in-Python](http://blog.notdot.net/2010/07/Getting-unicode-right-in-Python)
