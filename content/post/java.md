---
title: 'java基础语法'
date: 2025-11-06
lastmod: 2025-11-06
categories: ['java学习路线']
tags: ['编程语言']
cover: https://kidle9527.github.io/images/33.png
---

* 下载JDK，`www.oracle.com`，长期维护的版本：JDK8、11、17、21、25
* 下载IDEA，`Intellij IDEA`，ai插件`lingma`，免费滴！
* java里`src`代表源代码的意思

> 黑马配套视频笔记：https://heuqqdmbyk.feishu.cn/drive/folder/KLQsfGToJl3fZhdagWdc0jh5nye

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

## 🌞`Scanner`（输入） ✅

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

## 🔥`print`(输出）

在 Java 中，常用的输出手段主要用于将数据打印到控制台或其他输出流，不同的输出方式在功能、适用场景和使用方式上存在差异。以下是 Java 中主要的输出手段及其区别：

### 1. `System.out.println()`

- **功能**：打印指定内容后**换行**。

- **特点**：

  - 输出内容会自动在末尾添加换行符（`\n`，具体换行符与系统相关）。
  - 可以接收任意类型的参数（基本类型、对象等），会自动调用对象的`toString()`方法。

- **适用场景**：需要换行的普通输出，如打印一行信息、调试日志等。

- **示例**：

  ```java
  System.out.println("Hello, World!"); // 输出后自动换行
  System.out.println(123); // 输出数字123并换行
  ```

### 2. `System.out.print()`

- **功能**：打印指定内容，不自动换行。

- **特点**：

  - 输出内容后不会添加换行符，后续输出会紧跟在当前内容之后。
  - 同样支持任意类型参数，自动调用`toString()`。

- **适用场景**：需要连续输出且不换行的场景，如拼接多个内容到同一行。

- **示例**：

  ```java
  System.out.print("Hello, ");
  System.out.print("Java!"); // 输出结果：Hello, Java!（无换行）
  ```

### 3. `System.out.printf()`

- **功能**：格式化输出，类似 C 语言的`printf`，支持格式控制符。

- **特点**：

  - 需指定格式字符串（含格式控制符，如`%d`、`%s`、`%f`等）和参数列表，按格式输出。
  - 不自动换行，需手动添加`\n`或使用`%n`（跨平台换行符）。

- **适用场景**：需要精确控制输出格式的场景，如数字保留小数位、对齐方式等。

- **示例**：

  ```java
  int age = 20;
  System.out.printf("年龄：%d岁%n", age); // 输出：年龄：20岁（%n换行）
  double pi = 3.14159;
  System.out.printf("π ≈ %.2f", pi); // 输出：π ≈ 3.14（保留2位小数）
  ```

### 4. `System.err` 相关方法（`println()`/`print()`/`printf()`）

- **功能**：与`System.out`的方法功能相同，但用于输出错误信息。

- **特点**：

  - 属于标准错误流（`stderr`），而`System.out`是标准输出流（`stdout`）。
  - 在控制台中，错误信息通常以红色显示（依赖终端配置），且输出顺序可能与`System.out`不同（因两者是独立流，缓冲机制不同）。

- **适用场景**：输出程序运行时的错误信息、异常提示等，便于区分正常输出和错误。

- **示例**：

  ```java
  System.err.println("发生错误：文件不存在"); // 错误信息，通常红色显示
  ```

### 5. `BufferedWriter`

- **功能**：字符缓冲输出流，通过缓冲提高写入效率，支持换行等操作。

- **特点**：

  - 属于 IO 流体系，需手动创建对象，关联输出目标（如控制台、文件）。
  - 需调用`flush()`或`close()`方法才能将缓冲数据真正输出（`close()`会自动调用`flush()`）。
  - 提供`newLine()`方法，跨平台输出换行符（推荐替代`\n`）。

- **适用场景**：大量字符输出（如文件写入），需高效缓冲或跨平台换行。

- **示例**：

  ```java
  import java.io.BufferedWriter;
  import java.io.OutputStreamWriter;
  
  public class Test {
      public static void main(String[] args) throws Exception {
          BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(System.out));
          writer.write("Hello");
          writer.newLine(); // 跨平台换行
          writer.write("BufferedWriter");
          writer.close(); // 关闭时自动刷新
      }
  }
  ```

### 6. `PrintWriter`

- **功能**：字符打印流，提供类似`System.out`的`print()`/`println()`/`printf()`方法，支持自动刷新。

- **特点**：

  - 可关联文件、网络流等，不仅限于控制台。
  - 构造时可指定`autoFlush`参数（`true`时，调用`println()`等方法会自动刷新）。
  - 支持格式化输出，功能更灵活。

- **适用场景**：需要向文件、网络等非控制台目标输出，或需要自动刷新的场景。

- **示例**：

  ```java
  import java.io.PrintWriter;
  
  public class Test {
      public static void main(String[] args) throws Exception {
          // 向控制台输出，自动刷新
          PrintWriter out = new PrintWriter(System.out, true);
          out.println("PrintWriter"); // 自动刷新
          
          // 向文件输出
          PrintWriter fileOut = new PrintWriter("output.txt");
          fileOut.printf("数字：%d", 100);
          fileOut.close();
      }
  }
  ```

### 核心区别总结

| 输出方式               | 核心特点                             | 适用场景                         | 输出目标               |
| ---------------------- | ------------------------------------ | -------------------------------- | ---------------------- |
| `System.out.println()` | 自动换行，简单直接                   | 控制台普通输出，需换行           | 控制台（stdout）       |
| `System.out.print()`   | 不换行，连续输出                     | 控制台拼接输出                   | 控制台（stdout）       |
| `System.out.printf()`  | 格式化输出，需手动换行               | 控制台格式化内容（如数字、对齐） | 控制台（stdout）       |
| `System.err` 相关      | 错误信息，通常红色显示，独立流       | 输出错误 / 异常信息              | 控制台（stderr）       |
| `BufferedWriter`       | 缓冲高效，跨平台换行，需手动刷新     | 大量字符输出（如文件）           | 文件、控制台等         |
| `PrintWriter`          | 支持多种输出目标，自动刷新，功能全面 | 文件、网络等非控制台输出         | 文件、网络流、控制台等 |

选择输出方式时，需根据是否需要格式化、是否换行、输出目标（控制台 / 文件等）以及效率需求综合判断。日常控制台简单输出常用`System.out`的方法，文件或高效输出则倾向于`BufferedWriter`或`PrintWriter`。

### 转义字符 💦

| 转义字符 | 含义                      | 书写格式 | 说明                                                         |
| -------- | ------------------------- | -------- | ------------------------------------------------------------ |
| `\n`     | 换行符（Line Feed）       | `"\n"`   | 使光标移到下一行的开头（不同系统换行行为可能有差异，Java 会自动适配）。 |
| `\t`     | 制表符（Horizontal Tab）  | `"\t"`   | 类似键盘 `Tab` 键，使光标跳到下一个制表位（通常是 8 个字符位置），用于对齐。 |
| `\r`     | 回车符（Carriage Return） | `"\r"`   | 使光标移到当前行的开头（不换行），单独使用较少，常与 `\n` 组合（如 `\r\n` 是 Windows 系统的换行标准）。 |
| `\"`     | 双引号（Double Quote）    | `"\""`   | 在字符串中表示双引号（避免与字符串的起止引号冲突）。         |
| `\'`     | 单引号（Single Quote）    | `'\''`   | 在字符常量中表示单引号（避免与字符的起止引号冲突）。         |
| `\\`     | 反斜杠（Backslash）       | `"\\"`   | 表示一个普通的反斜杠（因 `\` 本身是转义符，需用 `\\` 转义）。 |
| `\b`     | 退格符（Backspace）       | `"\b"`   | 使光标回退一格（删除前一个字符的效果）。                     |
| `\f`     | 换页符（Form Feed）       | `"\f"`   | 用于打印机换页，屏幕输出中较少使用。                         |

## 🐸 运算符

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

| 操作符 | 类型          | 短路特性                | 适用场景                     |                              |                            |
| ------ | ------------- | ----------------------- | ---------------------------- | ---------------------------- | -------------------------- |
| `&`    | 逻辑与 / 位与 | 无（总是执行两边）      | 位运算或需要执行两边表达式时 |                              |                            |
| `&&`   | 短路与        | 有（左为 false 则短路） | 仅需要逻辑判断时（更高效）   |                              |                            |
| `|`    | 逻辑或/位或   | 逻辑或 / 位或           | 无（总是执行两边）           | 位运算或需要执行两边表达式时 |                            |
| `||`   | 短路或        | 左边为true直接返回      | 短路或                       | 有（左为 true 则短路）       | 仅需要逻辑判断时（更高效） |

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

> 注意：字符 `'*'` 是字符类型，参与运算时会被转为对应的 ASCII 码（数字），触发**算术运算**
>
> 比如： `1 + '*' + 2` --> `*`的ascall是42，结果是45；`1 + "*" + 2`--> `1*2`

### 运算符优先级

* 从高到低大致顺序：
  * 括号 `()` → 单目运算符（`!`、`~`、`++`、`--`）→ 算术运算符 → 移位运算符 → 比较运算符 → 逻辑运算符 → 条件运算符 → 赋值运算符

* 实际开发中，建议使用括号明确指定运算顺序，提高代码可读性。

## 🐮 类型转换

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

---

## 🐱 流程控制

Java 的流程控制用于控制程序执行的顺序，主要包括以下几类结构：

### 1. 顺序结构

这是 Java 程序的默认执行方式，代码从上到下依次执行，没有任何跳转或分支。

```java
System.out.println("第一步");
System.out.println("第二步");
// 先输出"第一步"，再输出"第二步"
```

### 2. 分支结构

根据条件判断执行不同的代码块：

#### if-else 语句

```java
int score = 85;
if (score >= 90) {
    System.out.println("优秀");
} else if (score >= 60) {
    System.out.println("及格");
} else {
    System.out.println("不及格");
}
```

#### switch 语句

适用于多值判断，JDK7 + 支持 String 类型

* default的位置和省略情况：default可以放在任意位置，也可以省略

- case穿透：不写break会引发case穿透现象（🌛其实就是多个case对应一个情况）这并不是一个绝对的坏现象

```java
int day = 3;
switch (day) {
    case 1:
        System.out.println("星期一");
        break;
    case 2:
        System.out.println("星期二");
        break
   	case 8:case 9:
    		System.oout.println("嘉豪日")；
    default:
        System.out.println("其他星期");
}
```

#### switch区别于其他语言的新特性🌟

Java 从 JDK 7 开始对 `switch` 语句进行了多次增强，尤其是 JDK 12 引入的 **switch 表达式**（Switch Expressions）带来了诸多新特性，使代码更简洁、功能更强大。以下是这些新特性的详细说明：

##### 1. 箭头标签（`->`）：简化 case 分支

​	传统 `switch` 使用 `:` 分隔 case 条件和代码块，且需要显式使用 `break` 避免穿透。

**新特性**：可使用 `->` 替代 `:`，箭头右侧的代码块会自动终止（无需 `break`），更简洁。

示例：

```java
int day = 3;
switch (day) {
    case 1 -> System.out.println("周一");
    case 2 -> System.out.println("周二");
    case 3 -> System.out.println("周三"); // 输出：周三
    default -> System.out.println("其他");
}
```

- 箭头右侧可以是单个表达式、代码块（用 `{}` 包裹），或直接返回值（用于 switch 表达式）。
- 自动避免穿透：执行完当前 case 后直接退出 switch，无需手动写 `break`。

##### 2. 多值 case：一个 case 匹配多个值

传统 `switch` 中一个 case 只能写一个值，多值需要多个 case 堆叠。

**新特性**：允许在一个 `case` 后用逗号分隔多个值，表示 “匹配其中任意一个”。

示例：

```java
int month = 2;
switch (month) {
    case 1, 3, 5, 7, 8, 10, 12 -> System.out.println("31天");
    case 4, 6, 9, 11 -> System.out.println("30天");
    case 2 -> System.out.println("28或29天"); // 输出：28或29天
    default -> System.out.println("无效月份");
}
```

- 等价于传统写法中多个 `case` 共用同一逻辑（如 `case 1: case 3: ...`），但代码更紧凑。

##### 3. switch 表达式：有返回值的 switch

传统 `switch` 是**语句**（statement），仅执行逻辑，无返回值。

**新特性**：`switch` 可作为**表达式**（expression），有返回值，可直接赋值给变量或作为方法返回值。

示例：

```java
int day = 2;
String weekday = switch (day) {
    case 1 -> "周一";
    case 2 -> "周二"; // 匹配此分支，返回"周二"
    case 3 -> "周三";
    default -> "未知";
};
System.out.println(weekday); // 输出：周二
```

- 表达式要求**所有可能的分支都必须有返回值**（包括 `default`），否则编译报错。
- 箭头右侧的表达式结果会作为整个 switch 表达式的返回值。

##### 4. `yield` 关键字：在代码块中返回值🌛

当 switch 表达式的分支需要复杂逻辑（多行代码）时，无法直接用箭头右侧的单个表达式返回值。

**新特性**：`yield` 关键字用于在代码块中指定 switch 表达式的返回值。

示例：

```java
int num = 3;
String result = switch (num) {
    case 1 -> "一";
    case 2 -> {
        System.out.println("处理数字2");
        yield "二"; // 返回"二"
    }
    case 3 -> {
        System.out.println("处理数字3");
        yield "三"; // 返回"三"
    }
    default -> "其他";
};
System.out.println(result); // 输出：三
```

- `yield` 仅用于 switch 表达式中，作用类似 `return`，但只退出当前 switch 分支，不影响外部方法。
- 若分支是代码块（用 `{}` 包裹），必须通过 `yield` 显式返回值，否则编译报错。

##### 5. 增强的 null 处理

传统 `switch` 中，若传入 `null` 会直接抛出 `NullPointerException`。

**新特性**：可在 `case` 中显式处理 `null` 值（JDK 17+ 优化）。

示例：

```java
String str = null;
switch (str) {
    case null -> System.out.println("值为null"); // 输出：值为null
    case "a" -> System.out.println("a");
    default -> System.out.println("其他");
}
```

- 无需手动判断 `str == null`，直接在 `case null` 中处理，更简洁。

### 总结：新特性对比传统 switch

| 特性           | 传统 switch                    | 新 switch 表达式               |
| -------------- | ------------------------------ | ------------------------------ |
| 形式           | 语句（无返回值）               | 可作为表达式（有返回值）       |
| 分支分隔符     | `:`（需手动 `break` 避免穿透） | `->`（自动终止，无需 `break`） |
| 多值匹配       | 多个 `case` 堆叠               | 单个 `case` 用逗号分隔多值     |
| 复杂逻辑返回值 | 无法直接返回                   | 用 `yield` 在代码块中返回      |
| null 处理      | 直接抛异常                     | 支持 `case null` 显式处理      |

这些特性使 `switch` 更灵活，尤其适合替代复杂的 `if-else` 链，代码可读性和简洁性大幅提升。实际开发中，推荐在 JDK 17+ 环境下使用这些新特性（需注意项目的 JDK 版本兼容性）。

---

### 3. 循环结构

重复执行某段代码，直到满足终止条件：

#### for 循环

适合已知循环次数的场景

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i); // 输出0到4
}
```

#### while 循环

适合未知循环次数，先判断后执行

```java
int count = 0;
while (count < 3) {
    System.out.println(count);
    count++;
}
```

#### do-while 循环

先执行后判断，**至少执行一次**

```java
int num = 5;
do {
    System.out.println(num);
    num--;
} while (num > 0);
```

### 4. 跳转语句

用于改变程序的执行流程：

- **break**：终止当前循环或 switch 语句

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break; // 当i=5时跳出循环
    System.out.println(i);
}
```

- **continue**：跳过本次循环剩余部分，进入下一次循环

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue; // 跳过偶数
    System.out.println(i);
}
```

- **return**：结束当前方法的执行并返回结果

```java
public int add(int a, int b) {
    return a + b; // 返回到方法调用处
}
```

这些流程控制结构可以相互嵌套，形成复杂的程序逻辑，帮助开发者实现各种业务需求。

---

## 🍇`Random`类

Random跟Scanner一样，也是Java提前写好的类，我们不需要关心是如何实现的，只要直接使用就可以了。

* 导入Random包：`import java.util.Random;`
* 使用

```java
// 创建对象
Random r = new Random();

// 获取随机数
int number = r.nextInt(n); // [0, n)
int number = r.nextInt(); // 随机生成int类型范围
int number = r.nextInt(1, 3); // [1, 3) 
```

> 随机数范围的特点：左关右开，默认左0

在 Java 中，`Math.random()` 和 `java.util.Random` 都是用于生成随机数的工具，但它们的设计和使用方式存在差异，这也导致了是否需要导入包的区别：

#### 1. 什么是 `Math.random()`？

`Math.random()` 是 Java 核心类 `java.lang.Math` 中的一个静态方法，用于生成一个 **[0.0, 1.0) 区间的随机双精度浮点数（double）**。

其特点是：

- 调用简单，直接通过类名调用（`Math.random()`），无需创建对象。
- 底层依赖于一个全局的随机数生成器（类似 `Random` 实例），但封装了细节，适合简单的随机数需求。

#### 2. 为什么 `Math.random()` 不需要导入包？

因为 `Math` 类位于 `java.lang` 包下。

Java 规定：**`java.lang` 包中的类是默认自动导入的**，无需显式使用 `import` 语句声明。

因此，`Math` 类及其静态方法 `random()` 可以直接使用，无需额外导入。

#### 3. 为什么 `Random r = new Random();` 需要导入包？

`Random` 类位于 `java.util` 包下（全类名为 `java.util.Random`）。

`java.util` 包不属于默认自动导入的范围，**所有非 `java.lang` 包中的类，使用时都需要显式导入**（除非使用全类名）。

如果不导入，直接写 `Random r = new Random();` 会报错，因为编译器无法识别 `Random` 类的位置。必须通过以下方式之一解决：

- 显式导入：在代码开头添加 `import java.util.Random;`。
- 使用全类名：`java.util.Random r = new java.util.Random();`。

总结

- `Math.random()` 属于 `java.lang.Math`，因 `java.lang` 包自动导入，所以无需额外操作。
- `Random` 属于 `java.util` 包，非默认导入，因此需要显式导入或使用全类名。

两者的本质区别源于所在包的不同，以及 Java 对 `java.lang` 包的特殊处理（自动导入）。

---

## 💎数组

### 数组语法详解

​	Java、C++、C 中的数组在核心功能上一致（存储同类型元素的连续内存），但语法、内存管理和特性存在显著差异。以下从异同点和用法示例两方面说明：

| 特性             | C 数组                                                       | C++ 数组                                               | Java 数组                                                    |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| **声明语法**     | 类型 数组名 [长度];如：`int arr[5];`                         | 同 C，还支持 `int[5] arr;`（较少用）                   | 类型 [ ] 数组名；如：`int[] arr;`                            |
| **初始化方式**   | 静态初始化：`int arr[] = {1,2,3};`动态初始化：无（需手动赋值） | 同 C，还支持 `int arr[] {1,2,3};`（C++11 起，省略`=`） | **静态：`int[] arr = {1,2,3};`动态：`int[] arr = new int[3];`** |
| **长度获取**     | 需手动计算：`sizeof(arr)/sizeof(arr[0])`                     | 同 C，或用 `std::size(arr)`（C++17 起）                | 直接通过 `length` 属性：`arr.length`                         |
| **内存管理**     | 栈上数组自动释放；堆上需 `free()`（用 `malloc` 分配）        | 栈上自动释放；堆上需 `delete[]`（用 `new` 分配）       | 由 JVM 自动管理（堆上分配，垃圾回收）                        |
| **边界检查**     | 无，越界访问导致未定义行为                                   | 无，越界访问导致未定义行为                             | 运行时检查，越界抛 `ArrayIndexOutOfBoundsException`          |
| **数组作为参数** | 传递的是指针（丢失长度信息）                                 | 同 C，或用引用传递保留长度：`void f(int (&arr)[3])`    | 传递的是引用（保留 `length` 属性）                           |
| **多维数组**     | 本质是数组的数组（行优先存储）                               | 同 C，还支持动态多维数组（如 `int**arr`）              | 本质是数组的数组（行数固定，列数可动态）                     |

### 代码解释

```java
public class JavaArrayExample {
    public static void main(String[] args) {
        // 1. 静态初始化
        int[] staticArray = {1, 2, 3, 4, 5}; //完整写法 int[] staticArray = new int[]{1, 2, 3, 4, 5};
        
        // 2. 动态初始化
        int[] dynamicArray = new int[5];
        
        // 3. 赋值
        for (int i = 0; i < dynamicArray.length; i++) {
            dynamicArray[i] = i * 2;
        }
        
        // 4. 访问元素
        System.out.println("静态数组第三个元素: " + staticArray[2]);
        
        // 5. 遍历数组
        System.out.println("动态数组元素:");
        for (int num : dynamicArray) {  // 增强for循环
            System.out.print(num + " ");	// 0 2 4 6 8 
        }
        
        // 6. 数组越界会抛出ArrayIndexOutOfBoundsException
        // System.out.println(staticArray[10]); // 运行时异常
    }
}
```

### 注意事项

* 数组名：其实就是名字而已，方便以后使用，在起名字的时候遵循**小驼峰命名法**。例：arr   namesArr
* 索引一定是从0开始的。
* 动态初始化手动指定数组长度，由系统给出默认初始化值。～array --> 数组的英文～

---

## 🌹方法

在 Java 中，**方法（Method）** 是一段封装了特定功能的可重用代码块，类似于其他语言中的 “函数”。它的主要作用是提高代码的复用性、可读性和可维护性。

### 一、方法的基本结构

Java 方法的定义格式如下：

```java
修饰符 返回值类型 方法名(参数列表) {
    // 方法体（功能实现代码）
    return 返回值; // 若返回值类型为void，则无此句
}
```

各部分说明：

1. **修饰符**：控制方法的访问权限或特性（如`public`、`private`、`static`、`final`等）。
2. **返回值类型**：方法执行后返回结果的数据类型。若无需返回值，用`void`表示。
3. **方法名**：遵循标识符规则（驼峰命名法，如`calculateSum`）。
4. **参数列表**：方法接收的输入数据，格式为`数据类型 参数名`，多个参数用逗号分隔（可无参数）。
5. **方法体**：实现具体功能的代码块。
6. **return 语句**：将结果返回给调用者（若返回值类型为`void`，可省略或仅写`return;`）。

### 二、方法的分类

根据是否有参数和返回值，可分为 4 类：

1. **无参无返回值**

   ```java
   public void printHello() {
       System.out.println("Hello");
   }
   ```

2. **无参有返回值**

   ```java
   public int getRandomNumber() {
       return (int)(Math.random() * 100); // 返回0-99的随机数
   }
   ```

3. **有参无返回值**

   ```java
   public void printMessage(String message) {
       System.out.println(message);
   }
   ```

4. **有参有返回值**

   ```java
   public int add(int a, int b) {
       return a + b;
   }
   ```

### 三、方法的调用

方法定义后需被调用才能执行，调用方式取决于方法是否为`static`（静态）：

1. **静态方法（static 修饰）**：直接通过`类名.方法名()`调用。

   ```java
   public class MathUtil {
       public static int multiply(int a, int b) {
           return a * b;
       }
   }
   // 调用
   int result = MathUtil.multiply(3, 4); // 结果为12
   ```

   

2. **非静态方法**：需先创建类的对象，再通过`对象.方法名()`调用。

   ```java
   public class Greeting {
       public void sayHi(String name) {
           System.out.println("Hi, " + name);
       }
   }
   // 调用
   Greeting greeting = new Greeting();
   greeting.sayHi("Java"); // 输出：Hi, Java
   ```

### 四、方法的特性

1. **重载（Overload）**

   同一类中，允许存在多个同名方法，但参数列表（类型、个数、顺序）必须不同。与返回值类型无关。

   * 重载的条件：
     * 多个方法在一个类中且具有相同的方法名
     * 多个方法的参数不同 ----> 参数类型不同或者参数数量不同🔥

   > ⚠️也就是说，返回类型无关重载！！！

   ```java
   public class Calculator {
       public int add(int a, int b) { return a + b; }
       public double add(double a, double b) { return a + b; } // 参数类型不同
       public int add(int a, int b, int c) { return a + b + c; } // 参数个数不同
   }
   ```

2. **递归（Recursion）**

   方法自身调用自身，用于解决可分解为同类子问题的场景（如阶乘、斐波那契数列）。

   示例（计算阶乘）：

   ```java
   public int factorial(int n) {
       if (n == 1) return 1; // 终止条件
       return n * factorial(n - 1); // 递归调用
   }
   ```

### 五、注意事项

- 方法不能嵌套定义（不能在一个方法内部定义另一个方法）。
- 方法的参数是 “值传递”：基本类型传递副本，引用类型传递地址副本（修改引用指向的对象内容会影响原对象）。
- `main`方法是程序入口，格式固定为：`public static void main(String[] args)`。

* 方法是 Java 编程的核心组成部分，合理使用方法能大幅提升代码质量。

---

## java原理

#### 一、Java 的运行机制

Java 是一种**跨平台的编程语言**，其核心运行机制依赖于 “一次编写，到处运行”（Write Once, Run Anywhere, WORA），关键在于 **JVM（Java 虚拟机）** 的存在。

1. **编译阶段**：

   开发者编写的 `.java` 源文件通过 `javac` 编译器编译为 **字节码文件（.class）**。字节码是一种与平台无关的二进制指令，不直接对应特定操作系统的机器码。

2. **运行阶段**：

   不同平台（如 Windows、Linux、macOS）安装对应的 JVM，JVM 负责将 `.class` 字节码**解释执行**（或通过 JIT 即时编译器编译为本地机器码加速执行）。

   - 解释执行：JVM 逐条将字节码翻译成机器码并执行（启动快，执行慢）。
   - JIT 编译：热点代码（频繁执行的代码）被编译为机器码缓存，后续直接执行（启动稍慢，执行快）。

**总结**：Java 程序通过 “源文件 → 字节码 → JVM 解释 / 编译执行” 的流程，实现跨平台性，而 JVM 是连接字节码与底层系统的桥梁。

#### 二、内存与内存地址

内存是程序运行时数据的存储区域，Java 的内存由 JVM 统一管理（避免直接操作物理内存，更安全）。

1. **内存的本质**：

   物理内存是一组连续的存储单元，每个单元有唯一的**物理地址**（如 0x00000001），用于标识数据存储位置。但 Java 中开发者无法直接访问物理地址，而是通过 JVM 抽象的**逻辑内存模型**操作数据。

2. **Java 中的 “内存地址”**：

   Java 中没有直接暴露物理内存地址，而是通过**引用（Reference）** 间接操作对象。引用可以理解为 “指向对象在堆内存中位置的标识”，类似 “逻辑地址”。例如：

   ```java
   Object obj = new Object(); // obj 是引用，指向堆中 new Object() 创建的对象
   ```

   这里的 `obj` 存储的不是物理地址，而是 JVM 维护的对象在堆中的逻辑位置标识（具体实现依赖 JVM，如指针或句柄）。

#### 三、Java 内存分配（JVM 内存模型）

JVM 运行时将内存划分为多个区域，各自有明确的功能和生命周期，主要包括以下部分（基于 JDK 8+，元空间替代永久代）：

1. **程序计数器（Program Counter Register）**：
   - 作用：记录当前线程执行的字节码指令地址（行号），线程私有（每个线程独立拥有）。
   - 特点：唯一不会 OOM（内存溢出）的区域。
2. **虚拟机栈（VM Stack）**：
   - 作用：存储**方法调用的栈帧**（包括局部变量表、操作数栈、方法出口等），线程私有。
   - 细节：
     - 局部变量（如基本类型 `int`、`char`，以及对象引用）存储在局部变量表中。
     - 方法调用时创建栈帧入栈，方法执行完毕栈帧出栈。
   - 异常：栈深度超过 JVM 限制时抛出 `StackOverflowError`；栈扩展失败时抛出 `OutOfMemoryError`。
3. **本地方法栈（Native Method Stack）**：
   - 作用：类似虚拟机栈，但为 Native 方法（如 C 语言实现的方法）服务，线程私有。
4. **堆（Heap）**：
   - 作用：存储**对象实例和数组**，是 JVM 中最大的内存区域，线程共享。
   - 特点：
     - 垃圾回收（GC）的主要区域，几乎所有对象的内存都在这里分配。
     - 可通过 `-Xms`（初始堆大小）和 `-Xmx`（最大堆大小）参数调整。
   - 异常：对象创建时堆内存不足，抛出 `OutOfMemoryError`。
5. **方法区（Method Area）**：
   - 作用：存储已被 JVM 加载的类信息（类名、字段、方法、常量池等）、静态变量、即时编译器编译后的代码等，线程共享。
   - JDK 8 及以上：方法区的实现为**元空间（Metaspace）**，直接使用本地内存（不再受 JVM 内存限制，而受系统内存限制）。
   - 异常：类信息加载过多时，元空间不足会抛出 `OutOfMemoryError`。

#### 四、Java 内存分配的基本规则

1. **基本类型 vs 引用类型**：
   - 基本类型（`int`、`double` 等）：变量直接存储值，局部变量存于虚拟机栈的局部变量表中；静态基本类型变量存于方法区。
   - 引用类型（对象、数组）：变量存储引用（指向堆中的对象），对象本身存于堆中；静态引用变量的引用存于方法区，指向堆中的对象。
2. **对象的分配流程**：
   - 优先在堆的**新生代（Eden 区）** 分配内存（大多数对象朝生夕灭，新生代 GC 频率高）。
   - 大对象可能直接进入**老年代**（避免频繁复制大对象）。
   - 长期存活的对象（经历多次 GC 仍存活）会进入老年代。

> 变量存在栈里，变量指向对象，对象存在堆里，堆指向类，类在方法里，方法被调到栈中执行❌不完全对，看后面

* 这句话的描述存在部分不准确，需要结合 Java 的内存模型修正。以下是详细解释，并结合汽车例子说明：

  * 一、修正后的内存模型
    1. **类（Class）**：类的信息（属性定义、方法字节码等）存储在**方法区**（JDK 8 后为元空间），而非方法里。类在程序加载时被 ClassLoader 加载到方法区，是全局唯一的模板。
    2. **对象（Object）**：通过`new`创建的对象实例存储在**堆内存**中，包含对象的属性值。堆中的对象会指向方法区的类信息（通过类指针），以便知道自己属于哪个类。
    3. **变量（引用变量）**：基本类型变量（如`int`）直接存储在**栈内存**；引用类型变量（如对象引用）也存储在栈中，但存储的是**堆中对象的地址**（即 “指向对象”）。
    4. **方法执行**：方法被调用时，会在**栈内存**中创建一个 “栈帧”（包含局部变量、方法参数等），方法执行完毕后栈帧被销毁。方法本身的字节码在方法区（属于类信息的一部分）。

  * 二、汽车例子的内存分析

    以`Car`类和对象为例，代码如下：

```java
// 类的定义（加载后存放在方法区）
class Car {
    String color;  // 属性（对象的属性值在堆中）
    String brand;

    void run() {  // 方法（字节码在方法区，属于Car类信息）
        System.out.println(brand + "在行驶");
    }
}

public class Main {
    public static void main(String[] args) {  // main方法栈帧在栈中
        Car myCar = new Car();  // 引用变量myCar在栈中，指向堆中的Car对象
        myCar.color = "红色";   // 堆中对象的属性被赋值
        myCar.brand = "特斯拉";
        myCar.run();  // 调用run()，创建run方法的栈帧，执行后销毁
    }
}
```

* 内存分布如下：

  - **方法区**：存放`Car`类的信息（`color`、`brand`属性定义，`run()`方法的字节码）和`Main`类信息。

  - **堆内存**：存放`new Car()`创建的对象，包含具体属性值（`color="红色"`，`brand="特斯拉"`），并通过类指针指向方法区的`Car`类。

  - **栈内存**：
    - `main`方法的栈帧：包含局部变量`myCar`（引用类型），存储堆中`Car`对象的地址（即`myCar`指向堆对象）。
    - 调用`run()`时，创建`run`方法的栈帧，执行完毕后栈帧被弹出栈。

* 三、原句错误点修正
  1. **“类在方法里”**：错误。类是独立的模板，方法是类的一部分，类信息存放在方法区，与方法的执行栈帧无关。
  2. **“堆指向类”**：不准确。堆中的对象通过**类指针**指向方法区的类信息，而非堆整体指向类。
  3. **“方法被调到栈中执行”**：部分正确。方法执行时会在栈中创建栈帧，但方法本身的字节码在方法区（属于类信息），并非方法整体 “移动” 到栈中。

* 总结，正确的逻辑是：

**引用变量在栈中，指向堆中的对象；堆中的对象通过类指针关联方法区中的类信息；方法执行时在栈中创建栈帧，方法的字节码存放在方法区的类信息中**。

---

## 🌟辩证：对象、类、方法

在 Java 面向对象编程中，**对象**、**类**和**方法**是三个最基础且核心的概念，它们之间相互关联，共同构成了面向对象编程的骨架。我们可以通过现实世界的类比来理解它们：

### 1. 类（Class）：事物的 “模板” 或 “设计图”

类是对现实世界中同一类事物的**抽象描述**，它定义了这类事物共同的**属性**（特征）和**方法**（行为），但不具体指向某个个体。

可以理解为：类是一张 “设计图”，比如 “汽车设计图”，它规定了汽车有轮子、发动机（属性），能行驶、刹车（方法），但图纸本身不是真实的汽车。

**示例**：

定义一个 `Car` 类，描述 “汽车” 这类事物：

```java
// 类的定义：抽象描述“汽车”
class Car {
    // 属性（特征）：所有汽车都有的共性特征
    String color;  // 颜色
    int wheels;    // 轮子数量
    String brand;  // 品牌

    // 方法（行为）：所有汽车都能执行的操作
    void run() {
        System.out.println(brand + "汽车在行驶");
    }

    void brake() {
        System.out.println(brand + "汽车刹车了");
    }
}
```

### 2. 对象（Object）：类的 “实例” 或 “具体个体”

对象是类的**具体实例**，是根据类这个 “模板” 创建出来的真实存在的个体。它拥有类中定义的所有属性和方法，且属性有具体的值。

比如，根据 “汽车设计图”（类）生产出的 “一辆红色特斯拉汽车”（对象），它有具体的颜色（红色）、品牌（特斯拉），能实际行驶和刹车。

**示例**：

基于 `Car` 类创建具体的对象：

```java
public class Main {
    public static void main(String[] args) {
        // 创建对象：用 new 关键字，根据 Car 类生成具体实例
        Car tesla = new Car();
        
        // 给对象的属性赋值（具体特征）
        tesla.color = "红色";
        tesla.wheels = 4;
        tesla.brand = "特斯拉";
        
        // 调用对象的方法（执行具体行为）
        tesla.run();   // 输出：特斯拉汽车在行驶
        tesla.brake(); // 输出：特斯拉汽车刹车了
    }
}
```

**总结**：类是 “抽象概念”，对象是 “具体实例”；一个类可以创建多个不同的对象（比如再创建一辆 “蓝色宝马”）。

### 3. 方法（Method）：对象的 “行为” 或 “功能”

方法是定义在类中的**一段可执行代码块**，用于描述对象的行为或能完成的功能。它可以操作对象的属性，也可以实现特定逻辑。

比如 “汽车” 的 “行驶”“刹车” 就是方法，“人” 的 “吃饭”“睡觉” 也是方法。

**方法的特点**：

- 必须定义在类中，属于类的一部分；
- 可以接收参数（输入数据），也可以返回结果（输出数据）；
- 通过 “对象。方法名 ()” 的方式调用，让对象执行具体操作。

**示例**：带参数和返回值的方法

```java
class Calculator {
    // 方法：接收两个整数，返回它们的和（功能：计算加法）
    int add(int a, int b) {
        return a + b; // 返回结果
    }
}

// 使用方法
public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator(); // 创建对象
        int result = calc.add(3, 5); // 调用方法，传入参数 3 和 5
        System.out.println(result); // 输出：8（方法返回的结果）
    }
}
```

### 三者的关系

- **类**包含**属性**和**方法**，是对象的 “模板”；
- **对象**是类的实例，拥有类中定义的属性（有具体值）和方法（可执行）；
- **方法**是对象的行为，通过对象调用，实现具体功能。

简单说：**类是 “是什么”**（定义特征和行为），**对象是 “具体是谁”**（根据类创建的个体），**方法是 “能做什么”**（对象的行为）。

---
