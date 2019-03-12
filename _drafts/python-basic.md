---
layout: post
title:  "python语法入门"
categories: python basic
---

- 交互模式下_保存最后一次的计算结果
```python
>>> 1 + 1
2
>>> _ * 5
10
```

- yeild 带有 yield 的函数在 Python 中被称之为 generator（生成器), 用户迭代场所
- dir() 列出对象上的可用方法
- help() 获取帮助文件, 打印"__doc__"变量, pydoc命令
- @staticmethod将方法声明为静态方法
- hash加密
```
>>>hashlib.new('md5', b'test').hexdigest()
>>>hmac.new(key, message, hashlib.sha1).digest()
```



### 参考资料
1. [https://www.python.org/](https://www.python.org/)
2. [https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/](https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/)
