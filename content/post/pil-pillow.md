+++
title = "PIL -> Pillow"
date = "2014-01-16"
tags = ["pillow", "pil", "python", "setuptools", "virtualenv"]
+++

[Pillow](http://pillow.readthedocs.org/en/latest/index.html) is a
[PIL](http://pythonware.com/products/pil/) fork created to add new
features. setuptools support was also added. A more frequent release
cycle was also promised. With Pillow you can have PIL as a package
dependency in setuptools and virtualenv. That means less clutter and
robustness for us.

Pillow allows you to continue to use ` import PIL `, so there is no need
to change your current PIL related code. 0 migration overhead.

Archlinux already dropped support for PIL in favor of Pillow.

**TL;DR PIL \> Pillow**
