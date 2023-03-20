---
layout: single
title:  "接口兼容的一个小技巧"
date:   2023-03-20 10:47:00 +0800
author: lanhao

---

面向可复用编程时，需要考虑接口的兼容。


#### 原始函数

```python

def test_a(a):
    pass

```

#### 当需要调整函数名，或者作较大更改时

```python

def test_b(a):
    pass

def test_a(a):
    return test_b(a)

```

#### 引入warning 来提示

```python

import warnings

def test_b(a):
    pass

def test_a(a):
    warnings.warn("test_a will be delete,you may change to use test_b",stacklevel=2)
    return test_b(a)

```

#### 配合版本号作提示

```python
import warnings

__version__ = "0.0.9"

def test_b(a):
    pass

def test_a(a):
    if __version__ <= "0.0.5":
        warnings.warn("test_a will be delete,you may change to use test_b",stacklevel=2)
        return test_b(a)
    else:
        raise Exception("test_a can not be use")
```

#### 多次版本更新后，去除兼容

```python

def test_b(a):
    pass

```

