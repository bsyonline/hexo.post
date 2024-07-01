---
title: python
tags: []
category:
  - python
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



### 基本语法

#### 运算符
| 运算符                                                   | 说明            |
| ----------------------------------------------------- | ------------- |
| [] [:]                                                | 切片            |
| **                                                    | 指数            |
| ~                                                     | 按位取反          |
| ^                                                     | 按位异或          |
| \|                                                    | 按位或           |
| `+`， `-`， `*`， `/`， `%`， `//`                         | 加、减、乘、除、取模、整除 |
| >>                                                    | 右移            |
| <<                                                    | 左移            |
| `is`， `is not`                                        | 身份运算          |
| `in`， `not in`                                        | 成员运算          |
| `not`， `or`， `and`                                    | 逻辑运算          |
| `<=` ，`<` ，`>`， `>=`，`==`， `!=`                       | 比较运算          |
| `=`， `+=`， `-=`， `*=`， `/=`， `%=`， `//=`， `**=`， `&=` | 赋值运算          |



#### 模块
```python
def foo():  
    print('hello, world!')  
  
def foo():  
    print('hello, python!')
```
python 中一个文件就代表一个模块（module），如果同一个 .py 文件中有多个同名函数，只会有一个生效（最后一个）。不同的 module 中可以定义相同名称的函数。
`foo.py`
```python
def foo():  
    print('hello, world!')  
```
`bar.py`
```python
def foo():  
    print('hello, python!')  
```
`test.py`
```python
from foo import foo  
foo()  
from bar import foo  
foo()

import foo as f1  
import bar as f2  
f1.foo()  
f2.foo()
```


#### 字符串
##### 字符串的表示
用单引号、双引号都可以。
```python
s1 = 'hello'
s2 = "world"
s3 = """
hello,
world!
"""
```

`\` 表示转义，`r` 表示不转义。
```python
s4 = "hello\nworld"
s5 = r"hello\nworld"
```

##### 字符串的格式化
```python
name = 'Tom'  
print('hello, %s' % name)  
print('hello, {}'.format(name))  
print(f'hello, {name}')
```

##### 字符串操作
`+` 拼接
```python
s6 = "hello" + "world"
```

`*` 重复
```python
s7 = "hello"*3
print(s7) #hellohellohello
```

截取
```python
s8 = 'hello world'  
print(s8[0])  #h
print(s8[0:5])  #hello
print(s8[6:])  #world
print(s8[-1])  #d
print(s8[-5:-1]) #worl
```

字符串的方法
```python
s9 = 'hello world hello python'  
# 指定字符串出现的次数  
print(s9.count('hello'))    # 2  
# 将字符串以指定的宽度居中并在两侧填充指定的字符  
print(s9.center(50, '*'))   # ****************hello world hello python****************  
# 将字符串以指定的宽度靠右放置左侧填充指定的字符  
print(s9.rjust(50, ' '))    #                                  hello world hello python  
# 通过内置函数len计算字符串的长度  
print(len(s9))  # 24  
# 获得字符串修剪左右两侧空格之后的拷贝  
print(s9.strip())   # hello world hello python  
# 获得字符串首字母大写的拷贝  
print(s9.capitalize())  # Hello world hello python  
# 获得字符串变大写后的拷贝  
print(s9.upper())   # HELLO WORLD HELLO PYTHON  
# 获得字符串变小写后的拷贝  
print(s9.lower())   # hello world hello python  
# 获得字符串每个单词首字母大写的拷贝  
print(s9.title())   # Hello World Hello Python  
# 检查字符串是否以指定的字符串开头  
print(s9.startswith('hello'))   # True  
# 检查字符串是否以指定的字符串结尾  
print(s9.endswith('world')) # False  
# 从字符串中查找子串所在位置  
print(s9.find('world')) # 6  
print(s9.find('python'))    # -1  
# 与find类似但找不到子串时会引发异常  
print(s9.index('world'))    # 6  
print(s9.replace('world', 'python'))    # hello python hello python  
print(s9.split(' '))  
print(s9.split(' ', 1)) # ['hello', 'world hello python']  
print(s9.split(' ', 2)) # ['hello', 'world', 'hello python']  
print(s9.split(' ', 3)) # ['hello', 'world', 'hello', 'python']  
# 检查字符串是否由数字构成  
print(s9.isdigit())  # False  
# 检查字符串是否以字母构成  
print(s9.isalpha())  # False  
# 检查字符串是否以数字和字母构成  
print(s9.isalnum())  # False  
# 检查字符串是否以空格构成  
print(s9.isspace())  # False  
# 检查字符串是否以标题显示(每个单词首字母大写)  
print(s9.istitle())  # False  
# 检查字符串是否以大写显示  
print(s9.isupper())  # False  
# 检查字符串是否以小写显示  
print(s9.islower())  # True  
# 检查字符串是否以首字母大写显示  
print(s9.istitle())  # False
```

#### 列表
```python
# 定义  
fruit = ['apple', 'orange', 'banana']  
print(fruit) # ['apple', 'orange', 'banana']  
print(type(fruit)) # <class 'list'>  
# 获取长度  
print(len(fruit)) # 3  
# 获取元素  
print(fruit[1]) # orange  
print(fruit[-1]) # banana  
# 修改元素  
fruit[1] = 'black barry'  
print(fruit) # ['apple', 'black barry', 'banana']  
# 增加元素  
fruit.append('greap')  
print(fruit) # ['apple', 'black barry', 'banana', 'greap']  
# 元素类型可以不同  
fruit.append(100)  
print(fruit) # ['apple', 'black barry', 'banana', 'greap', 100]  
# 从末尾删除元素  
fruit.pop()  
print(fruit) # ['apple', 'black barry', 'banana', 'greap']  
# 删除索引位置的元素  
a = fruit.pop(2)  
print(a) # banana  
print(fruit) # ['apple', 'black barry', 'greap']  
# 插入元素  
fruit.insert(1, 'orange')  
print(fruit) # ['apple', 'orange', 'black barry', 'greap']  
# 根据索引删除  
del fruit[0]  
print(fruit) # ['orange', 'black barry', 'greap']  
# 根据索引删除  
fruit.pop(1)  
print(fruit) # ['orange', 'greap']  
# 根据值删除  
fruit.remove('greap')  
print(fruit) # ['orange', 'black barry']  
# 清空  
fruit.clear()  
print(fruit) # []  
fruit = ['apple', 'orange', 'banana']  
# 遍历  
for f in fruit:  
    print(f) # orange black barry  
# 列表生成式  
for r in range(1, 10):  
    print(r ** 2) # 1 4 9 16 25 36 49 64 81  
vagetable = ['potato', 'tomato']  
fruit.extend(vagetable)  
print("---")  
print(fruit) # ['apple', 'orange', 'banana', 'potato', 'tomato']  
# 合并列表  
food = fruit + vagetable  
print(food) # ['apple', 'orange', 'banana', 'potato', 'tomato']  
# 复制列表  
food_copy = food[:]  
print(food_copy) # ['apple', 'orange', 'banana', 'potato', 'tomato']  
# 列表排序  
food.sort()  
print(food) # ['apple', 'banana', 'orange', 'potato', 'tomato']  
# 列表倒序  
food.sort(reverse=True)  
print(food) # ['tomato', 'potato', 'orange', 'banana', 'apple']  
# 列表反转  
food.reverse()  
print(food) # ['apple', 'banana', 'orange', 'potato', 'tomato']  
# 列表切片  
print(food[1:3]) # ['banana', 'orange']  
print(food[:3]) # ['apple', 'banana', 'orange']  
print(food[1:]) # ['banana', 'orange', 'potato', 'tomato']  
print(food[-2:]) # ['potato', 'tomato']  
# 列表反向切片复制  
food = food[::-1]  
print(food) # ['tomato', 'potato', 'orange', 'banana', 'apple']
```


#### 元组
元组是不可变的。
```python
# 定义  
tuple = ('a', True, 1, 1.1, [1, 2, 3])  
print(tuple) # ('a', True, 1, 1.1, [1, 2, 3])  
print(type(tuple)) # <class 'tuple'>
# 获取元素  
print(tuple[0]) # a  
  
# 元组的元素不可修改  
# tuple[0] = 'b' # TypeError: 'tuple' object does not support item assignment  
# 元组的元素可以是列表，列表的元素是可以修改的  
tuple[4][0] = 2  
print(tuple) # ('a', True, 1, 1.1, [2, 2, 3])  
  
# 遍历  
for t in tuple:  
    print(t) # a True 1 1.1 [2, 2, 3]  
  
# 元组的方法  
# 获取元素的索引  
print(tuple.index(1)) # 2  
# 获取元素的个数  
print(tuple.count(1)) # 1  
# 获取元组的长度  
print(len(tuple)) # 5  
# 元组的合并  
tuple1 = (1, 2, 3)  
tuple2 = (4, 5, 6)  
tuple3 = tuple1 + tuple2  
print(tuple3) # (1, 2, 3, 4, 5, 6)  
# 元组的复制  
tuple4 = tuple3[:]  
print(tuple4) # (1, 2, 3, 4, 5, 6)  
# 元组的排序  
tuple5 = (3, 2, 1)  
tuple6 = sorted(tuple5)  
print(tuple6) # [1, 2, 3]  
# 元组的反转  
tuple7 = (1, 2, 3)  
tuple8 = tuple7[::-1]  
print(tuple8) # (3, 2, 1)  
# 元组的清空  
tuple9 = (1, 2, 3)  
tuple9 = ()  
print(tuple9) # ()  
# 元组的删除  
tuple10 = (1, 2, 3)  
del tuple10  
# print(tuple10) # NameError: name 'tuple10' is not defined  
# 元组的切片  
tuple11 = (1, 2, 3, 4, 5)  
print(tuple11[1:3]) # (2, 3)  
print(tuple11[1:]) # (2, 3, 4, 5)  
print(tuple11[:3]) # (1, 2, 3)  
print(tuple11[-1]) # 5  
print(tuple11[-3:-1]) # (3, 4)  
# 元组的拷贝  
tuple12 = (1, 2, 3)  
tuple13 = tuple12  
print(tuple13) # (1, 2, 3)  
# 元组的比较  
tuple14 = (1, 2, 3)  
tuple15 = (1, 2, 3)  
print(tuple14 == tuple15) # True  
print(tuple14 != tuple15) # False  
print(tuple14 > tuple15) # False  
  
# 元组的解包  
a, b, c = (1, 2, 3)  
print(a) # 1  
  
# 将元组转换成列表  
list = list(tuple)  
print(list) # ['a', True, 1, 1.1, [2, 2, 3]]  
  
# 将列表转换成元组  
tuple = tuple(list)  
print(tuple) # ('a', True, 1, 1.1, [2, 2, 3])
```


#### 集合
```python
# 创建集合  
set = {1, 2, 3, 3, 3, 4, 5}  
print(set) # {1, 2, 3, 4, 5}  
print(type(set)) # <class 'set'>
# 集合的长度  
print(len(set)) # 5  
# 遍历集合  
for s in set:  
    print(s) # 1 2 3 4 5  
  
# 集合的方法  
# 添加单个元素
set.add(6)  
print(set) # {1, 2, 3, 4, 5, 6}  
# 添加多个元素
set.update([7, 8])  
print(set) # {1, 2, 3, 4, 5, 6, 7, 8}
# 删除元素，如果元素不存在于集合中，它会引发KeyError异常  
set.remove(6)  
print(set) # {1, 2, 3, 4, 5}  
# 删除元素，如果元素不存在于集合中，它什么都不会做  
set.discard(5)  
print(set) # {1, 2, 3, 4}  
# 删除元素，随机删除一个元素  
set.pop()  
print(set) # {2, 3, 4}  
# 清空集合  
set.clear()  
print(set) # set()  
# 删除集合  
del set  
# print(set) # NameError: name 'set' is not defined  
# 集合的运算  
set1 = {1, 2, 3}  
set2 = {3, 4, 5}  
# 集合的并集，去重  
set3 = set1 | set2  
print(set3) # {1, 2, 3, 4, 5}  
# 集合的交集，取相同的元素  
set4 = set1 & set2  
print(set4) # {3}  
# 集合的差集，取不同的元素  
set5 = set1 - set2  
set6 = set2 - set1  
print(set5) # {1, 2}  
print(set6) # {4, 5}  
# 集合的对称差集  
set7 = set1 ^ set2  
print(set6) # {1, 2, 4, 5}  
# 集合的子集判断  
print(set1.issubset(set3)) # True  
print(set1 <= set3) # True  
# 集合的超集判断  
print(set3.issuperset(set1)) # True  
print(set3 >= set1) # True
# 集合的相等  
print(set1 == set2) # False  
# 集合的不相等  
print(set1 != set2) # True  
# 集合的拷贝  
set8 = set1.copy()  
print(set8) # {1, 2, 3}  
# 集合的转换  
list = [1, 2, 3]  
set9 = set(list)  
print(set9) # {1, 2, 3}  
# 集合的排序  
set10 = sorted(set9)  
print(set10) # [1, 2, 3]
```

#### 字典
是由一个键和一个值组成的“键值对”，键和值通过冒号分开。
```python
# 字典定义  
d = {"name": "zhangsan", "age": 20}  
d1 = dict(name="zhangsan", age=20)  
d2 = dict([("name", "zhangsan"), ("age", 20)])  
d3 = dict({"name": "zhangsan", "age": 20})  
d4 = dict(zip(["name", "age"], ["zhangsan", 20]))  
d5 = {k: v for k, v in [("name", "zhangsan"), ("age", 20)]}  
d6 = {k: v for k, v in {"name": "zhangsan", "age": 20}.items()}  
d7 = {("name", "age")[i]: ("zhangsan", 20)[i] for i in range(2)}  
d8 = dict.fromkeys(["name", "age"], ["zhangsan", 20])  
print(d)    # {'name': 'zhangsan', 'age': 20}  
print(type(d))  # <class 'dict'>  
# 通过键可以获取字典中对应的值  
print(d["name"])    # zhangsan  
print(d.get("age")) # 20  
print(d.get("age"), 0) # 20  
# 通过键可以修改字典中对应的值  
d["gender"] = 'male'  
print(d)   # {'name': 'zhangsan', 'age': 20  
d = {}  
d['name'] = "lisi"  
d['age'] = 20  
print(d)    # {'name': 'lisi', 'age': 20}  
d['age'] = 21  
print(d)    # {'name': 'lisi', 'age': 21}  
d.update(gender='female', age=22)  
print(d)    # {'name': 'lisi', 'age': 22, 'gender': 'female'}  
# 删除字典中的元素  
d.popitem()  
print(d)    # {'name': 'lisi', 'age': 22}  
d.pop('age')  
print(d)    # {'name': 'lisi'}  
del d['name']  
print(d)    # {}  
d.clear()  
print(d)    # {}  
#对字典进行遍历  
for e in d1:  
    print(e.title() + ":" + str(d1[e]))  
  
#遍历字典的键  
for k in d1.keys():  
    print(k)  
  
#遍历字典的值  
for v in d1.values():  
    print(v)  
  
for k,v in d1.items():  
    print(k.title() + ":" + str(v))
```

### 面向对象
#### 类的定义
`class`关键字定义类
```python
class Student:  
    # __init__是一个特殊方法用于在创建对象时进行初始化操作  
    # 默认方法，python会自动执行
    def __init__(self, name, age):  
        self.name = name  
        self.age = age  
  
  
def main():  
    s = Student("zhangsan", 18)  
    print(s.name)  
    print(s.age)  
  
  
if __name__ == '__main__':  
    main()
```

#### 可见性
python 没有严格的可见性，比如上面的 `name` 是公开的，如果希望属性是私有的，可以用 `_name` 和 `__name` 。`_name` 是一种命名惯例，表示这个属性是私有的，但是外部仍然是可以访问的。`__name` 是私有的，无法从外部访问，但是了解 python 的名称的更换规则的话，还是可以访问的。
```python
class Student:  
    def __init__(self, name, age, gender):  
        self.name = name  
        self._age = age  
        self.__gender = gender  
  
  
def main():  
    s = Student("zhangsan", 18, 'male')  
    print(s.name)  
    print(s._age)  
    # print(s.__gender) # AttributeError: 'student' object has no attribute '__gender'  
    print(s._Student__gender)  
  
  
if __name__ == '__main__':  
    main()
```

#### 访问器和修改器
推荐的访问属性的方式。
```python
class Teacher:  
    def __init__(self, name, age, gender):  
        self._name = name  
        self._age = age  
        self.__gender = gender  
  
    # 访问器 - getter方法  
    @property  
    def name(self):  
        return self._name  
  
    # 访问器 - getter方法  
    @property  
    def gender(self):  
        return self.__gender  
  
    # 访问器 - getter方法  
    @property  
    def age(self):  
        return self._age  
  
    # 修改器 - setter方法  
    @age.setter  
    def age(self, age):  
        self._age = age  
  
  
def main():  
    s = Teacher("zhangsan", 18, 'male')  
    print(s.name)  
    print(s.gender)  
    print(s.age)  
    s.age = 21  
    print(s.age)  
  
  
if __name__ == '__main__':  
    main()
```

#### \_\_solts__
python 是动态语言，可以在运行时绑定属性和方法，如果希望 class 只能绑定某些属性和方法，可以使用 ``__slots__`` 。``__slots__`` 只在当前类有效，对子类无效。
```python
class Person:  
    __slots__ = ('_name', '_age')  
  
    def __init__(self, name, age=None):  
        self._name = name  
        if age is not None:  
            self._age = age  
  
    @property  
    def name(self):  
        return self._name  
  
    @property  
    def age(self):  
        return self._age  
  
    @age.setter  
    def age(self, age):  
        self._age = age  
  
  
def main():  
    s = Person("zhangsan")  
    print(s.name)  
    s.age = 18  
    print(s.age)  
    # s.gender = 'male'  
    # print(s.gender)  
  
if __name__ == '__main__':  
    main()
```

#### 静态方法
静态方法用 `@staticmethod` 修饰，通过类来调用。
```python
class Person:  
  
    def __init__(self, name, age):  
        self._name = name  
        self._age = age  
  
    @property  
    def name(self):  
        return self._name  
  
    @property  
    def age(self):  
        return self._age  
  
    @age.setter  
    def age(self, age):  
        self._age = age  
  
    @staticmethod  
    def is_age_valid(age):  
        return 0 < age < 120  
  
  
def main():  
    age = -1  
    if Person.is_age_valid(age):  
        s = Person("zhangsan", age)  
        print(s.name)  
        s.age = 18  
        print(s.age)  
    else:  
        print("Valid age")  
  
  
if __name__ == '__main__':  
    main()
```

#### 类方法
类方法使用 `@classmethod` 修饰，第一个参数为 `cls` 代表当前类的对象。
```python
from datetime import datetime  
  
  
class Clock:  
    @classmethod  
    def now(cls):  
        return datetime.now()  
  
  
def main():  
    current_time = Clock.now()  
    print(current_time)  
  
if __name__ == "__main__":  
    main()
```

#### 继承和多态
子类继承父类可以继承父类的属性和方法，还可以定义自己的属性和方法。
```python
from abc import ABCMeta, abstractmethod  
  
  
class Employee(object, metaclass=ABCMeta):  
    def __init__(self, id, name):  
        self.id = id  
        self.name = name  
  
    @abstractmethod  
    def work(self):  
        pass  
  
  
class Sales(Employee):  
    def __init__(self, id, name):  
        super().__init__(id, name)  
        self.type = "sales"  
  
    def work(self):  
        print("sales work")  
  
  
class RD(Employee):  
    def __init__(self, id, name):  
        super().__init__(id, name)  
        self.type = "RD"  
  
    def work(self):  
        print("RD work")  
  
  
class Car(object):  
    def work(self):  
        print("car run")  
  
def main():  
    e = Sales(1, "zhangsan")  
    print(e.id)  
    print(e.name)  
    e.work()  
    m = RD(2, "lisi")  
    print(m.id)  
    print(m.name)  
    print(m.type)  
    m.work()  
    c = Car()  
    c.work()  
  
  
if __name__ == '__main__':  
    main()
```

`abc` 是 Abstract Base Classes 模块，`ABCMeta` 是一个 metaclass ，用来创建抽象基类。`@abstractmethod` 是一个装饰器，用来修饰抽象类。子类在继承父类之后可以重写父类的方法，叫做多态。
python 是一中动态语言，只要一个对象有需要的方法和属性（比如这里的 `Car` 类具有 `work()` 方法），不需要真的继承，就可以被视为 `Employee` 类型，这叫做鸭子类型。

### 文件
| 操作模式 | 说明                                              |
| ---- | ----------------------------------------------- |
| `r`  | 读（默认）                                           |
| `w`  | 写（如果文件已经存在，`'w'` 模式会覆盖文件的内容。）                   |
| `x`  | 写（如果文件已经存在，`'x'` 模式会引发一个 `FileExistsError` 异常。） |
| `a`  | 追加                                              |
| `b`  | 二进制模式                                           |
| `t`  | 文本模式（默认）                                        |
| `+`  | 更新（可读写）                                         |
<<<<<<< HEAD


![](.\images\python_file_option.png)

=======
!["python_file_option.png"](https://raw.githubusercontent.com/bsyonline/pic/master/img/python_file_option.png)

>>>>>>> 79564e61a430deb03e3ecac80a8704b8cea00668
### 进程和线程
Python 既支持多进程又支持多线程，因此使用 Python 实现并发编程主要有3种方式：多进程、多线程、多进程+多线程。

#### 多进程
```python
from time import time, sleep  
from multiprocessing import Process  
  
  
def foo(name):  
    print(name)  
    for i in range(5):  
        print(f"{name} {i}", flush=True)  
        sleep(1)  
    print(f"End {name}")  
  
  
def bar(name):  
    print(name)  
    for i in range(10):  
        print(f"{name} {i}", flush=True)  
        sleep(1)  
    print(f"End {name}")  
  
  
def main():  
    start = time()  
    p1 = Process(target=foo, args=('foo',))  
    p1.start()  # 启动进程p1  
    p2 = Process(target=bar, args=('bar',))  
    p2.start()  # 启动进程p2  
    p1.join()  # 等待进程p1结束  
    p2.join()  # 等待进程p2结束  
    end = time()  
    print("Execution time: %.2f" % (end - start))  
  
  
if __name__ == "__main__":  
    main()
```


#### 多线程
```python
from threading import Thread  
from time import sleep, time  
  
  
def foo(name):  
    print(name)  
    for i in range(5):  
        print(f"{name} {i}", flush=True)  
        sleep(1)  
    print(f"End {name}")  
  
  
def bar(name):  
    print(name)  
    for i in range(10):  
        print(f"{name} {i}", flush=True)  
        sleep(1)  
    print(f"End {name}")  
  
  
def main():  
    start = time()  
    t1 = Thread(target=foo, args=('foo',))  
    t1.start()  
    t2 = Thread(target=bar, args=('bar',))  
    t2.start()  
    t1.join()  
    t2.join()  
    end = time()  
    print("Execution time: %.2f" % (end - start))  
  
  
if __name__ == "__main__":  
    main()
```

通过继承 Thread 类的方式也可以创建线程。
```python
from threading import Thread  
from time import sleep, time  
  
  
class Foo(Thread):  
    def __init__(self, name):  
        super().__init__()  
        self.name = name  
  
    def run(self):  
        print(self.name)  
        for i in range(5):  
            print(f"{self.name} {i}", flush=True)  
            sleep(1)  
        print(f"End {self.name}")  
  
  
def main():  
    start = time()  
    t1 = Foo('foo')  
    t1.start()  
    t2 = Foo('bar')  
    t2.start()  
    t1.join()  
    t2.join()  
    end = time()  
    print("Execution time: %.2f" % (end - start))  
  
  
if __name__ == "__main__":  
    main()
```

#### 线程同步

#### 单线程+异步I/O


[collections — Container datatypes — Python 3.12.3 documentation](https://docs.python.org/3/library/collections.html)


### 常用库
#### numpy
##### 创建数组
第一种，通过列表创建。
```python
arr1 = np.array([1, 2, 3])
```
也可以创建多维。
```python
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
```

第二种，使用 `arange` 。
```python
# 不包括3
arr3 = np.arange(1, 3)
# 1到5，步长为2
arr4 = np.arange(1, 5, 2)
```

第三种，使用 `linspace` 。
```python
# 从1到3，分成3份
arr5 = np.linspace(1, 3, 3)
```

第四种，从字符串创建。
```python
arr7 = np.fromstring("1, 2, 3", sep=",")
```

第五种，用随机数创建。
```python
arr8 = np.random.randint(1, 10, 3)
```

第六种，使用 `zeros` , `ones` , `empty` 。
```python
arr9 = np.zeros(3)
arr11 = np.ones(3)
arr13 = np.empty(3)
```
也可以创建多维数组。
```python
arr10 = np.zeros((3, 3))
arr12 = np.ones((3, 3))
arr14 = np.empty((3, 3))
```

第七种，从文件加载。
```python
np.save("data.npy", arr14)  
arr15 = np.load("data.npy")
arr14.tofile("data.txt", sep=",")  
arr16 = np.fromfile("data.txt", sep=",")
```

##### 数组的属性
size：数组元素个数。
```python
arr.size
```

shape：数组的形状。
```python
arr.shape
```

dtype：数组元素的数据类型。
```python
arr.dtype
```

ndim：数组的维度。
```python
arr.ndim
```

itemsize：数组单个元素占用内存空间的字节数。
```python
arr.itemsize
```

nbytes：数组所有元素占用内存空间的字节数。
```python
arr.nbytes
```

##### 切片索引
切片索引是形如`[开始索引:结束索引:跨度]`的语法，通过指定**开始索引**（默认值无穷小）、**结束索引**（默认值无穷大）和**跨度**（默认值1），从数组中取出指定部分的元素并构成新的数组。因为开始索引、结束索引和步长都有默认值，所以它们都可以省略，如果不指定步长，第二个冒号也可以省略。

![](https://raw.githubusercontent.com/bsyonline/pic/master/img/%E4%BA%8C%E7%BB%B4%E6%95%B0%E7%BB%84%E6%99%AE%E9%80%9A%E7%B4%A2%E5%BC%95%E5%92%8C%E5%88%87%E7%89%87%E7%B4%A2%E5%BC%95.png)

> [!NOTE] 注意
> 切片索引创建的数组和原数组共享数据。


##### 花式索引
花式索引是用保存整数的数组充当一个数组的索引。
```python
arr23 = np.arange(1, 10)  
print(arr23)  # [1 2 3 4 5 6 7 8 9]  
# 整数索引
arr24 = arr23[[0, 1, 1, -1, 4, -1]]  
print(arr24)  # [1 2 2 9 5 9]  
print(arr24.shape)  # (6,)

arr15 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
# 多维数组索引
arr25 = arr15[[0, 2], [1, 2]]  # 第一个数组是行索引，第二个数组是列索引  
print(arr25)  # [2 9]  
print(arr25.shape)
```

##### 布尔索引
布尔索引就是通过保存布尔值的数组充当一个数组的索引，布尔值为`True`的元素保留，布尔值为`False`的元素不会被选中。布尔值的数组可以手动构造，也可以通过关系运算来产生。
```python
arr26 = arr23[[True, False, True, False, True, False, True, False, True]]  
print(arr26)  # [1 3 5 7 9]  
arr27 = arr23[arr23 % 2 == 0]  
print(arr27)  # [2 4 6 8]  
arr28 = arr23[arr23 > 5]  
print(arr28)  # [6 7 8 9]  
arr29 = arr23[~(arr23 < 5)]  
print(arr29)  # [5 6 7 8 9]
```

##### 运算
```python
arr30 = np.arange(1, 10, 1)  
# 判断是否所有元素都大于0  
print(np.all(arr30 > 0))  # True  
# 转换数据类型  
print(arr30.astype(np.float32))  
# 转换形状  
print(arr30.reshape(3,3))  
# 求和  
print(arr30.sum())  # 45  
# 求平均值  
print(arr30.mean())  # 5.0  
print(np.mean(arr30))  # 5.0  
# 最大值  
print(np.max(arr30))  # 9  
# 最大值，等价于np.max()  
print(np.amax(arr30))  # 9  
# 最小值  
print(np.min(arr30))  # 1  
# 最小值，等价于np.min()  
print(np.amin(arr30))  # 1  
# 中位数，50分位数即中位数  
print(np.quantile(arr30, 0.5))  # 5.0  
# 最大值的索引  
print(arr30.argmax())  # 8  
# 最小值的索引  
print(arr30.argmin())  # 0
```

ptp，峰值到峰值（peak-to-peak）的范围（range）。这个值也常被称为数组的“全距”（range）或“跨度”（span）。
```python
# 最大值与最小值之间的差值  
print(np.ptp(arr30))  # 8
```

所有元素的方差（variance）。方差是衡量数据集中数值与其平均值（均值）之间差异程度的另一个统计量，与标准差不同，方差是每一个数与平均数的差值的平方的平均值。
```python
# 方差  
print(np.var(arr30))  # 6.666666666666667
```

std，标准差是一个衡量数据集中数值与其平均值（均值）之间差异程度的统计量。标准差=对方差求平方根
```python
# 标准差  
print(np.std(arr30))  # 2.581988897471611
```

变异系数（Coefficient of Variation，简称CV）是一种用于衡量数据相对离散程度的统计量，它反映了数据集中各观测值相对于其平均值的变异程度。计算公式为：变异系数 C·V = (标准偏差 SD / 平均值 Mean) × 100%。
```python
print(np.std(arr30) / np.mean(arr30) * 100)  # 51.63977794943222
```

