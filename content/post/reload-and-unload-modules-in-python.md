+++
title = "Reload and unload modules in Python"
date = "2014-01-13"
tags = ["python", "module"]
+++

```python
# python 2.7
import math

reload(math) # or import math again to reload the module
del(math) # unload module
```

```python
# python 3.x
import math

# the reload function was eliminated on python 3
import math # or use exec("import math")
del(math) # remove module
```
