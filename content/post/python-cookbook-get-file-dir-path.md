+++
title = "Python cookbook: get the file dir path"
date = "2014-01-09"
tags = ["dir", "os", "python"]
+++

```python
import os
os.path.dirname(os.path.abspath(__file__))
```
