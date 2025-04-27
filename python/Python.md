# 语法

## 环境

```perl
#创建虚拟环境
python3 -m venn yukienv

#激活虚拟环境
#Mac Unix
source yukienv/bin/activate

#Windows
yukienv\Scripts\activate

#两者目录不同说明创建虚拟环境成功
python3 -c 'import sys; print(sys.prefix,system.base_prefix)'

#退出虚拟环境
deactivate

#设置pip源
pip3 config set global.index-url https:xxxxxx
```

## 集合

### list

```python
    # create
    listX = []
    listY = [1, 2, 3, 4, 'qca', False, 567]

    #增加一个元素
    listX.append(777)
    #把列表打散后加入list X
    listX.extend(listY)# [777, 1, 2, 3, 4, 'qca', False, 567]

    # update
    listX[0] = 'cqmyG'
    
    # delete
    del listX[1]
    print(listX.pop(0))  # cqmyGa
    
    # traversing
    for element in listX:
        print(f"{element}+++++++++++")
    for index in range(len(listX)):
        print(f"{listX[index]}------------")
    print(listX)#[2, 3, 4, 'qca', False, 567]
    
    #slice
    #下标从0开始，不包括最后一个下标，左闭右开区间
    # [0，2)
    print(listX[:3]) #[2, 3, 4]
    # [2,end]
    print(listX[2:]) #[4, 'qca', False, 567]
    # [3,end-1]
    print(listX[3:-1]) #['qca', False]
    # [3,end-2]
    print(listX[3:-2]) #['qca']
```

### tuple

```python
 	#create 元组一旦创建不可修改值
    tupleX=()
    tupleY=(1,2,3,4,'qca',False,567)
    tupleX=tupleX+tupleY
    print(tupleX)
    #traversing【同列表】
    for element in tupleX:
        print(f"{element}+++++++++++")
    for index in range(len(tupleX)):
        print(f"{tupleX[index]}----------")
    #slice【同列表】
    print(tupleX[1:])
    print(tupleX[:4])
    print(tupleX[2:-1])
```

### dict

```python
    #create
    dictX={}
    dictY={'name':123,'age':456,'sex':False}
    dictZ=dict(id=777,email='123.@gmail.com',name='cqmyG',password='<PASSWORD>')

    #insert&update
    dictX['country']='American'
    dictX['name']='alice'
    #合并字典方法一
    dictX.update(dictY)
    #合并字典方法二
    dictX={**dictX,**dictZ}

    #delete
    del dictX['password']

    #traversing
    for key in dictX:
        print(f"key={key},dictX[{key}]=",dictX[key])

    for value in dictX.values():
        print(f'value={value}')

    for key,value in dictX.items(): # dictX.items()返回(key,value)的二元组
        print(key,value)
```

### set

```python
    #create
    setX={'id',1,'email','<EMAIL>',1,2}
    listX=[777, 1, 2, 3, 4, 'qca', False, 567]

    #去重
    listX=list(set(listX))

    #traversing
    for element in setX:
        print(element)
```

## IO

with关键字会自动关闭连接

```python
with open() as file1,open() as file2:
	file1,file2逻辑
```

open(filename, mode, encoding)需要传递三个参数

* filename

  * 相对路径
  * 绝对路径

* mode

  * r 【read 读】

    ```python
    for line in file:
        line=line.strip()
    ```

  * w 【write 覆盖写】

  * a 【append 追加写】

  * b 【bytes 处理的是字节（读、写）】

* encoding

  * gbk
  * utf-8

```python
file.write()
#读取全部
file.read()
#逐行读取
file.readline()

with open('1.txt','r') as file:
    while True:
        line=file.readline()
        if not line:
            break
        print(line)
```

## 基本操作

- id(变量名)：打印内存地址

- 三元表达式：结果1 if 条件 else 结果2

- 列表推导式：[结果 for循环 if条件]
- 字典推导式：{key:value for循环 if条件}
- 集合推导式：{key for循环 if条件}
- 生成器推导式：(结果 for循环 if条件)  **节省内存** 元组没有推导式

- 模版字符串：

  ```python
  s='My name is %s, age is %d' % (name,int(age))
  s=f'My name is {name},weight is {weight:.2f}' #保留两位小数:.2f
  ```

- 字符集编码

  ```python
  stringx='ciallo~'
  bytex=stringx.encode('utf-8')
  stringy=bytex.decode('utf-8')
  ```

- 操作符

  ```python
  #算数运算符
  + - * / // % **  【//整除 **乘方】
  
  #逻辑运算符
  and、or、not
  is、is not 【用于判空 is None】
  in、 not in 【判断成员是否存在，如列表 字符串】
  ```

- 布尔值：所有可以表示“空”的东西都是False

  - None：False
  - 所有的数字：非零的是True，零就是False
  - 所有的字符串：非空字符串是True，空字符串是False
  - 所有的列表：非空列表是True，空列表[]是False

## 字符串

```python
stringx='hello'
stirngy='world'

#slice
print(stringx[1:]) #ello
print(stringx[:-1]) #hell
print(stirngy[2:4]) #rl

#replace,join
print(stringx.replace('e','E'))
print(stringx.replace('or','OR').replace('ld','LD'))
print(stringx.join(stirngy))    #whelloohellorhellolhellod

#find
print(stringx.find('ll'))
print(stringx.index('ll'))

#trim
#删除前后空格
print(stringx.strip())
#删除左边空格
print(stringx.lstrip())
#删除右边空格
print(stringx.rstrip())

#split
#return list
words=stringx.split(',')

#assert
print(stringx.startswith('h'))
print(stringx.endswith('x'))

#uppercase or lowercase
print(stringx.upper())
print(stringx.lower())
#首字母大写
print(stringx.capitalize())
#每个单词的首字母大写
print(stringx.title())
```

## 函数

### 参数

```python
optionx='ciallo~'

#形参可以设置默认值，需要写末尾
#可以用形参:类型 标注变量类型【可以让IDE给出方法提示】
def student(name:str,sex,age=12):
    #若要修改全局参数的值，使用global引入全局参数，不修改时不需要引入，尽量少用！！！
    global optionx
    optionx=721
    print(optionx,name,age,sex)

def plus1():
    a=10
    def plus2():
        #nolocal引入局部变量a 不需要修改时不需要加
        nonlocal a
        a=7
        def plus3():
            #会一层一层找plus2中没有就去plus1找，直到找到最外层的局部，不会去找全局
            nonlocal a
            a=20
            print(f'{a}-----plus3')
        print(f'{a}-----plus2')
        plus3()
    print(f'{a}-----plus1')
    plus2()

#动态传参【无敌传参：动态元组+动态字典】可以接收一切参数
def animal(*classify,**quality):
    print(classify,quality)

#定义形参时 位置参数 > *args > 默认值参数 > **kwargs
def example(id,name,*args,sex=False,**kwargs):
    print(id,name,args,sex,kwargs)

if __name__ == '__main__':
    # 位置参数需要严格遵循顺序，指定参数【无视顺序】
    student('Alice',age=14,sex=False)
    plus1()
    #*动态接收所有位置参数，接收到的内容会自动组织成元组
    animal('bird','lion','eagle',food='grass',habitat='forest',position='Aisa')
    classify=['birfx','lionx','eaglex']
    qualitat={'food':'grass','habitat':'forest','position':'Aisax'}
    #实参中 *可以把一个列表打散，**可以把一个字典打散 变成单个元素传给形参
    animal(*classify,**qualitat)
    
    example(1,'Alice',True,23333,'Darkness','Sakura',age=19,classx=2,teacher='Li')
```

### 返回值

```python
#多返回值会返回一个元组
def operator(a,b):
    return a+b,a-b,a*b,a/b

#参数可以传传函数，函数名本身是变量名
#变量()表示函数调用，变量不加括号时表示变量
def func(gunc):
    gunc()
    
if __name__ == '__main__':
    print(operator(2,4))
    #可以使用多变量接收一个集合
    a,b,c,d = operator(2,7)
    print(a,b,c,d)
    a,b,c,d=[1,2,3,4]
    print(a,b,c,d)
    a,b,c,d={1,2,3,4}
    print(a,b,c,d)
```

## 装饰器

在不改变原有函数的基础上，给函数增加新的功能（类似代理模式）

### **装饰器通用写法**

```python

def wrapper(func):#wrapper是装饰器的名字，func是被装饰的函数
    #inner：装饰后的函数，替代原函数
    def inner(*args,**kwargs):#为适应各种函数的参数排列组合，需要用无敌传参
        。。。。。#增加函数调用前的逻辑（如记录方法调用前的日志）
        result=func(*args,**kwargs) #执行被装饰的函数，参数原封不动传回去
        。。。。。#增加函数调用后的逻辑（如记录方法调用后的日志）
        return result #返回原函数的返回值
    return inner #返回被装饰的函数

@wrapper
def func(*args,**kwargs):
    pass
      
if __name__ == '__main__':
    func()#此时调用的func时装饰器的inner函数  
```

### 多个装饰器

多个装饰器，遵循就近原则，gua1装饰game返回inner，gua2装饰gua1的inner，gua3装饰gua2的inner

```python
def gua1(game):
    def kaigua(*args,**kwargs):
        print('外挂1开启....')
        result=game(*args,**kwargs)
        print('外挂1关闭....')
        return result
    return kaigua

def gua2(game):
    def kaigua(*args,**kwargs):
        print('外挂2开启....')
        result=game(*args,**kwargs)
        print('外挂2关闭....')
        return result
    return kaigua

def gua3(game):
    def kaigua(*args,**kwargs):
        print('外挂3开启....')
        result=game(*args,**kwargs)
        print('外挂3关闭....')
        return result
    return kaigua


'''
外挂3开启....
外挂2开启....
外挂1开启....
原神，启动！！！
外挂1关闭....
外挂2关闭....
外挂3关闭....
'''
#多个装饰器，遵循就近原则，gua1装饰game返回inner，gua2装饰gua1的inner，gua3装饰gua2的inner
@gua3
@gua2
@gua1
def genjin_impact()->None:
    print('原神，启动！！！')
```

### 带参数的装饰器

```python
def cheats(name):
    def gua(game):
        def kaigua(*args,**kwargs):
            print(f'外挂————{name}————启动！')
            result=game(*args,**kwargs)
            print(f'外挂————{name}————关闭！')
            return result
        return kaigua
    return gua

'''
外挂————耄耋————启动！
dnf 启动！
外挂————耄耋————关闭！'''
@cheats('耄耋')#cheats('耄耋')相当于调用零cheats函数，会返回gua的装饰器【@cheats('耄耋')相当于@gua】
def dnf()->None:
    print('dnf 启动！')

'''
外挂————饕餮————启动！
cf 启动！
外挂————饕餮————关闭！'''
@cheats('饕餮')
def cf()->None:
    print('cf 启动！')
```

## 迭代器

```python
#特别省内存
def iterx(collection):
    it=collection.__iter__()
    while 1:
        try:
            print(it.__next__())
        except StopIteration:
            break

if __name__ == '__main__':
    listx=[1,2,3,4,'111']
    tuplex=('1','2','3','a')
    dictx={'a':1,'b':2,'c':3,'d':4}
    setx={1,1,2,3,4,5,6,7,8,9,10}
    iterx(listx)
    iterx(tuplex)
    iterx(dictx)
    iterx(setx)
```

## 生成器

生成器的本质是迭代器，需\_\_next\_\_( )迭代，每次使用\__next__( )迭代，函数会执行到下一个yield为止，并返回yield的值

返回列表时，不需要把整个列表返回，可以一次返回一个元素，非常省内存

```python
def generator():
    for i in range(1,10):
        print(233*i)    #每次迭代时执行
        yield i*7

if __name__ == '__main__':
    g=generator()
    print(g.__next__())
    print(g.__next__())
```

## 推导式

列表推导式：[结果 for循环 if条件]

字典推导式：{key:value for循环 if条件}

集合推导式：{key for循环 if条件}

元组没有推导式

生成器表达式：(结果 for循环 if条件)

```python
def comprehension():
    source=['sh','bj','tokyo','american',233,233]

    listx=[i for i in source if len(str(i))>2]
    print(listx)#['tokyo', 'american', 233, 233]

    dictx={"city":i for i in source if type(i)==type("")}
    print(dictx)#{'city': 'american'}

    setx={str(i)+'xx' for i in source if len(str(i))>2}
    print(setx)#{'233xx', 'americanxx', 'tokyoxx'}

    gx=(source[i] for i in range(len(source)) if str(source[i]).startswith('b'))
    #list(迭代器)会直接便利迭代器内所有值
    print(list(gx))#['bj']

if __name__ == '__main__':
    comprehension()
```

## Lambda

语法：lambda 参数1 参数2: 返回值

结合内置函数使用，内置函数返回迭代器对象

- filter(函数，可迭代对象) 将可迭代对象的每一个元素传递 作为参数传递给函数执行，根据执行的结果选择保留还是删除
- map(函数，可迭代独享) 映射
- sorted(可迭代对象，key，reverse)
  - key：key=函数 把可迭代对象的内容传递给key函数，根据key函数的执行结果进行排序
  - reverse：reverse=False/True，是否翻转，默认False，从小到大

```python
source=['上海','阿斯顿马丁','北京','乌鲁木齐','莫斯科']

filterx=filter(lambda x:len(x)>3,source)
print(list(filterx))#['阿斯顿马丁', '乌鲁木齐']

mapx=map(lambda x:f'{x}-777',source)
print(list(mapx))#['上海-777', '阿斯顿马丁-777', '北京-777', '乌鲁木齐-777', '莫斯科-777']

sortedx=sorted(source,key=lambda x:len(x),reverse=True)
print(sortedx)#['阿斯顿马丁', '乌鲁木齐', '莫斯科', '上海', '北京']
```

## 异常处理

```python
import traceback

try:
    a=1/0
except NotImplementedError as e:
    print(123)
    #raise关键字 抛出异常
    raise ZeroDivisionError(e)

except ZeroDivisionError as e:
    #打印队战信息
    print(traceback.format_exc())
finally:
    print('successfully')
```

## 面向对象

**启动文件一定要在引入的包外面！！！**

```python
import xxxx
或
import xxx from xxxx
```

### 创建对象

对象=类名( )，类名后面这个小括号实际上是\__init__( )

对象.属性

对象.方法( )

注意：对象.xxx在对象中找xxx，如果找到了，就运行xxx，如果找不到，回去类中查找xxx

### 类的成员

- 类变量

  - 对象.类变量（可以访问）
  - 类.类变量（可以访问和修改）

- 实例方法：**第一个参数必须是self**

  - def func(self, arg1, arg2)

- 类方法：一般用于初始化对象

  - 在方法上增加@classmethod
  - 该方法**第一个参数是cls**, 表示当前类
  - 类方法一般用类调用，不用对象调用

- 静态方法：声明在类中的函数

  - 在方法上面增加@staticmethod
  - 使用场景：创建和初始化一个对象

- 属性方法：将方法变成一个属性，调用方式：对象.方法名【等同于对象.属性】

  ```python
  @property
  def 属性名(self):
      pass
  
  @属性名.setter	#使用场景不多
  def xxxx(self, 变量)
      pass
  
  对象.属性名=xxxx
  ```

- 特殊方法：每个类中都内置了一大堆特殊方法，这些方法在特殊场景下自动被调用

  > 有\__方法名__的都是内置的特殊方法，可以使用方法名( )调用

  - `__len__()`: len()

  - `__iter__()`: iter()

  - `__getitem__()`:对象[‘xxx’], list[2]

  -  `__del__()`: del 对象

  - `__add__()`: 对象1+对象2  xxx+xxx 自动调用\__add__

  - `__call__()`: 对象( )

  - `__enter__()`: 进入with的时候调用

  - `__exit__()`: 离开with的时候

  - `__new__()`: 创建对象的时候分配内存

  - `__init__()`: 初始化对象的时候自动调用

    ```python
    #构建对象的过程：__new__() => __init__()
    def __new__(cls, *args, **kwargs):	#创建对象的地方
        print('创建对象')
        return object.__new__(cls)		#分配内存，创建一个对象
    
    def __init__(self):
        print('init')
    ```

### 继承

class 类(父类1，父类2…)

- 子类会继承父类中除私有方法的所有其他内容

- 双下划线开头命名的变量表示私有，无法被继承

- 子类对象调用父类方法，【哪个类的下一个：A–>B–>C 调用C的方法，super里写B】

  ```python
  super(哪个类的下一个, self).方法()
  ```

- 运行时 注意self是哪个类的对象，如果是子类的，即使传回父类中，也是调用自类的方法

- python中所有的类都会继承Object

  ```python
  类.mro() #查看方法的查找路径
  ```

## 反射

重点：

- hasattr(obj, name): 从obj对象中判断是否存在某个属性或功能
- getattr(obj, name): 从obj对象中获取某个属性或功能

了解：

- setattr(obj, name, value): 动态给某个obj对象设置某个属性或方法，值为value
- delator(obj, name): 从obj对象中删除某个属性，慎用！！！

## 并发
