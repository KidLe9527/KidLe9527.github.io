---
title: 'java学习'
date: 2025-11-06
lastmod: 2025-11-06
categories: ['java学习路线']
tags: ['编程语言']
cover: https://kidle9527.github.io/images/33.png
---

* 下载JDK，`www.oracle.com`，长期维护的版本：JDK8、11、17、21、25
* 下载IDEA，`Intellij IDEA`，ai插件`lingma`，免费滴！
* java里`src`代表源代码的意思

## 注释

* 单行注释： // 注释信息
* 多行注释： /* 注释信息 */
* 文档注释： /** 注释信息 */

## 关键字

* `package`：软件包，表示当前的类定义在那个包下面
* java程序的主入口，类似cpp的main函数，打印helloworld

```java
public static void main(String[] args) {
        System.out.println("hello, world!");
    }
```

## 字面量

* **定义**：字面量是直接表示数据值的符号，是程序中硬编码的具体值。
* **作用**：为变量赋值或直接参与运算，是数据的 “字面表现形式”。
* **分类**：根据数据类型不同，字面量可分为：
  - 整数字面量：`100`、`-5`、`0x1A`（十六进制）
  - 浮点数字面量：`3.14`、`2.5e3`（科学计数法）
  - 字符字面量：`'A'`、`'\n'`（转义字符）
  - 字符串字面量：`"Hello"`
  - 布尔字面量：`true`、`false`
  - 空字面量：`null`（仅用于引用类型）
* 整数、小数、字符串、字符、布尔、空类型
  * 整数和小数自接写
  * 字符串双引号，字符单引号
  * 布尔：true、false
  * 空类型：null

## 变量

* 程序中，用来存储单个数据的容器 <----经常变化的数据
* java中一条语句可以定义多个变量，也可哟连续赋值
* 在计算机中，任意数据都是以二进制的形式存储的
* 字节是计算机的最小存储单元，1 type = 8 bit；
* java中，int类型占据4个字节

## 数据类型

- **定义**：数据类型是对数据的分类，规定了数据的取值范围、存储方式以及可进行的操作。
- **作用**：限制变量能存储的数据种类，决定了内存分配的大小和方式。
- **分类**：
  - 基本数据类型（8 种）：`byte`、`short`、`int`、`long`、`float`、`double`、`char`、`boolean`。
  - 引用数据类型：类（`class`）、接口（`interface`）、数组（`[]`）、枚举（`enum`）等。
- 细节：
  - long类型数据必须以L结尾，大小写不论；比如：`long  tem = 10000000L`，L不能省略
  - float类型的数据必须以f结尾，而double不需要，因为java默认小数的数据类型是double
  - 布尔类型的数据类型是`boolean`

## 标识符

* 就是自己起的变量名
* 标识符的命名规则
  * 由数字，下划线`_`，字母，美元符`$`组成
  * 不能以数字开头，不能是关键字
  * 区分大小写

| 对比维度                | 小驼峰命名法（lowerCamelCase）                               | 大驼峰命名法（UpperCamelCase/PascalCase）                    |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **首字母规则**          | 首字母小写，后续每个单词的首字母大写                         | 首字母大写，后续每个单词的首字母大写                         |
| **核心特点**            | 标识符整体以小写开头，“驼峰” 从第二个单词开始显现            | 标识符整体以大写开头，“驼峰” 从第一个单词即显现              |
| **常见使用场景**        | 变量、函数、方法、参数                                       | 类、接口、构造函数、枚举、结构体                             |
| **示例 1（变量）**      | `userName`（用户姓名）、`orderTotalPrice`（订单总价）        | 不推荐（类名才用大驼峰，如`User`）                           |
| **示例 2（函数）**      | `getUserInfo()`（获取用户信息）、`calculateTax()`（计算税费） | 不推荐（类名才用大驼峰，如`UserService`）                    |
| **示例 3（类 / 接口）** | 不推荐（变量才用小驼峰，如`user`）                           | `User`（用户类）、`OrderService`（订单服务类）、`Payable`（可支付接口） |

## `Scanner`

* 在 Java 中，`Scanner`是`java.util`包中的一个工具类，主要用于从各种输入源（如键盘、文件、字符串等）读取数据。它提供了便捷的方法来解析基本数据类型（如`int`、`double`）和字符串，是处理输入的常用工具。

### 一、基本使用步骤

1. **导入 Scanner 类**：由于`Scanner`位于`java.util`包中，需在代码开头导入：

   ```java
   import java.util.Scanner;
   ```

2. **创建 Scanner 对象**：指定输入源（最常用的是键盘输入`System.in`）：

   ```java
   Scanner scanner = new Scanner(System.in); // 从键盘读取输入
   ```

3. **读取数据**：使用`Scanner`的各种`nextXxx()`方法读取不同类型的数据。

4. **关闭 Scanner**：使用完毕后调用`close()`方法释放资源：

   ```java
   scanner.close();
   ```

### 二、常用读取方法

`Scanner`提供了针对不同数据类型的读取方法，以下是最常用的几种：

| 方法            | 功能描述                                   | 示例                                   |
| --------------- | ------------------------------------------ | -------------------------------------- |
| `next()`        | 读取一个字符串（以空格 / 回车为分隔符）    | `String str = scanner.next();`         |
| `nextLine()`    | 读取一整行字符串（以回车为分隔符）         | `String line = scanner.nextLine();`    |
| `nextInt()`     | 读取一个整数                               | `int num = scanner.nextInt();`         |
| `nextDouble()`  | 读取一个双精度浮点数                       | `double d = scanner.nextDouble();`     |
| `nextBoolean()` | 读取一个布尔值（`true`或`false`）          | `boolean b = scanner.nextBoolean();`   |
| `hasNextXxx()`  | 判断是否还有对应类型的数据（用于循环读取） | `while (scanner.hasNextInt()) { ... }` |

### 三、使用示例

#### 示例 1：读取键盘输入的基本类型

```java
import java.util.Scanner;

public class ScannerDemo {
    public static void main(String[] args) {
        // 创建Scanner对象（关联键盘输入）
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("请输入姓名：");
        String name = scanner.next(); // 读取字符串（遇到空格停止）
        
        System.out.print("请输入年龄：");
        int age = scanner.nextInt(); // 读取整数
        
        System.out.print("请输入身高（米）：");
        double height = scanner.nextDouble(); // 读取小数
        
        // 输出结果
        System.out.println("姓名：" + name);
        System.out.println("年龄：" + age);
        System.out.println("身高：" + height + "米");
        
        // 关闭资源
        scanner.close();
    }
}
```

#### 示例 2：使用`nextLine()`读取整行内容

```java
import java.util.Scanner;

public class ScannerLineDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("请输入一句话：");
        String sentence = scanner.nextLine(); // 读取一整行（包括空格）
        System.out.println("你输入的是：" + sentence);
        
        scanner.close();
    }
}
```

#### 示例 3：循环读取多个整数（用`hasNextInt()`判断）

```java
import java.util.Scanner;

public class ScannerLoopDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int sum = 0;
        
        System.out.println("请输入多个整数（输入非整数结束）：");
        // 循环读取，直到没有整数输入
        while (scanner.hasNextInt()) {
            int num = scanner.nextInt();
            sum += num;
            System.out.println("已输入：" + num + "，当前总和：" + sum);
        }
        
        System.out.println("最终总和：" + sum);
        scanner.close();
    }
}
```

### 四、注意事项

1. **`next()`与`nextLine()`的区别**：

   - `next()`：只读取到空格或回车前的内容，**不会吸收回车符**。
   - `nextLine()`：读取一整行（包括空格），**会吸收回车符**。
   - 若在`nextInt()`/`nextDouble()`后直接使用`nextLine()`，可能会读取到空字符串（因为前一个方法未吸收回车符）。解决方法：在中间加一个`scanner.nextLine()`吸收回车。

   ```java
   // 错误示例（nextInt()后直接用nextLine()）
   System.out.print("输入年龄：");
   int age = scanner.nextInt();
   System.out.print("输入地址：");
   String address = scanner.nextLine(); // 会读取到空字符串
   
   // 正确示例
   System.out.print("输入年龄：");
   int age = scanner.nextInt();
   scanner.nextLine(); // 吸收遗留的回车符
   System.out.print("输入地址：");
   String address = scanner.nextLine(); // 正常读取
   ```

2. **输入不匹配异常**：若输入的数据类型与`nextXxx()`方法不匹配（如用`nextInt()`读取字符串），会抛出`InputMismatchException`。建议结合`hasNextXxx()`先判断再读取。

3. **关闭资源**：使用完毕后务必调用`close()`，避免资源泄露。关闭后无法再使用该`Scanner`对象读取数据。

4. **其他输入源**：`Scanner`不仅能读取键盘输入，还能读取文件、字符串等：

   ```java
   // 读取字符串
   Scanner strScanner = new Scanner("hello world 123");
   // 读取文件（需处理IOException）
   Scanner fileScanner = new Scanner(new File("test.txt"));
   ```

### 总结

* `Scanner`是 Java 中处理输入的便捷工具，通过`nextXxx()`方法可轻松读取各种数据类型。使用时需注意`next()`与`nextLine()`的区别，以及输入类型匹配问题，合理搭配`hasNextXxx()`能提高代码的健壮性。

## 运算法

> 小数直接参与运算，可能导致运算结果不准确

在 Java 中，运算符是用于执行各种操作的特殊符号，根据功能可以分为以下几类：

### 1. 算术运算符

用于基本的数学运算：

- `+`：加法（也可用于字符串拼接）
- `-`：减法
- `*`：乘法
- `/`：除法（整数相除会舍去小数部分）
- `%`：取余（模运算）
- `++`：自增（前缀先增后用，后缀先用后增）
- `--`：自减（规则同自增）

示例：

```java
int a = 10, b = 3;
System.out.println(a + b);  // 13
System.out.println(a / b);  // 3（整数除法）
System.out.println(a % b);  // 1
System.out.println(a++);    // 10（先输出后自增）
System.out.println(++a);    // 12（先自增后输出）
```

### 2. 赋值运算符

用于给变量赋值：

- `=`：基本赋值
- 复合赋值：`+=`、`-=`、`*=`、`/=`、`%=`等

示例：

```java
int x = 5;
x += 3;  // 等价于 x = x + 3 → x=8
x *= 2;  // 等价于 x = x * 2 → x=16
```

### 3. 比较运算符

用于比较两个值，返回布尔值（`true`/`false`）：

- `==`：等于
- `!=`：不等于
- `>`：大于
- `<`：小于
- `>=`：大于等于
- `<=`：小于等于

示例：

```java
int a = 5, b = 10;
System.out.println(a == b);  // false
System.out.println(a < b);   // true
```

### 4. 逻辑运算符

用于逻辑运算，操作数和结果都是布尔值：

- `&&`：短路与（两边都为`true`才返回`true`，左边为`false`则不执行右边）
- `||`：短路或（一边为`true`就返回`true`，左边为`true`则不执行右边）
- `!`：非（取反）

示例：

```java
boolean p = true, q = false;
System.out.println(p && q);  // false
System.out.println(p || q);  // true
System.out.println(!p);      // false
```

### 5. 位运算符

直接对二进制位进行操作：

- `&`：按位与
- `|`：按位或
- `^`：按位异或（相同为 0，不同为 1）
- `~`：按位取反
- `<<`：左移（乘以 2 的 n 次方）
- `>>`：右移（除以 2 的 n 次方，保留符号位）
- `>>>`：无符号右移（高位补 0）

示例：

```java
int a = 6;  // 二进制：0110
int b = 3;  // 二进制：0011
System.out.println(a & b);  // 2（0010）
System.out.println(a << 1); // 12（1100，相当于6×2）
```

### 6. 条件运算符（三目运算符）

格式：`条件表达式 ? 表达式1 : 表达式2`
如果条件为`true`，返回表达式 1 的值，否则返回表达式 2 的值。

示例：

```java
int score = 85;
String result = score >= 60 ? "及格" : "不及格";
System.out.println(result);  // 及格
```

### 7. 其他运算符

- `instanceof`：判断对象是否为某个类的实例

  ```java
  String str = "hello";
  System.out.println(str instanceof String);  // true
  ```

- 括号`()`：改变运算优先级或用于方法调用

- 点`.`：访问对象的属性或方法

### 8. 字符串拼接

```java 
1 + 2 ---> 3
"hello" + ", world" --> "hello, world"
1 + 2 + "hello" + 3 + 4 ---> "3hello34" // 注意1 + 2 还是整数相加，后面就是字符串拼接了！
```



### 运算符优先级

* 从高到低大致顺序：
  * 括号 `()` → 单目运算符（`!`、`~`、`++`、`--`）→ 算术运算符 → 移位运算符 → 比较运算符 → 逻辑运算符 → 条件运算符 → 赋值运算符

* 实际开发中，建议使用括号明确指定运算顺序，提高代码可读性。

## 类型转换

​	在 Java 中，类型转换是将一种数据类型的值转换为另一种数据类型的过程，主要分为两种类型：自动类型转换（隐式转换）和强制类型转换（显式转换）。

### 1. 自动类型转换（隐式转换）

当两种数据类型兼容，且目标类型的取值范围大于源类型时，Java 会自动进行类型转换，无需手动干预。

> 注意⚠️！在java中，含有byte、short的数据类型在进行计算时会自动转换成int类型
>
> `byte sum = (short)a + (byte)b;` ---> 由于a和b默认转换成int类型，所以这句代码实际上会报错
>
> 解决方法：1.修改byte sum 为 int sum；2. 强制类型转换 (byte)(a + b)，注意可能会丢失精度兄弟！

**规则**：

- 从小范围类型向大范围类型转换（不会丢失数据）
- 整数类型可以自动转换为浮点类型
- `char`类型可以自动转换为`int`类型（通过 ASCII 值）

**示例**：

```java
// 整数类型之间的转换
byte b = 100;
short s = b;  // byte -> short
int i = s;    // short -> int
long l = i;   // int -> long

// 整数转浮点
float f = l;  // long -> float
double d = f; // float -> double

// char转int
char c = 'A';
int charCode = c;  // 结果为65（'A'的ASCII值）
```

转换顺序（从左到右可自动转换）：
`byte → short → int → long → float → double`
`char → int`

### 2. 强制类型转换（显式转换）

当需要将大范围类型转换为小范围类型，或两种类型不兼容时，必须使用强制类型转换。语法为：`目标类型 变量名 = (目标类型) 源变量/值;`

**注意**：

- 可能导致数据丢失或精度下降
- 必须显式使用括号指定目标类型

**示例**：

```java
// 大范围转小范围（可能丢失数据）
int largeNum = 300;
byte smallNum = (byte) largeNum;  // 结果为44（因为byte范围是-128~127）

// 浮点转整数（丢失小数部分）
double pi = 3.14159;
int intPi = (int) pi;  // 结果为3

// 不兼容类型转换
String str = "123";
// int num = (int) str;  // 编译错误，String与int不兼容
```

### 3. 引用类型转换

对于对象类型，转换规则有所不同：

- 向上转型：子类对象可以自动转换为父类类型（`子类对象 → 父类类型`）
- 向下转型：父类对象转换为子类类型时，必须进行强制转换，且要确保对象实际类型与目标类型兼容，否则会抛出`ClassCastException`

**示例**：

```java
class Animal {}
class Dog extends Animal {}

// 向上转型（自动转换）
Animal animal = new Dog();  // 正确

// 向下转型（需要强制转换）
Dog dog = (Dog) animal;    // 正确，因为animal实际是Dog类型

// 错误示例
Animal anotherAnimal = new Animal();
// Dog wrongDog = (Dog) anotherAnimal;  // 运行时抛出ClassCastException
```

使用`instanceof`运算符可以在转换前检查类型兼容性，避免异常：

```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;  // 安全转换
}
```

### 总结

- 自动转换：小范围→大范围，无需显式操作
- 强制转换：大范围→小范围，需显式声明，可能丢失数据
- 引用类型转换需遵循继承关系，向下转型前建议使用`instanceof`检查
