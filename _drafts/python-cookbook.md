---
layout: post
title:  "python语法入门"
categories: progam python
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
>>>hmac.new(key, message, hashlib.sha1).hexdigest()
>>>hmac.new(bytes("test", 'utf8'), bytes("test", 'utf8'), hashlib.sha1).hexdigest()

>>>base64.b64encode(hmac.new(bytes("\n", 'utf8'), bytes("test", 'utf8'), hashlib.sha1).digest())


>>>base64.b64encode(hmac.new(bytes("EhopO08ncu9IB9o216mAARJmGTJOB2", 'utf8'), bytes("GET\n\n\nTue, 12 Mar 2019 11:47:58 GMT\n", 'utf8'), hashlib.sha1).digest())


>>>hmac.new(bytes("EhopO08ncu9IB9o216mAARJmGTJOB2", 'utf8'), bytes("test", 'utf8'), hashlib.sha1).hexdigest()

h = hmac.new(b"OtxrzxIsfpFjA7SwPzILwy8Bw21TLhquhboDYROV",
             b"PUT\nODBGOERFMDMzQTczRUY3NUE3NzA5QzdFNUYzMDQxNEM=\ntext/html\nThu, 17 Nov 2005 18:49:58 GMT\nx-oss-magic:abracadabra\nx-oss-meta-author:foo@bar.com\n/oss-example/nelson", hashlib.sha1)

base64.b64encode(h.digest())

```
- Lambda 单行匿名函数，简洁代码


### 参考资料
1. [https://www.python.org/](https://www.python.org/)
2. [https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/](https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/)
