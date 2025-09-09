---
title: 'python学习过程心得体会'
lastmod: 2025-08-18
categories: ['笔记']
tags: ['python','脚本语言']
cover: https://kidle9527.github.io/images/88.png
---

# 快速过一遍python的知识~

## 基本语法

* 解释型：Python是边解释边执行的，Python解释器会将源代码转换为中间字节码形式，然后将其解释为机器语言并执行

* python 3采用双字节Unicode编码，包含亚洲文字编码，如中文、日文、韩文等字符。

* Python中标识符的命名规则如下。
  1 区分大小写：Myname与myname是两个不同的标识符。
  2 首字符可以是下画线（_）或字母，但不能是数字。
  3 除首字符外的其他字符必须是下画线、字母和数字。
  4 关键字不能作为标识符。
  5 不要使用Python的内置函数作为自己的标识符

* python只有33个关键字，其中False、None、True的首字母大写，其他关键字全都是小写
* python语句结束时可以加分号，但不符合python编程规范
* `a = b = c = 10`，支持链式赋值
* 代码注释：`# 这是一个注释`
* 在Python中一个模块就是一个文件，模块是保存代码的最小单位，在模块中可以声明变量、函数、属性和类等Python代码元素。

```
1. import m2 # m2是一个模块，通过这种方式导入模块m2中的所有代码元素，在访问时需要加前缀m2.
eg：print(m2.x)

2. from m2 import x # 从m2中导入x变量，在访问时不需要加前缀m2.

3.from m2 import x as x2 # 类似2，只是可能由于变量名冲突，起了个别名
```

---

## 数据类型

​	python内置了主要6种数据类型：数字、字符串、列表、元组、集合和字典，其中：列表、元组、集合和字典可以容纳多项数据

### 数字类型

​	数字类型主要有4种：整数类型、浮点类型、复数类型和布尔类型（布尔类型事实上是整数类型的一种）

```
二进制表示方法，以阿拉伯数字0和英文字母b作为前缀（都不区分大小写）: 0b11100 -> 28
八进制表示方法，以阿拉伯数字0和英文字母o作为前缀: 0o34 -> 28
十六进制表示方法，以阿拉伯数字0和字母x作为前缀：0x1c -> 28
```



* 浮点类型主要用于存储小数数值，python的浮点类型为float，只支持双精度浮点类型

```
3.36e2 = 336.0;  .56e-2 = 0.0056  --->科学表示法，会使用e/E表示10的指数
```

* 复数类型：复数在数学中被表示为a + bi, 其中a为实部，b为虚部，i被称为虚数单位

~~~
python中的复数格式：a + bj,这里的虚数单位是j，类型是：complex
~~~

* python中的布尔类型为bool，bool是int类型的子类，只有两个值：True和False

```
bool(0) \ bool('') \ bool([]) \ bool({}) = False ---> 整数0、空字符串、空列表、空字典被判False
```

* 在python的数字类型中，除了复数以外，其他三种数字类型都可以相互转换，分为隐式转换和显示转换

~~~
隐式转换：
a = 1 + True --> 2
a = 1.0 + 1 ---> 2.0
a = 1.0 + True ---> 2.0
a = 1.0 + 1 + False --> 2.0
总结一下：转换成bool类型的优先级最低；转换成int还是float主要跟随等式的第一个变量
~~~

* 显示类型的转换
  * 函数int()、float()、bool()，就是强制类型转换嘛~
  * 注意：int(0.9) = 0; float(2) = 2.0

---

## 容器类型的数据

​	python内置的数据类型如序列（列表、元组）、集合和字典等都是可以容纳多项数据，称之为容器类型的数据

### 1. 序列（sequence）

​	是一种可迭代的、元素有序的容器类型的数据；序列包括列表（list）、字符串（str）、元组（tuple）和字节序列（bytes)

* 1.1序列的索引操作

```
正值索引：第一个元素的索引是0，向右递增
负值索引：最后一个元素的索引是-1，向左递减
若索引超过范围，则会发生IndexError错误
max()函数用户返回最后一个元素；min()函数用于返回第一个元素；len（）函数用于获取序列的长度
```

* 1.2加和乘操作

~~~
'hello' + 'world' = 'helloworld'
'hello' * 2 = 'hellohello'
~~~

* 1.3切片操作

  * 序列的切片（Slicing)就是从序列中切分出小的子序列
    * 切片的语法：`[start : end : step]`
    * start是开始索引，end是结束suoyin，step是步长（切片时获取的元素的间隔，可正可负）
    * 切片包括start不包括end，start和end都可以省略；步长省略默认值为1
    * `步长为负值时，从右向左获取元素`

  ~~~
  a = 'hello'
  a[1:3] = 'el'
  a[:3] = 'hel'	#默认开始索引为0
  a[1:] = 'ello'	#默认结束索引是序列长度
  a[:] = 'hello'
  a[1:-1] = 'ell'	#支持负值索引
  a[0:5:2] = 'hlo'#修改步长为2
  a[::-1]		#步长为负值时，从右向左获取元素！！！！
  ~~~

* 1.4成员测试

​		成员测试运算符有两个：`in`和`not in`，用于测试元素是否被包含在序列中，`返回值是bool`

### 2.列表

​	列表是一种可变序列类型，可以追加、插入、删除和替换列表中的元素

* 2.1创建列表

  * list（iterable）函数：参数iterable是可迭代对象（字符串、列表、元组、集合和字典等）
  * [元素1，元素2，元素3，⋯]：指定具体的列表元素，元素之间以逗号分隔，列表元素需要使用中括号括起来。

  ```
  x = list(('hello', 'worlf', 1, 3, 5)) --->['hello', 'worlf', 1, 3, 5] #从元组创建字符串列表
  x = ['hello', 'worlf', 1, 3, 5] ---> 直接用方括号
  z = list('hello') ---> z = ['h', 'e', 'l', 'l', 'o']
  ```

* 2.2追加元素

  * 在列表中追加单个元素时，可以使用列表的append（x）方法
  * 在列表中追加多个元素时，可以使用加（+）运算符或列表的extend()方法

  ~~~
  a = [1, 2, 3]
  a.append(4)  ->	a = [1, 2, 3, 4]
  b = [9, 8, 7]
  a += b	->	a = [1, 2, 3, 4, 9, 8, 7]
  a.extend(a)	->	a = [1, 2, 3, 4, 9, 8, 7, 1, 2, 3]
  ~~~

  注意，追加元素是在尾巴追加；

  方法和函数的区别：

  ​	方法隶属于类，通过类或者对象调用方法；函数不属于任何类，直接调用

  ​	表现在：

  ​		方法： a.方法

  ​		函数：函数(a)

* 2.3插入元素

  * 使用列表的list.insert(i, x)方法
  * i指定索引位置；x是要插入的元素

  ~~~
  [1,2,3].insert(2,999) = [1,2,999,3]
  ~~~

* 2.4替换元素

​	`list[1]  = 985`

* 2.5删除元素
  * 使用方法list.remove(x)
  * x是想要删除的元素，匹配到多个相同的元素，则**只会删除第一个匹配的元素**

### 3.元组

​	元组（tuple)是一种不可变序列类型

* 3.1创建元组

  * tuple（iterable）函数：参数iterable是可迭代对象（字符串、列表、元组、集合和字典等）
  * (元素1，元素2，元素3，⋯）：指定具体的元组元素，元素之间以逗号分隔。对于元组元素，可以使用小括号括起来，也可以省略小括号。

  ~~~
  x = (1, 2, 'hello') = 1, 2, 'hello' --> (1, 2, 'hello') # 当元组中有多个元素可以省略圆括号
  x = (42,) --> (42,) # 单个元素的元组需要在元素后加一个逗号，否则会被识别为普通值
  x = tuple("hello")     # 输出 ('h', 'e', 'l', 'l', 'o')
  x = () \ x = tuple()  # 创建空元组
  ~~~

* 3.2元组拆包

  * 创建元组，并将多个数据放到元组中，这个过程被称为元组打包。
  * 与元组打包相反的操作是拆包，就是将元组中的元素取出，分别赋值给不同的变量。

  ~~~
  a, b = ('wjl', 11)
  a = 'wjl'
  b = 11  --> 元组的拆包
  所有容器类型的数据中都可以保存任意类型的数据~
  ~~~

### 4.集合（类似unordered_set)

​	集合（set)是一种可迭代、无序、不能包含重复元素的容器类型的数据

* 4.1创建集合

  * set（iterable）函数：参数iterable是可迭代对象（字符串、列表、元组、集合和字典等）。
  * {元素1，元素2，元素3，⋯}：指定具体的集合元素，元素之间以逗号分隔。对于集合元素，需要使用大括号括起来

  ~~~
  x = {1, 2, 'wjl'} 等价于 x = set([1, 2, 'wjl']) 等价于 x = set((1, 2, 'wjl'))
  ~~~

* 4.2修改集合

  * 修改集合类似于修改列表，可以向其中插入和删除元素。
    * add（elem）：添加元素，如果元素已经存在，则不能添加，不会抛出错误。
    * remove（elem）：删除元素，如果元素不存在，则抛出错误
    * clear（）：清除集合。

  ~~~
  x = {1, 2, 99, 'wjl', 'hello'}
  x.add('nb') --> x = {1, 2, 99, 'wjl', 'hello', 'nb'}
  x.remove(2) --> x = {1, 99, 'wjl', 'hello', 'nb'}
  x.clear()  ---> x = set() # 代表空集合
  ~~~

### 5.字典

​	字典（dict）是可迭代的、通过键（key）来访问元素的可变的容器类型的数据。
​	字典由两部分视图构成：**键视图和值视图**。键视图不能包含重复的元素，值视图能。在键视图中，键和值是成对出现的。**值可以重复，键不可以**

* 5.1创建字典

  * dict（）函数
  * {key1：value1，key2：value2，...，key_n：value_n}：指定具体的字典键值对，键值对之间以**逗号**分隔，最后用大括号括起来。

  ~~~
  x = {'name' : 'wjl', 'age' : 25, 'gender' : 'man'}	# 使用大括号直接创建
  person = dict(name='Alice', age=25, city='New York')	# 使用关键字参数创建
  person = dict([('name', 'Alice'), ('age', 25), ('city', 'New York')])  #使用列表创建
  
  # 使用zip函数将两个可迭代对象打包成元组，第一个参数是键，第二个参数是值
  x = dict(zip([1,2,3], ['nb', 'wc', 'nt'])) -->	{1: 'nb', 2: 'wc', 3: 'nt'}
  
  # 为多个键设置相同的默认值
  keys = ['name', 'age', 'city']
  default_dict = dict.fromkeys(keys, None)  # {'name': None, 'age': None, 'city': None}
  ~~~

  ​	字典的创建方法很多，还可以通过字典推导式、fromkeys()方法等创建......

* 5.2修改字典

  ​	字典可以被修改，但都是针对键和值同时操作的，对字典的修改包括添加、替换和删除。

  ~~~
  x = {1: 'nb', 2: 'wc', 3: 'nt'}
  x[1] = 'wjl_nb' -> 	x = {1: 'wjl_nb', 2: 'wc', 3: 'nt'}	# 修改
  x[4] = '小乐' -> x = {1: 'nb', 2: 'wc', 3: 'nt', 4: '小乐'}	# 添加
  x.pop(3) -> x = {1: 'nb', 2: 'wc', 4: '小乐'} # 删除
  
  dict.pop(key)方法删除键值对，返回删除的值；如果key不存在，会抛出异常
  
  # 使用get()方法访问（键不存在时返回None或默认值）
  person = {'name': 'Alice', 'age': 25, 'city': 'New York'}
  print(person.get('age'))  # 输出: 25
  print(person.get('country', 'USA'))  # 输出: USA（默认值）
  
  # 使用update()方法批量更新
  person.update({'age': 27, 'country': 'USA'})
  
  # 删除指定键值对, 键不存在也会报错
  del person['city']
  
  # 使用popitem()删除并返回最后插入的键值对（Python 3.7+保证顺序）
  last_item = person.popitem()
  
  # 清空字典
  person.clear()
  
  # 删除整个字典
  del person
  ~~~

  总结：

  * 查找：x[key] 和 x.get(key)
  * 添加/修改：x[key] = value 和 x.update()批量更新
  * 删除：del x[key] 和 x.pop(key)
  * 清空字典：x.clear(); 删除字典：del x

* 5.3访问字典视图

​	可以通过字典中的三种方法访问字典视图：`x.items（）`：返回字典的所有键值对视图。`x.keys（）`：返回字典键视图。`x.values（）`：返回字典值视图。

* 5.4遍历字典

​	字典有两个视图，在遍历时可以只遍历值视图，也可以只遍历键视图，还可以同时遍历，都是通过for循环实现。

```
遍历键：for i in x.keys()
遍历值：for i in x.values()
遍历键值对：for k, v in x.items()
```
---

## 字符串

​	在python中，字符串（str)是一种不可变的字符序列，因此序列的所有操作完全适用于字符串。字符串中的字符采用Unicode编码表示

* 字符转义：如果想在字符串中包含一些特殊的字符，比如换行符、制表符的等，需要在前面加上反斜杠（\)，这叫做字符转义

```
常见的转义符：
\t -> 水平制表符
\n -> 换行符
\r -> 回车
\" -> 双引号
\' -> 单引号
\\ -> 反斜杠
```

### 6.1字符串的表示方式

* 字符串有三种表示方式：普通字符串、原始字符串和长字符串

  * 6.1.1普通字符串

    * 普通字符串是指用单引号（') 或双引号（")括起来的字符串

  * 6.1.2原始字符串（raw string)

    * 原始字符串中的特殊字符不需要被转义，按照字符串的本来样子呈现，在普通字符串的前面加`r`就是原始字符串

    ```
    'hello\tworld' -> 'hello	world'	#普通字符串
    r'hello\twolrd' -> 'hello\tworld'  # 原始字符串
    ```

  * 6.1.3长字符串

    * 如果要使用字符串表示一篇文章，其中**包含了换行、缩进等排版字符**，则可以使用长字符串表示。对于长字符串，要使用三个单引号（'''）或三个双引号（＂＂＂）括起来，这也是实现多行注释的手段，实际上是未赋值的字符串

### 6.2字符串与数字的相互转换

* 字符串和数字是不兼容的数据结构，不能够进行隐式转换，只能通过函数进行显示转换
* 6.2.1字符串转换成数字--->使用int()和float() 函数，如果不成功会引发异常
  * `int("AB") ` --> 异常，按照十进制无法转换“AB”
  * `int("AB",  16)` --> 171, 按照十六进制转换"AB"
  * 默认情况下，Int() 函数都将字符串参数当作十进制数字进行转换，**该系列函数可以指定基数（进制）**
* 6.2.2数字转换成字符串--->使用str() 函数，该函数可以将很多种类型的数据转换成字符串

### 6.3格式化字符串

* 可以使用字符串的`format()方法`，它不仅可以实现字符串的拼接，还可以格式化字符串，例如在计算需要保留小数点后几位，数字需要对齐等情况时，都可以使用该方法

* 6.3.1使用占位符

  * 要想将表达式的计算结果插入字符串中，则需要用到占位符。对于占位符，使用一对大括号（{}）表示
    * 在占位符中可以有参数序号，序号从0开始。


    ~~~
    '{} * {} = 144'.format(12, 12) --> '12 * 12  144'
    '{1} - {0} = 144'.format(2, 146) --> '146 - 2 = 144'	# 参数序号占位
    '{p1} + {p2} = {p3}'.format(p1=1, p2=3, p3=4) --> '1 + 3 = 4'	#参数名占位
    ~~~
    
    * 占位符实现高级格式化功能
    
    ~~~
    pi = 3.1415926
    print("PI值: {:.2f}".format(pi))      # 保留2位小数: PI值: 3.14	#控制小数位数
    print("百分比显示: {:.3%}".format(pi))	# 百分比显示: 314.159%	# 百分比显示
    
    number = 123456789
    print("人口: {:,}".format(number))     # 输出: 人口: 123,456,789	# 千位分隔符
    
    print("'{:>10}'".format(123))        # 输出: '       123'		# 右对齐（默认）
    print("'{:<10}'".format(123))        # 输出: '123       '		# 左对齐
    print("'{:^10}'".format(123))        # 输出: '   123    '		# 居中对齐
    print("'{:*^10}'".format(123))       # 输出: '***123****'		# 指定填充字符
    ~~~
    
    * 实现进制转换，d十进制（小写），x十六进制（可大小写），o八进制（小写），b二进制
      * `print("十进制: {:d}".format(255))    # 输出: 十进制: 255`
    * 截断字符串
    
    ~~~
    long_text = "这是一个很长的字符串示例"
    print("{:.5}".format(long_text))      # 输出: 这是一个很
    ~~~

  * 6.3.2格式化控制符

    * 在占位符中还可以有格式化控制符，对字符串的格式进行更加精准的控制。

    * 字符串的格式化控制符及其说明如下表所示。

      * 格式化控制符位于占位符索引或占位符名字的后面，之间用冒号分隔，语法：{参数序号：格式控制符}或{参数名：格式控制符}。注意**参数序号和冒号之间不能有空格**
      * 补充：s，字符串；d，十进制整数；f、F，十进制浮点数；g、G，十进制整数或浮点数；e、E，科学计数法表示浮点数；

      ~~~
      '{:0.2f}'.format(5834,5678) = '5834.57'
      '{:e}'.format(5834,5678) = '5.834568e+03'	# 科学表示法
      ~~~

### 6.4操作字符串

  * 6.4.1字符串查找
      * 字符串的find（）方法用于查找子字符串。该方法的语法为`str.find（sub[，start[，end]]）`，表示：在索引start到end之间(包含start，不包含end）查找子字符串sub，如果找到，则返回**最左端位置**的索引；如果没有找到，则返回-1
        * 注意，在python文档中[] 方括号中的内容表示可以省略，即[，start[，end]]可以省略


    ~~~
    str = 'hello,world!'
    str.find('l') = 2
    str.find('l', 7) = 9	# 从start = 7开始搜索
    str.find('l', 4, 6) = -1	# 在'0,w'中没找到
    ~~~

  * 6.4.2字符串替换

    * 若想进行字符串替换，则可以使用replace（）方法替换匹配的子字符串，返回值是替换之后的字符串。该方法的语法为`str.replace（old，new[，count]）`，表示：用new子字符串替换old子字符串。count参数指定了替换old子字符串的个数，如果count被省略，则替换所有old子字符串。

    ~~~
    str = 'ab cd ef gh ij'
    str.replace(' ', '|', 3) --> str = 'ab|cd|ef|gh ij'
    str.replace(' ', '*') --> str = 'ab*cd*ef*gh*ij'
    ~~~

  * 6.4.3字符串分割

    * 若想进行字符串分割，则可以使用split（）方法，按照子字符串来分割字符串，返回字符串列表对象。该方法的语法为`str.split（sep=None，maxsplit=-1）`，表示：使用sep子字符串分割字符串str。maxsplit是最大分割次数，如果maxsplit被省略，默认值-1则表示不限制分割次数。

    ~~~
    test = "Hello world from Python"
    
    # 默认按空白字符分割（空格、制表符、换行符等）
    result = text.split()	# 输出: ['Hello', 'world', 'from', 'Python']
    result = test.split(' ', 2) --> result = ['Hello', 'world', 'from Python']	# 限制分割次数
    ~~~


---

## 函数

​	在程序中需要反复执行的某些代码，可以使用函数进行封装。函数具有函数名、参数和返回值。

​	Python中的函数十分灵活，可以在模块中但是类之外定义，作用域是当前模块就称之为**函数**；也可以在别的函数中定义，称之为**嵌套函数**；还可以在类中定义，称之为**方法**

### 7.1定义函数

​	自定义函数的语法格式如下：

~~~
def 函数名（形式参数列表）:	# 注意形参和实参的区别
	函数体
	return 返回值
~~~

### 7.2调用函数

* 7.2.1使用位置参数调用函数
* 7.2.2使用关键字参数调用函数

~~~
def lele_nb(width, length):
	return width * length
	
r_area = lele_nb(250, 520)	# 位参，width = 250， length = 520
r_area = lele_nb(width = 234, length = 789)	# 关键字参数
~~~

### 7.3参数的默认值

* 函数重载会增加代码量，所以***在Python中没有函数重载的概念***， 而是为函数的参数提供默认值实现的

~~~
def test(name = 'wjl'):
	print('{} is nb'.format(name))	# 等价于	print(f'{name} is nb')
	
test() -->	wjl is nb
test('le') --> le is nb
~~~

* 主要优点：提高灵活性、简化调用而且向后兼容

* 非常重要的注意事项：默认参数只计算一次！

  * **规则：默认参数的值在函数被定义时（即 `def`语句执行时）计算并创建一次，之后的所有调用都会使用同一个最初创建的对象**
  * 这通常不会引起问题，除非你的默认值是**可变对象（Mutable Object）**，如列表（`list`）、字典（`dict`）或自定义的类实例。

  ~~~
  def append_to_list(value, my_list=[]): # 默认值 [] 在定义时被创建
      my_list.append(value)
      return my_list
  
  # 第一次调用，看起来很正常
  result1 = append_to_list(1)
  print(result1)  # 输出: [1]
  
  # 第二次调用，你期望得到 [2]，但实际并非如此！
  result2 = append_to_list(2)
  print(result2)  # 输出: [1, 2]
  # 因为两次函数调用中，参数 my_list使用的都是同一个默认列表对象。第一次调用后，这个列表变成了 [1]。第二次调用是在这个已经修改过的列表基础上再追加 2。
  ~~~

### 7.4可变参数

​	python中的函数可以定义接收不确定数量的参数，这种参数被称为可变参数。

* 可变参数有两种，即在参数前加 * 或者 ** 。

  * 7.4.1`*args`用于接收任意数量的**位置参数**，这些参数会被打包成一个**元组（tuple）** 传递给函数。

  ~~~
  def sum_numbers(*args):	"""计算任意个数字的和"""
      total = 0
      for num in args:
          total += num
      return total
  
  # 调用示例
  print(sum_numbers(1, 2, 3, 4, 5)) # 输出: 15
  print(sum_numbers())              # 输出: 0 (没有参数时，args 是空元组 ())
  ~~~

  * 7.4.2`**kwargs`用于接收任意数量的**关键字参数**，这些参数会被打包成一个字典（dict）传递给函数.

  ~~~
  def print_user_info(**kwargs):	 """打印用户信息"""
      for key, value in kwargs.items():
          print(f"{key}: {value}")
  
  # 调用示例
  print_user_info(name="Alice", age=25, city="New York")
  # 输出:	name: Alice, age: 25, city: New York
  
  print_user_info(username="bob", is_active=True)
  # 输出:	username: bob, is_active: True
  ~~~

  * 7.4.3可以在同一个函数中同时使用这两种可变参数，但必须遵循严格的顺序规则：
    * **正确顺序：** `(普通参数, *args, 关键字参数, **kwargs)`

|     特性     |        `*args`         |         `**kwargs`         |
| :----------: | :--------------------: | :------------------------: |
|   **用途**   |  接收任意数量位置参数  |   接收任意数量关键字参数   |
|  **打包成**  |      元组 (tuple)      |        字典 (dict)         |
| **参数顺序** |   必须在普通参数之后   |     必须在所有参数之后     |
| **典型应用** | 数学运算、装饰器、继承 | 配置选项、元数据、函数包装 |

### 7.5函数中变量的作用域

​	变量可以在**模块**中创建，作用域（变量的有效范围）是整个模块，被称为**全局变量**。变量也可以在**函数中**创建，在默认情况下作用域是整个函数，被称为**局部变量**。

* 对于函数中和模块中的同名变量，因为会发生命名冲突，函数中的同名变量会屏蔽模块中的同名变量

~~~
inx = 10

def le():
    inx = 88
    print('函数内部inx = {}'.format(inx))

le()	-->	函数内部inx = 88
print(f'模块中的inx={inx}')	-->	模块中的inx=10
~~~

* 如果想在函数内部访问被屏蔽的全局变量，使用关键字`global`，能够将函数内的变量提升为全局变量
* 对于嵌套函数，如果想修改外部函数的变量（非全局变量），使用 `nonlocal`关键字。

~~~
def outer():
    count = 0  # 外部函数的局部变量
    
    def inner():
        nonlocal count  # 声明 count 是外部函数的变量
        count += 1	# 可以理解为这里操作的是外部函数的变量而不是在内部函数新建的变量
        print("内部函数:", count)
    
    inner()
    print("外部函数:", count)

outer()
---
输出结果：	内部函数: 1	；外部函数: 1
~~~

### 7.6函数类型

​	Python中的任意一个函数都有数据类型，这种数据类型是function，被称为函数类型。

* 7.6.1理解函数类型

  * 函数类型的数据与其他类型的数据是一样的，任意类型的数据都可以作为函数返回值使用，还可以作为函数参数使用。因此，一个函数可以作为另一个函数返回值使用，也可以作为另一个函数参数使用。

  ~~~
  def add(a, b):
  	return a + b
  def sub(a, b):
  	return a - b
  	
  def calculateXX(opr):	# 定义计算函数，返回function类型的数据，即另一个函数add或sub
  	if opr == '+':
  		return add
  	else:
  		return sub
  		
  f1 = calculateXX('+') --> add()
  f2 = calculateXX('-') --> sub()
  
  print("10 + 5 = {}".format(f1(10, 5)))	# f1(10, 5)是调用发指向的add()函数，相当于调用add(10, 5)
  
  ~~~

  * 函数的函数类型有所区别，主要体现在函数的参数列表；例如有两个参数的函数和有1个参数的函数是不同的函数类型；参数列表数量相同但是类型不同的应该也是不同的函数类型

* 7.6.2过滤函数

  * 在Python中定义了一些用于数据处理的函数，如filter（）和map（）等

    * 7.6.2.1 过滤函数filter（）函数用于对容器中的元素进行过滤处理。
      * 语法：`filter(function, iterable)`
        * **function**: 一个函数，用于判断每个元素是否应该被保留。如果为 `None`，则过滤掉所有值为假的元素。
        * **iterable**: 一个可迭代对象（如列表、元组、字符串等）。
        * **返回值**: 一个 `filter`对象（迭代器），包含所有使函数返回 `True`的元素。
      * 在调用filter()函数时，iterable会被遍历，它的元素会被逐一传入function()函数中。function()函数若返回True则元素会被保留；返回False则会被过滤。最后遍历完成，已保留的元素被放到一个新的容器数据中
    
    ~~~
    def is_even(n):
        return n % 2 == 0
    
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    # 使用 filter 函数，过滤偶数
    even_numbers = filter(is_even, numbers)
    
    # 转换为列表查看结果	--> 因为filter函数的返回值是迭代器
    print(list(even_numbers))  # 输出: [2, 4, 6, 8, 10]
    ~~~
    
    * 7.6.2.2 映射函数map()
    
      * `map()`是 Python 的一个**内置高阶函数**，用于对可迭代对象中的所有元素应用指定的函数，返回一个迭代器（iterator）。
        * 语法：`map(function, iterable, ...)`
          * 参数function是一个提供变换规则的函数，返回变换之后的元素
          * **iterable**: 一个或多个可迭代对象（如列表、元组、字符串等）
          * **返回值**: 一个 `map`对象（迭代器），包含应用函数后的结果
    
    ~~~
    def square(x):
        return x ** 2
    
    numbers = [1, 2, 3, 4, 5]
    
    # 使用 map 函数计算平方
    squared_numbers = map(square, numbers)
    
    # 转换为列表查看结果
    print(list(squared_numbers))  # 输出: [1, 4, 9, 16, 25]
    ~~~
    
    * map() 函数和fliter()函数可以和其他函数搭配嵌套使用，比如处理字符串：
    
    ~~~python
    words = ["hello", "world", "python"]
    
    # 将每个单词转换为大写
    upper_words = map(str.upper, words)
    print(list(upper_words))  # 输出: ['HELLO', 'WORLD', 'PYTHON']
    
    # 获取每个单词的长度
    lengths = map(len, words)
    print(list(lengths))  # 输出: [5, 5, 6]
    ~~~
  
### 7.7 lambda()函数（~感觉应该是lambda表达式~）

* 在Python中使用lambda关键字定义匿名函数。lambda关键字定义的函数也被称为lambda（）函数，定义lambda（）函数的语法如下。“参数列表”与函数的参数列表是一样的，但不需要用小括号括起来
* Lambda 表达式（也称为**匿名函数**）是 Python 中用于创建小型、一次性使用的简单函数的语法。它允许你快速定义函数而无需使用 `def`关键字。
  * 语法：`lambda arguments: expression`
    * **lambda**: 关键字，表示这是一个 lambda 表达式
    * **arguments**: 函数的参数（可以多个，用逗号分隔）
    * **expression**: **单个表达式**，是函数的返回值（**不能包含多条语句**）
  * lambda()函数与有名称的函数一样属于函数类型

~~~python
data = [1, 2, 3, 4, 5]

filtered = filter(lambda x : (x > 3), data)
mapped = map(lambda x : (x ** 2), data)

print(list(filtered))	# [4, 5]
print(list(mapped))	# [1, 4, 9, 16, 25]
~~~

---

## 类与对象

### 9.1类的定义

* Python中的数据类型都是类，我们可以自定义类，即创建一种新的数据类型

~~~python
# Python中类的定义语法格式如下
class 类名[(父类)]:
    类体
    pass
~~~

* 在Python中任何一个类（除object外）都直接或间接地继承了父类（默认object），直接继承object时（object）部分的代码可以省略。
* 代码中的pass语句只能用于维持程序结构的完整。在编程时，若不想马上编写某些代码，又不想产生语法错误，就可以使用pass语句**占位**

### 9.2创建对象

​	类相当于一个模板，依据这样的模板来创建对象，就是类的实例化，所以对象也被称为“实例”

~~~python
class Car(object):
	# leiti
	pass
	
car = Car()
~~~

* `car = Car()`就是创建的一个小汽车对象，小括号表示调用构造方法，构造方法用于初始化对象
* 在python中，对象不再被使用时**不需要程序员手动释放对象**，由python垃圾回收器在后台释放对象，不需要手动销毁，与c++不同

### 9.3类的成员

* 在类体中也可以包含类的成员：成员变量、构造方法、成员方法、属性

  * 成员变量也被称为数据成员，保存了类或对象的数据。例如，学生的姓名和学号。
  * 构造方法是一种特殊的函数，用于初始化类的成员变量
  * 成员方法是在类中定义的函数
  * 属性是对类进行封装而提供的特殊方法。

* 辨析：实例变量和实例方法属于对象，通过对象调用；类变量和类方法属于类，通过类调用

* 9.3.1实例变量

  * 实例变量就是对象个体特有的“数据”，例如狗狗的名称和年龄等。

  ~~~python
  class Dog():
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
  d = Dog('乐乐', 3)
  print('狗子{0}, {1}岁了'.format(d.name, d.age)) # 等价但更现代 print(f'狗子{d.name}, {d.age}岁了')
  ~~~

  * 其中，`__init__方法（）`是构造方法，用于初始化实例变量，注意init前后是两个下划线

* 9.3.2构造方法

  * 类中的`__init__（）`方法是一个非常特殊的方法，用来创建和初始化实例变量，这种方法就是“构造方法”
  * 在定义构造方法时，它的第1个参数应该是self，之后的参数用来初始化实例变量。调用构造方法时不需要传入self参数
    * `def __init__(self, name, age):`是类的**构造方法**（初始化方法），在创建对象时自动调用。
    * `self`表示当前对象实例（Python 会自动传入），第一个参数必须是self
    * `name`和 `age`是构造方法的参数，**参数支持带有默认值的构造方式，能够给调用者提供多个不同版本的构造方法**
    * `self.name = name`和 `self.age = age`把传入的 `name`和 `age`赋值给对象的属性

* 9.3.3实例方法

  * 实例方法与实例变量一样，都是某个实例（或对象）个体特有的方法。
  * 定义实例方法时，它的第1个参数也应该是self，这会将当前实例与该方法绑定起来，这也说明该方法属于实例。在调用方法时不需要传入self，类似于构造方法。

  ~~~python
  class Dog():
      # 构造方法
      def __init__(self, name, age, sex = '雄性'):
          self.name = name
          self.age = age
          self.sex = sex
  
  	# 实例方法
      def run(self):
          print(f'{self.name}在跑.....')
  
       # 实例方法
      def speak(self, sound):
          	print(f'{self.name}在狗叫，{sound} !')
  
  dog = Dog('乐乐', 3)
  dog.run()
  dog.speak('看你妈！~~~~')
  
  # 效果 -->
  乐乐在跑.....
  乐乐在狗叫，看你妈！~~~~ !
  ~~~

  * 所以，窃以为构造方法就是构造函数；实例方法就是针对具体对象的类函数~大概~

* 9.3.4类变量

  * 类变量是属于类的变量，不属于单个对象
    * 例如，有一个Account（银行账户）类，它有三个成员变量：amount（账户金额）、interest_rate （利率）和owner（账户名）。
    * amount和owner对于每一个账户都是不同的，而interest_rate对于所有账户都是相同的。
    * amount和owners是实例变量，interest_rate是所有账户实例共享的变量，它属于类，被称为“类变量”。

  ~~~python
  class Account:
      interest_rate = 0.078  # 类变量，利率
  
      def __init__(self, owner, amount):
          self.owner = owner
          self.amount = amount
  
  account = Account('wjl', 3.84)
  
  print(f"账户名：{account.owner}, 账户余额：{account.amount}, 利率：{Account.interest_rate}")
  # 账户名：wjl, 账户余额：3.84, 利率：0.078	
  ~~~

  * 注意不同的变量之间的访问形式不同
    * 对类变量通过"类名.类变量"形式访问
    * 对实例变量通过"对象.实例变量"形式访问

* 9.3.5类方法

  * 类方法与类变量类似，属于类，不属于个体实例。在定义类方法时，它的第1个参数不是self，而是类本身\

  ~~~python
  # 定义类方法
  class MyClass:
      @classmethod
      def my_class_method(cls, arg1, arg2, ...):
          # 方法体
          return something
  ~~~

  * **`@classmethod`**：装饰器，表示这是一个类方法。
  * **`cls`**：第一个参数，代表类本身（类似于实例方法中的 `self`）。
  * **可以访问类属性**，但不能访问实例属性（因为没有 `self`）。

  ~~~python
  # 实例：
  class Account:
      interest_rate = 0.078  # 类变量，利率
  
      # 创建并初始化实例变量
      def __init__(self, owner, amount):
          self.owner = owner
          self.amount = amount
  
      # 类方法
      @classmethod
      def interest_by(cls, amt):
          return cls.interest_rate * amt
  
  interest = Account.interest_by(12000.0)
  print(f'计算利息：{interest:.4f}')   # 12000 * 0.078
  ~~~

### 9.4封装性

​	封装性是面向对象重要的基本特性之一。封装隐藏了对象的内部细节，只保留有限的对外接口，外部调用者不用关心对象的内部细节，使得操作对象变得简单

* 9.4.1私有变量

  * 为了防止外部调用者随意存取类的内部数据（成员变量），内部数据（成员变量）会被封装为“私有变量”。外部调用者只能通过方法调用私有变量
  * 在默认情况下，Python中的变量是公有的，可以在类的外部访问它们。如果想让它们成为私有变量，则在变量前加上双下画线`__`即可

  ~~~python
  # 在类中定义一个私有变量（__balance），实际上存储为_类名__私有变量
  class BankAccount:
      def __init__(self, balance):
          self.__balance = balance  # 实际存储为 _BankAccount__balance
  ~~~

* 9.4.2私有方法

  * 私有方法与私有变量的封装是类似，在方法前加上双下画线就是私有方法了

* 9.4.3使用属性

  * 为了实现对象的封装，在一个类中不应该有公有的成员变量，这些成员变量应该被设计为私有的，然后通过公有的`set`（赋值）和`get`（取值）方法访问。
    * 在面向对象编程（OOP）中，**get 和 set 方法**（也称为 **访问器 Accessors 和修改器 Mutators**）用于**安全地访问和修改类的私有属性**

  ~~~python
  class Person:
      def __init__(self, age):
          self.__age = age  # 私有属性
  
      def get_age(self):
          return self.__age
  
      def set_age(self, new_age):
          if 0 <= new_age <= 120:  # 年龄必须在合理范围内
              self.__age = new_age
          else:
              print("Invalid age!")
  
  p = Person(25)
  p.set_age(30)  # 正常修改
  p.set_age(150) # 触发错误提示
  ~~~

  * 在类中定义属性，属性可以代替get()和set()两个公共方法
    * 属性在本质上就是两个方法，在方法前加上装饰器使得方法成为属性。属性使用起来类似于公有变量，可以在赋值符（=）左边或右边，左边被赋值，右边取值。
    * Python 提供了 `@property`装饰器，可以**将方法伪装成属性**，使代码更简洁

  ~~~python
  class Person:
      def __init__(self, name):
          # 双下划线开头的变量是私有变量，会被Python进行名称修饰(name mangling)
          # 实际存储为 _Person__name，外部不能直接访问
          self.__name = name	
  
      @property	# 将方法转换为属性getter
      def name(self):	# 代替get_name(self)，定义name属性的get()方法，方法名就是属性名即name
          return self.__name.upper()
  
      @name.setter  # Setter
      def name(self, new_name):	# 代替set_name(self, new_name)
          if len(new_name) <= 10:  # 名字长度限制
              self.__name = new_name
          else:
              print("Name too long!")
  
  p = Person("Alice")	# 创建实例
  
  # 访问属性（自动调用@property修饰的getter方法）
  # 注意这里不是调用方法p.name()，而是像访问属性一样直接 p.name
  print(p.name)  # 输出 "ALICE"（调用 getter）// 通过属性取值，访问形式是："实例.属性"
  
  # 设置属性（自动调用@name.setter修饰的setter方法）
  p.name = "Bob"  # 调用 setter
  p.name = "VeryLongName"  # 触发错误提示
  ~~~

  * 总结：

|         类型          |   命名方式    |           访问范围            |       示例        |
| :-------------------: | :-----------: | :---------------------------: | :---------------: |
|  **公有（Public）**   |    无前缀     |           任意访问            |    `self.name`    |
| **保护（Protected）** | 单下划线 `_`  |     约定俗成，不强制限制      |    `self._age`    |
|  **私有（Private）**  | 双下划线 `__` | 仅类内部访问（Name Mangling） | `self.__password` |

### 9.5继承性

	继承性也是面向对象重要的基本特性之一。
​	在现实世界中继承关系无处不在。例如猫与动物之间的关系：猫是一种特殊动物，具有动物的全部特征和行为，即数据和操作。

​	在面向对象中动物是一般类，被称为“父类”；猫是特殊类，被称为“子类”。

​	特殊类拥有一般类的全部数据和操作，可称之为子类继承父类。**只有公共的成员变量和方法才可以被继承**

* 9.5.1python中的继承

  * 在Python中声明子类继承父类，语法很简单，定义类时在类的后面使用一对小括号指定它的父类就可以了。

  ~~~python
  # 基本继承语法
  class ParentClass:
      # 父类/基类
      pass
  
  class ChildClass(ParentClass):
      # 子类/派生类
      pass
  ~~~

  * 单继承（子类只有一个父类）

  ~~~python
  class Animal:
      def __init__(self, name):
          self.name = name
      
      def speak(self):
          print(f"{self.name} makes a sound")
  
  class Dog(Animal):  # Dog继承Animal
      def speak(self):  # 方法重写
          print(f"{self.name} barks")
  
  dog = Dog("Buddy")
  dog.speak()  # 输出: Buddy barks
  ~~~

* 9.5.2多继承

  * 在Python中，当子类继承多个父类时，如果在多个父类中有相同的成员方法或成员变量，则子类优先继承左边父类中的成员方法或成员变量，从左到右继承级别从高到低。
  * 如果子类的方法名与父类的方法名相同，则在这种情况下，子类的方法会重写（Override）父类的同名方法。

  ~~~python
  class Horse:
      def __init__(self, name):
          self.name = name    # 实例变量name
  
      def show_info(self):
          return f"马的名字是{self.name}"
  
  class Donkey:
      def __init__(self, name):
          self.name = name
  
      def show_info(self):
          return f"驴的名字是{self.name}"
  
      def run(self):
          print("驴在跑...")
  
      def roll(self):
          print("驴打滚...")
  
  class Nule(Horse, Donkey):  # 骡子类，多继承自Horse和Donkey（注意继承顺序）
      def __init__(self, name, age = 18):
          super().__init__(name)  # 调用第一个父类(Horse)的__init__
          self.age = age  # 实例变量age，默认18岁
  
      def show_info(self):    # 重写父类方法（覆盖Horse和Donkey的show_info）
          return f"骡子：{self.name}, {self.age}"
  
  # 创建骡子实例
  m = Mule('小乐', 21)
  
  # 方法调用说明：
  m.run()     # 输出"马在跑..." 
              # 解释：由于Mule继承顺序是(Horse, Donkey)，所以优先使用Horse的run()方法
  
  m.roll()    # 输出"驴打滚..." 
              # 解释：从第二个父类Donkey继承的方法
  
  print(m.show_info())    # 输出"骡子：小乐, 21岁" 
                          # 解释：调用子类重写后的方法，不是父类的方法
  ~~~

  * 可以使用 `print(Mule.__mro__)`查看方法解析顺序

    * Python会按照 `Mule.__mro__`的顺序查找方法，在多继承中，`super()`遵循MRO顺序

    * 当前顺序是：Mule -> Horse -> Donkey -> object

### 9.6多态性

~~~
多态性也是面向对象重要的基本特性之一。“多态”指对象可以表现出多种形态。
~~~

​	例如，猫、狗、鸭子都属于动物，它们有“叫”和“动”等行为，但是叫的方式不同，动的方式也不同。

* 9.6.1继承和多态

  * 在多个子类继承父类，并重写父类方法后，这些子类所创建的对象之间就是多态的。这些对象采用不同的方式实现父类方法。

* 9.6.2鸭子类型测试与多态(Duck Typing)

  * Python的多态性更加灵活，支持鸭子类型测试。鸭子类型测试指：若看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟可以被称为鸭子
  * 由于支持鸭子类型测试，所以Python解释器**不检查发生多态的对象是否继承了同一个父类**，只要它们有相同的行为（方法），它们之间就是多态的。

  ~~~python
  class Duck:
      def quack(self):
          print("Quack, quack!")
      
      def fly(self):
          print("The duck is flying")
  
  class Person:
      def quack(self):
          print("I'm quacking like a duck!")	 # 人类模仿鸭子叫
      
      def fly(self):
          print("I'm flapping my arms!")	 # 人类模仿鸭子飞
  
  def duck_test(thing):
      thing.quack()
      thing.fly()
  
  # 测试
  duck = Duck()
  person = Person()
  
  duck_test(duck)    # 输出正常的鸭子行为
  duck_test(person)  # 输出人类模仿鸭子的行为
  ~~~

  * 鸭子类型的关键特点
    1. **不检查类型**
       * `duck_test()`**不检查** `thing`是否是 `Duck`类。
       * 只要 `thing`有 `quack()`和 `fly()`方法，就能调用。
    2. 动态绑定
       * Python 在运行时检查对象是否有对应方法，而不是编译时。
    3. 多态性
       * `Duck`和 `Person`是完全不同的类，但都能通过 `duck_test()`测试。

---

## 异常处理

### 10.1第一个异常——除零异常

* `Traceback (most recent call last)`：Traceback 信息是“异常堆栈信息”，描述了程序运行的过程及引发异常的信息
* 除零异常：`ZeroDivisionError: division by zero`
* 在python中，异常类命名的主要后缀有Exception、Error、Warning，也有少数几个没有采用这几个后缀命名，统一将其称为异常

### 10.2捕获异常

* 什么是异常捕获
  * 异常捕获是Python中处理程序运行时错误的一种机制。当程序执行过程中出现错误（如除以零、访问不存在的索引等）时，Python会"抛出"一个异常。如果不处理这些异常，程序会终止运行。异常捕获允许我们优雅地处理这些错误情况，而不是让程序崩溃。

* try_except语句

  ~~~py
  try:
      # 可能会引发异常的代码块
      risky_operation()
  except ExceptionType:
      # 当指定异常发生时执行的代码
      handle_error()
  ~~~

* 在except语句中还可以指定具体的异常类型。如果不指定具体的异常数据类型，则except语句能够捕获try中发生的所有异常；如果指定了具体的异常类型，则except语句只能捕获try中发生的指定类型的异常

### 10.3常见异常类型

1. `ZeroDivisionError`: 除数为零
2. `IndexError`: 索引超出序列范围
3. `KeyError`: 字典中不存在的键
4. `TypeError`: 操作或函数应用于不适当类型的对象
5. `ValueError`: 函数接收到正确类型但不合适的值
6. `FileNotFoundError`: 尝试打开不存在的文件
7. `IOError`: 输入/输出操作失败
8. 自定义异常

~~~python
class MyCustomError(Exception):
    pass

try:
    raise MyCustomError("这是我的自定义错误")
except MyCustomError as e:
    print(f"捕获到自定义错误: {e}")
~~~

### 10.4多个异常的处理

* 多条语句可能会引发多种不同的异常，对每一种异常都会采用不同的处理方式。针对这种情况，我们可以在一个try后面跟多个except代码块
* 省略异常类型的except代码块是默认的except代码块，**只能**被放到最后，捕获上面没有匹配到的异常类
* try_except语句支持嵌套写法（不建议）

~~~python
try:
    # 尝试执行的代码
    result = 10 / 0
except ZeroDivisionError:
    # 处理特定的异常（除零错误）
    print("不能除以零！")
except (TypeError, ValueError) as e:	# 这是多重异常捕获的语法形式
    # 处理多种异常，并获取异常对象
    print(f"类型或值错误: {e}")
except Exception as e:	# 这里的e是异常对象，输出内容是对应异常的英文提示
    # 处理所有其他异常
    print(f"发生了未知错误: {e}")
else:
    # 如果没有异常发生，执行此代码块
    print("操作成功完成！")
finally:
    # 无论是否发生异常，都会执行的代码块
    print("清理工作...")
~~~

### 10.5使用finally代码块释放资源

* 有时在try-except语句中会占用一些资源，例如打开的文件、网络连接、打开的数据库及数据结果集等都会占用计算机资源，需要程序员释放这些资源。为了确保这些资源能够被释放，可以使用finally代码块
* 无论是try代码块正常结束还是except代码块异常结束，都会执行finally代码块

### 10.6自定义异常类

* 实现自定义异常类，需要继承Exception类或其子类，之前的ZeroDivisionError和ValueError异常都属于Exception的子类

~~~python
class wjl_Exception(Exception):	# wjl_Exception这是自定义异常类的名称
    def __init__(self, message):	# 构造方法，message是异常描述信息
        super().__init__(message)	# 调用父类构造方法，并把参数message传递给父类构造方法
~~~

---

## 常用的内置模块

### 第十一章，马上就要看完了~加油！


傻逼好像更新不能关掉cmd，是吗

??~~


---

## 运算符

* 算数运算符

~~~
a / b : 求a除以b的商(含小数), 10 / 3 = 3.3333, 4 / 2 = 2.0
a % b : 求a除以b的余数, 10 % 3 = 1
a // b : 求小于a和b商的最大整数(就是求商，不含小数), 10 // 3 = 3
a ** b : 求a的b次幂
~~~

* 比较运算符
  1. 浮点数可以和整数比较：1.0 == 1 --> True；
  2. <u>比较运算符</u>可以用于任意类型的数据，但是参与比较的两种类型的数据必须要**相互兼容**，即那个进行隐式转换。例如，整数、浮点和布尔三种类型是相互兼容的~
  3. <u>比较字符串的大小</u>，即逐一比较字符的Unicode编码大小，如果两个字符串的第一个字符不能比较出大小，就顺序比较第二个，直到比较出结果
  4. <u>比较列表的大小</u>，跟字符串相同，注意：在需要比较的两个列表中的元素类型必须兼容
* 逻辑运算符，用于对bool变量进行运算，结果也是bool类型，not、and、or
* 位运算符，以二进制为单位进行运算，操作数和结果都是整数类型的数据，注意：`计算机内部存储的都是补码，而且一般是32位`,所以178的8位二进制是0b10110010，但是全32位二进制是0x000000B2,所以符号是0，正数~
  * tips : `~x = -x - 1`， eg : ~178 = -179

```
~x: 将x的值按位取反
x & y: 与运算
x | y: 或运算
x ^ y: 异或
x >> a: 将x向右移a位，高位采用符号位补位
x << a: 将x向左移a位，低位补0
```

* 赋值运算符
* 运算符的优先级

```
优先级从高到低：
1. ()
2. **	#幂
3. ~	#位反
4. +, - #正负号
5. *, /, %, //
6. +, - #加减
7. <<, >>
8. &
9. ^
10. |
11. <, <=, >, >=, !=, == #比较
12. not
13. and, or
```

​	运算符的大体顺序：算数运算符->位运算符->关系运算符->逻辑运算符->赋值运算符

---

## 程序流程控制

* 分支语句(没有switch)

```
if
if-else
if-elif-else

score = int(input())
if score >= 90:
	grade = 'A'
elif score >= 80:
	grade = 'B'
else:
	grade = 'C'
```

* 循环语句(没有do.while)

  * while

  ```
  while 循环条件：
  	循环体语句组
  [else:
  	语句组]	#else这部分可以省略，else子语句是在循环体正常结束时才执行的语句，当循环被中断时不执行，当遇到break、return和有异常发生时都会中断循环
  	
  x = 1
  while x < 100:
      x += 1
      if x == 50:
          break	#产生循环中断
  else:
      print('hello') #不执行
  print(x)   #------------->输出50
  ```

  * for语句

  ```
  for 变量 in 可迭代对象:
  	循环体语句组
  [else:
  	语句组]	#可迭代对象包括：字符串、列表、元组、集合、字典等
  	
  for i in 'hello,world!':
  	print(i)
  ```

* 跳转语句，能够改变程序的执行顺序，包括break，continue，return

### 小练一手，求水仙花

```
i = 1
while i < 1000:
    cnt = 0
    idx = i
    while idx > 0:
        cnt += (idx % 10) ** 3	# 这里水仙花的定义是立方和不是平方和
        idx //= 10
    if i == cnt:
        print(i)
    i += 1
# 有一个严重的小问题：~就是外层循环i，不能改变其值，会导致外层循环失效~注意一下
# 1通常不被认为是水仙花数，1000内的水仙花数有153、370、371、407
```



---

## 基本语句

1. `print(value,..., sep='', end = '\n', file = None)`

```
a = 10
b = 20
print(10)
print(a) # 10
print(a + b) # 30

print('牛逼')
print("不牛逼")
print('''三个单引号''')
print("""三个双引号"")
print(a,b,'nb') # a b nb --->注意有空格

函数chr() ---> ascall编码转字符
ord() --->ascall字符转编码
print(chr(56)) # [

print('wjl' + 'nb') # 字符串连接
```

* print写入文件操作

```
fq = open('note.txt','w')
print('wjl牛逼',file = fq)
fq.close()
```

* 参数说明

```
#print(value,..., sep='', end = '\n', file = None)

sep=''，在输出参数之间有空格; end,结束符，默认输出换行
```

2. 输入函数input：`x = input()` --->输入的数据无论是什么，x的数据类型都是字符串类型

```
x = input('这里的任何内容都不会作为输出，只是提示')
print(x)
```

再接再厉

---

## 学习python的一些疑惑

~~~
给我讲一下python中的类的继承和调用方法、成员函数和具体调用父类的方法函数的实现顺序，其中参数的传递是什么样的，子类调用父类的函数或方法，子类的参数会不会对父产生影响
~~~

### 1. 方法调用顺序（方法解析顺序 - MRO）

* Python使用C3算法来确定方法调用顺序，可以通过`类名.__mro__`查看。

~~~python
class A:
    def method(self):
        print("A的方法")

class B(A):
    def method(self):
        print("B的方法")

class C(A):
    def method(self):
        print("C的方法")

class D(B, C):
    pass

print(D.__mro__)  # 输出方法解析顺序
d = D()
d.method()  # 输出"B的方法"
~~~

* `D.__mro__`的输出方法顺序：(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
  *  `D`的MRO会按深度优先、从左到右的顺序遍历继承树，并在遇到重复类时保留最后一个（C3算法）。
* **最后一个元素 `<class 'object'>`**：
  * 这是Python所有类的终极基类
  * 它出现在MRO末尾是因为所有自定义类默认继承自 `object`（Python 3中隐式继承）
  * 这保证了任何方法查找最终都会回溯到 `object`类（例如 `__str__`等基础方法）

### 2. 调用父类方法的几种方式

1. 直接使用父类名调用（不推荐）

~~~python
class Child(Parent):
    def method(self):
        Parent.method(self)  # 显式调用父类方法
        print("子类方法")
~~~

2. 使用super()函数（推荐）

~~~python
class Child(Parent):
    def method(self):
        super().method()  # Python3简洁写法
        print("子类方法")
~~~

* `super()`不是简单地调用父类，而是按照MRO顺序调用下一个类
  * `super()`的行为并不是直接指向当前类的父类，而是按照 **MRO 链** 找到**下一个要调用的类**。
    * 在多重继承中，`super()`的调用顺序由 `__mro__`决定，而不是简单地按照继承关系。
    * 举例：B、C继承A，D继承B、C
      * `__mro__ `决定了(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)的顺序，当执行到`B_super()`时，由于MRO 中 C 在 A 之前，所以super()在B中指向C而不是A

### 3. 参数传递机制

1. 子类实例调用父类方法时，会将self自动传递给父类方法
2. 父类方法不能直接访问子类的特有属性，除非子类显式传递

~~~py
class Parent:
    def __init__(self, name):
        self.name = name	# 姓名：张三
    
    def show(self, prefix):
        print(f"{prefix}: {self.name}")

class Child(Parent):
    def __init__(self, name, age):
        super().__init__(name)  # 调用父类__init__，传递name参数
        self.age = age
    
    def show(self, prefix):
        super().show(prefix)  # 调用父类show方法，传递prefix参数
        print(f"年龄: {self.age}")	# 年龄：25

c = Child("张三", 25)
c.show("姓名")  # 输出姓名和年龄
~~~

3.参数传递对父类的影响

*  **不会直接影响父类实例**：子类调用父类方法时，参数传递不会影响其他父类实例
* **影响当前实例的继承属性**：通过super()调用的父类方法会修改当前实例的属性

~~~py
class Parent:
    def __init__(self):
        self.value = 0	# 在python里不需要事先声明变量~就是方便呐
    
    def increment(self, amount=1):
        self.value += amount

class Child(Parent):
    def increment(self, amount=1):
        super().increment(amount * 2)  # 修改传递给父类的参数
        print(f"值现在是: {self.value}")

p = Parent()
c = Child()

p.increment(5)  # p.value = 5
c.increment(5)  # c.value = 10 (因为传递了5 * 2=10) -> 改变当前实例的数据
print(p.value)  # 仍然是5，不受子类影响
print(c.value)  # 10
~~~

