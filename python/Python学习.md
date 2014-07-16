# python模块
---
## 定义模块 ##
- 定义一个模块 nester.py

```python
def print_lol(the_list):
  for each_item in the_list:
    if isinstance(each_item,list):
      print_lol(each_item)
    else:
      print(each_item)
```

## 准备发布 ##
- 为模块创建一个文件夹（nester）
- 在nester文件夹下创建一个setup.py文件
```python
from distutils.core import setup

setup(
      name          = 'nester',
      version       = '1.0.0',
      py_modules    = '['nester']',
      author        = 'lieo',
      author_email  = 'lieo@gmail.com',
      url           = 'http://www.lieo.com',
      description   = 'A simple printer of nested lists',
     )
```

## 构建发布 ##
- 构建一个发布文件
```python
进入netster目录
python setup.py sdist
```
- 将发布安装到你的python本地副本中
```python
python setup.py install
```

## 发布预览 ##
- 安装前
nester
  |
  |----nester.py(代码文件）
  |----setup.py（元数据文件）

- 安装后
nester
  |
  |----MANIFEST(这个文件包含发布中的文件列表
  |----build
  |      |
  |      |----lib
  |            |----nester.py(代码在这个文件中）
  |
  |----dist
  |      |----nester-1.0.0.tar.gz(这是发布包)
  |
  |----nester.py(代码在这个文件中)
  |----nester.pyc(这个文件中是编译版本的代码)
  |----setup.py(元数据在这)

## 导入模块并使用 ##
- 写一个小程序，导入新创建的模块，并定义一个小列表，名为“cast”，然后使用模块提供的函数在屏幕上显示这个列表的内容。

```python
import nester

cast = ['palin','cleese','idle']
print_lol(cast) (此时会发现NameError，因为你没有对名做出限定）
nester.print_lol(cast) (正确输出)
```
