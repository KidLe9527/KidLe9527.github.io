---
title: 'java面向对象语法'
date: 2025-11-18
lastmod: 2025-11-18
categories: ['java学习路线']
tags: ['编程语言']
cover: https://kidle9527.github.io/images/66.png
---

## 🤫面向对象编程

在 Java 中，对象是类的实例，是面向对象编程（OOP）的核心概念。以下从对象的相关语法层面，系统梳理其核心内容：

### 一、对象的创建（实例化）

对象通过类实例化产生，语法为：

```java
类名 对象名 = new 构造方法(参数列表);
```

- **构造方法**：与类名同名，无返回值，用于初始化对象。**若类中未定义构造方法，编译器会自动生成默认无参构造方法；若自定义了构造方法，默认构造方法会失效。**✅ 没有返回值哦，也叫构造器！

  ```java
  class Person {
      String name;
      // 构造方法
      public Person(String n) { 
          name = n; 
      }
  }
  // 创建对象
  Person p = new Person("Alice"); 
  ```

### 二、对象的成员访问

通过 **`.` 运算符**访问对象的成员变量和成员方法：

```java
// 访问成员变量
对象名.成员变量名;
// 调用成员方法
对象名.成员方法名(参数列表);
```

示例：

```java
class Person {
    String name; // 成员变量
    public void sayHi() { // 成员方法
        System.out.println("Hi, " + name);
    }
}
Person p = new Person();
p.name = "Bob"; // 赋值成员变量
p.sayHi(); // 调用方法，输出 "Hi, Bob"
```

### 三、对象的引用与 null

- Java 中对象变量存储的是**引用**（内存地址），而非对象本身。

- `null` 表示引用不指向任何对象，访问 `null` 引用的成员会抛出 `NullPointerException`。

  ```java
  Person p = null;
  p.sayHi(); // 运行时异常：NullPointerException
  ```

### 四、对象的比较

1. **`==` 运算符**：比较两个引用是否指向**同一个对象**（内存地址是否相同）。

2. **`equals()` 方法**：默认与 `==` 功能一致，可通过重写自定义比较逻辑（如比较内容）。

   ```java
   String s1 = new String("abc");
   String s2 = new String("abc");
   System.out.println(s1 == s2); // false（不同对象）
   System.out.println(s1.equals(s2)); // true（内容相同，String 重写了 equals）
   ```

### 五、对象的销毁（垃圾回收）

- Java 无需手动销毁对象，由**垃圾回收器（GC）** 自动回收不再被引用的对象所占用的内存。
- 可通过 `System.gc()` 建议 JVM 执行垃圾回收，但具体时机由 JVM 决定。
- 重写 `finalize()` 方法可在对象被回收前执行一些清理操作（不推荐依赖，因执行时机不确定）。

### 六、对象的转型

1. **向上转型**（自动转换）：子类对象赋值给父类引用，体现多态。

   ```java
   class Animal {}
   class Dog extends Animal {}
   Animal a = new Dog(); // 向上转型
   ```

2. **向下转型**（强制转换）：父类引用转换为子类引用，需先通过 `instanceof` 检查，否则可能抛出 `ClassCastException`。

   ```java
   if (a instanceof Dog) {
       Dog d = (Dog) a; // 向下转型
   }
   ```

### 七、对象的克隆（复制）

通过 `Cloneable` 接口和 `clone()` 方法实现对象复制：

1. 类实现 `Cloneable` 接口（标记接口，无方法）。

2. 重写 `Object` 类的 `clone()` 方法（默认是浅拷贝）。

   ```java
   class Student implements Cloneable {
       String name;
       @Override
       protected Object clone() throws CloneNotSupportedException {
           return super.clone();
       }
   }
   Student s1 = new Student();
   Student s2 = (Student) s1.clone(); // 克隆对象
   ```

### 八、对象的序列化

将对象转换为字节流以便存储或传输，需实现 `Serializable` 接口（标记接口）：

```java
import java.io.*;
class User implements Serializable {
    String username;
}
// 序列化
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.dat"));
oos.writeObject(new User());
// 反序列化
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.dat"));
User u = (User) ois.readObject();
```

### 总结

Java 对象的语法围绕其**创建、访问、生命周期、类型转换、复制、传输**等核心操作展开，结合类的定义（成员变量、方法、构造器等），共同构成了面向对象编程的基础。理解这些语法是掌握 Java OOP 思想（封装、继承、多态）的关键。

---

## 🌛类的基本语法

### 1. 构造器 --> 就是cpp的构造函数

在 Java 中，构造器（Constructor）是一种特殊的方法，用于创建对象时初始化对象的成员变量。它的名称必须与类名完全一致，且没有返回值（连 `void` 都不能写）。

##### 构造器的基本语法

```java
修饰符 类名(参数列表) {
    // 初始化代码（通常是为成员变量赋值）
}
```

- **修饰符**：可以是访问权限修饰符（`public`、`protected`、`private`），也可以省略（默认权限）。一般不使用 `static`、`final` 等修饰符（语法允许但无意义）。
- **类名**：必须与所在类的名称完全相同（包括大小写）。
- **参数列表**：与普通方法类似，可包含 0 个或多个参数（用于接收初始化对象时传入的值）。
- **方法体**：用于初始化对象，常见操作是为成员变量赋值，也可以调用其他方法（但不能调用自身构造器以外的构造器，除非用 `this()`）。

##### 构造器的分类

1. **无参构造器（默认构造器）**

   没有参数的构造器。如果类中没有显式定义任何构造器，Java 会自动生成一个默认的无参构造器（权限为默认）。

   ```java
   public class Person {
       private String name;
       private int age;
       
       // 显式定义无参构造器
       public Person() {
           name = "默认姓名";
           age = 0;
       }
   }
   ```

   注意：如果显式定义了任何构造器，Java 将不再自动生成默认无参构造器。

2. **有参构造器**

   带有参数的构造器，用于在创建对象时直接为成员变量赋值（避免创建后再调用 `setter` 方法）。

   ```java
   public class Person {
       private String name;
       private int age;
       
       // 有参构造器：初始化 name 和 age
       public Person(String n, int a) {
           name = n; // 为成员变量赋值
           age = a;
       }
   }
   ```

3. **构造器重载**

   一个类中可以定义多个构造器，只要它们的参数列表（参数个数、类型、顺序）不同，这称为**构造器重载**（类似方法重载）。

   ```java
   public class Person {
       private String name;
       private int age;
       
       // 无参构造器
       public Person() {
           this("默认姓名", 0); // 调用有参构造器（this() 必须在第一行）
       }
       
       // 单参数构造器（只初始化 name）
       public Person(String n) {
           this(n, 0); // 调用双参数构造器
       }
       
       // 双参数构造器
       public Person(String n, int a) {
           name = n;
           age = a;
       }
   }
   ```

##### 构造器的使用场景

- 创建对象时必须通过构造器初始化（`new 类名(参数)` 本质就是调用构造器）。
- 确保对象创建后处于合理的初始状态（例如避免成员变量为 `null` 或无效值）。
- 通过 `this(参数)` 复用其他构造器的代码（减少冗余）。

##### 注意事项

- 构造器不能有返回值（包括 `void`），否则会被视为普通方法。
- `this(参数)` 只能在构造器中使用，且必须是构造器的**第一行代码**（用于调用同类其他构造器）。
- 构造器不能被 `static`、`final`、`abstract` 修饰（构造器属于对象实例，而 `static` 属于类；`final` 和 `abstract` 对构造器无意义）。
- 子类构造器默认会调用父类的无参构造器（通过 `super()`），如果父类没有无参构造器，子类必须显式调用父类的有参构造器（否则编译报错）。

示例：使用构造器创建对象

```java
public class Main {
    public static void main(String[] args) {
        // 调用无参构造器
        Person p1 = new Person();
        // 调用单参数构造器
        Person p2 = new Person("张三");
        // 调用双参数构造器
        Person p3 = new Person("李四", 20);
    }
}
```

总之，构造器是 Java 中初始化对象的核心机制，通过合理定义构造器，可以简化对象创建过程并保证对象的初始状态合法性。

### 2. this关键字

在 Java 中，`this` 是一个关键字，本质上是一个**引用**（指针），用于指代当前对象 —— 即当前正在执行方法的那个对象实例。它的核心作用是区分对象的成员变量与局部变量，以及在类内部访问当前对象的其他成员。

#### `this` 的主要用法

1. **区分成员变量和局部变量**

   当方法的参数名或局部变量名与类的成员变量名相同时，`this` 可以明确指定访问的是成员变量。

   ```java
   public class Person {
       private String name; // 成员变量
       
       // 构造方法
       public Person(String name) { 
           this.name = name; // this.name 指代成员变量，name 指代参数
       }
       
       // 普通方法
       public void setName(String name) {
           this.name = name; // 同样用于区分
       }
   }
   ```

2. **调用当前类的其他构造方法**

   在构造方法中，可以通过 `this(参数)` 调用当前类的其他构造方法（**构造方法重载**时使用），且必须放在构造方法的第一行。

   ```java
   public class Person {
       private String name;
       private int age;
       
       // 无参构造
       public Person() {
           this("默认名称", 0); // 调用有参构造
       }
       
       // 有参构造
       public Person(String name, int age) {
           this.name = name;
           this.age = age;
       }
   }
   ```

3. **返回当前对象自身**

   在方法中，`return this` 可以返回当前对象的引用，常用于**方法链**编程（连续调用多个方法）。

   ```java
   public class Person {
       private int age;
       
       public Person setAge(int age) {
           this.age = age;
           return this; // 返回当前对象
       }
       
       public void print() {
           System.out.println("年龄：" + age);
       }
       
       public static void main(String[] args) {
           // 方法链调用
           new Person().setAge(20).print(); // 输出：年龄：20
       }
   }
   ```

4. **作为参数传递给其他方法**

   可以将当前对象通过 `this` 传递给其他方法，让外部方法操作当前对象。

   ```java
   public class Person {
       private String name;
       
       public void show() {
           OtherClass.process(this); // 将当前对象传给其他类的方法
       }
   }
   
   class OtherClass {
       public static void process(Person p) {
           // 操作传递过来的 Person 对象（即调用者）
       }
   }
   ```

##### 注意事项

- `this` 只能在**非静态方法**或**构造方法**中使用，静态方法（`static` 修饰）中不能使用 `this`（因为静态方法属于类，不依赖对象实例）。
- `this` 不能用于指代类本身，只能指代当前对象实例。

总结：`this` 的核心是 “当前对象” 的引用，通过它可以清晰地访问当前对象的成员、调用构造方法或传递自身引用，是 Java 面向对象编程中区分对象状态的重要工具。

### 3 .封装

在 Java 中，封装（Encapsulation）是面向对象编程（OOP）的三大核心特性之一（另外两个是继承和多态），其核心思想是**将类的数据（属性）和操作数据的方法（行为）捆绑在一起，并对外部隐藏内部实现细节**，仅通过公共接口与外部交互。

#### 封装的实现步骤

1. **隐藏数据（属性私有化）**

   将类的属性使用 `private` 修饰符，限制外部直接访问。

2. **提供公共接口（ getter/setter 方法）**

   通过 `public` 修饰的方法（通常命名为 `getXxx()` 和 `setXxx()`）允许外部间接访问或修改属性，同时可以在方法中添加逻辑验证，保证数据的合法性。

#### 语法示例

```java
public class Person {
    // 1. 属性私有化（隐藏数据）
    private String name;  // 姓名
    private int age;      // 年龄

    // 2. 提供公共的 getter 方法（获取属性值）
    public String getName() {
        return name;
    }

    // 3. 提供公共的 setter 方法（设置属性值，可添加验证）
    public void setName(String name) {
        // 例如：限制姓名不为空
        if (name != null && !name.isEmpty()) {
            this.name = name;
        } else {
            System.out.println("姓名不能为空！");
        }
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        // 例如：限制年龄在合理范围（0-120）
        if (age >= 0 && age <= 120) {
            this.age = age;
        } else {
            System.out.println("年龄必须在0-120之间！");
        }
    }

    // 其他业务方法（封装行为）
    public void introduce() {
        System.out.println("我叫" + name + "，今年" + age + "岁。");
    }
}
```

#### 调用封装类

外部无法直接访问 `private` 属性，必须通过 `getter/setter` 方法操作：

```java
public class Test {
    public static void main(String[] args) {
        Person p = new Person();
        
        // 错误：直接访问 private 属性会编译报错
        // p.name = "张三"; 
        // p.age = 200;

        // 正确：通过 setter 方法设置属性（自动触发验证）
        p.setName("张三");
        p.setAge(200);  // 会提示“年龄必须在0-120之间！”（age 保持默认值 0）
        p.setAge(25);   // 合法，age 被设置为 25

        // 通过 getter 方法获取属性
        System.out.println("姓名：" + p.getName());  // 输出：姓名：张三
        System.out.println("年龄：" + p.getAge());    // 输出：年龄：25

        p.introduce();  // 输出：我叫张三，今年25岁。
    }
}
```

#### 封装的优势

1. **数据安全性**：通过 `setter` 方法的逻辑验证，避免非法数据进入对象。
2. **隐藏实现细节**：外部只需关注如何使用（接口），无需知道内部如何实现。
3. **代码可维护性**：内部实现修改时，只要接口不变，外部调用代码无需修改。
4. **模块化**：将数据和操作封装在类中，实现代码的模块化组织。

#### 注意事项

- 并非所有属性都必须提供 `getter/setter`，例如只读属性（只提供 `getter`）或只写属性（只提供 `setter`）。
- 对于 `boolean` 类型的属性，`getter` 方法通常命名为 `isXxx()` 而非 `getXxx()`（例如 `isMarried()`）。
- 封装不仅限于属性，也包括类的方法（通过访问修饰符控制方法的可见性）。

### 4. Javabean

JavaBean 是 Java 语言中一种遵循特定规范的可重用组件模型，主要用于封装数据（也可包含简单的业务逻辑），便于在 Java 应用（尤其是 Java EE 开发、GUI 程序等）中进行数据传递和操作。

#### JavaBean 的核心规范（必须遵守）

1. **类的可访问性**：类必须是 `public` 修饰的，确保能被外部访问。
2. **无参构造方法**：必须提供一个无参数的 `public` 构造方法（即使手动定义了其他构造方法，也需显式声明无参构造），便于框架（如 Spring、Servlet）通过反射创建实例。
3. **属性私有化与 getter/setter**：
   - 类中的成员变量（属性）需用 `private` 修饰，隐藏内部实现。
   - 为每个属性提供 `public` 的 getter 方法（用于读取属性值）和 setter 方法（用于修改属性值），命名需遵循规范：
     - 对于属性 `xxx`，getter 方法为 `getXxx()`（布尔类型属性可简化为 `isXxx()`），setter 方法为 `setXxx(参数)`。

#### 示例代码

```java
public class User {
    // 私有属性
    private String name;
    private int age;
    private boolean isStudent;

    // 无参构造方法
    public User() {}

    // getter/setter 方法
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    // 布尔类型属性的 getter 可使用 isXxx()
    public boolean isStudent() {
        return isStudent;
    }

    public void setStudent(boolean isStudent) {
        this.isStudent = isStudent;
    }
}
```

#### JavaBean 的主要作用

1. **数据封装**：将数据（如用户信息、订单信息）及其操作（get/set）封装在类中，符合面向对象的封装特性。
2. **数据传递**：在多层架构（如 MVC）中，作为数据载体在层间传递（例如 Servlet 接收前端数据后，用 JavaBean 封装并传递给 Service 层）。
3. **框架适配**：许多 Java 框架（如 Spring、MyBatis、Struts）依赖 JavaBean 规范进行反射操作（如对象实例化、属性赋值），是框架交互的基础组件。
4. **可重用性**：标准化的设计使其可在不同场景中复用，减少重复代码。

#### 扩展说明

- JavaBean 通常仅包含属性和 getter/setter，也可包含简单的业务逻辑（如属性验证），但复杂逻辑一般放在 Service 层。
- 在 Java EE 中，JavaBean 常与 JSP 标签库（如 JSTL）配合使用，简化页面数据展示。
- 随着技术发展，一些框架（如 Spring Boot）对 JavaBean 的规范要求有所放宽（如通过 Lombok 的 `@Data` 注解自动生成 getter/setter），但核心思想仍保持一致。

总之，JavaBean 是 Java 开发中用于数据封装和传递的基础组件，其规范保证了代码的标准化和可扩展性。

### 5. static

在 Java 中，`static` 是一个关键字，用于修饰类的成员（变量、方法、代码块、内部类），表示该成员属于类本身，而非类的实例（对象）。这意味着**被 `static` 修饰的成员不需要创建对象就能访问，且所有实例共享同一份数据。**

#### 1. 静态变量（类变量）

- **定义**：用 `static` 修饰的成员变量，属于类，存储在方法区的静态区，只有一份副本。
- **访问方式**：可以通过 `类名.变量名` 直接访问，也可通过实例访问（但不推荐，易混淆）。
- **特点**：所有实例共享该变量的值，修改一个实例的静态变量会影响其他所有实例。

```java
public class Student {
    // 静态变量（类变量）：所有学生共享同一个学校名称
    public static String schoolName = "阳光中学";
    // 实例变量：每个学生有自己的姓名
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        // 直接通过类名访问静态变量
        System.out.println(Student.schoolName); // 输出：阳光中学

        Student s1 = new Student("张三");
        Student s2 = new Student("李四");

        // 通过实例访问静态变量（不推荐）
        System.out.println(s1.schoolName); // 输出：阳光中学

        // 修改静态变量，所有实例都会受影响
        Student.schoolName = "希望中学";
        System.out.println(s2.schoolName); // 输出：希望中学
    }
}
```

#### 2. 静态方法（类方法）

- **定义**：用 `static` 修饰的方法，属于类，可直接通过类名调用。
- **特点**：
  - 静态方法中不能直接访问非静态成员（变量 / 方法），因为非静态成员依赖于实例。
  - 静态方法中不能使用 `this` 或 `super` 关键字（它们指向当前实例）。
  - 非静态方法可以访问静态成员。
- **常见场景**：工具类方法（如 `Arrays.sort()`、`Math.random()`）、工厂方法等。

```java
public class MathUtil {
    // 静态方法：计算两数之和
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        // 直接通过类名调用静态方法
        int sum = MathUtil.add(3, 5);
        System.out.println(sum); // 输出：8
    }
}
```

#### 3. 静态代码块

- **定义**：用 `static` 修饰的代码块，在类加载时执行（仅执行一次），优先级高于构造方法。
- **作用**：用于初始化静态变量、加载资源（如配置文件）等类级别的初始化操作。

```java
public class StaticBlockDemo {
    static int num;

    // 静态代码块
    static {
        System.out.println("静态代码块执行");
        num = 100; // 初始化静态变量
    }

    public StaticBlockDemo() {
        System.out.println("构造方法执行");
    }

    public static void main(String[] args) {
        StaticBlockDemo obj1 = new StaticBlockDemo();
        StaticBlockDemo obj2 = new StaticBlockDemo();
        // 输出：
        // 静态代码块执行（类加载时执行一次）
        // 构造方法执行（obj1 创建时）
        // 构造方法执行（obj2 创建时）
    }
}
```

#### 4. 静态内部类

- **定义**：用 `static` 修饰的内部类，属于外部类本身，而非外部类的实例。
- **特点**：
  - 可以直接访问外部类的静态成员，但不能直接访问非静态成员。
  - 创建静态内部类实例时，无需先创建外部类实例。
- **用途**：通常用于与外部类逻辑相关但不依赖外部类实例的场景（如工具类的辅助类）。

```java
public class OuterClass {
    private static int staticVar = 10;
    private int nonStaticVar = 20;

    // 静态内部类
    public static class StaticInnerClass {
        public void print() {
            System.out.println("外部类静态变量：" + staticVar);
            // 无法直接访问外部类非静态变量：System.out.println(nonStaticVar);
        }
    }

    public static void main(String[] args) {
        // 直接创建静态内部类实例，无需外部类实例
        OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
        inner.print(); // 输出：外部类静态变量：10
    }
}
```

#### 总结 `static` 的核心特性：

- **属于类而非实例**：无需创建对象即可访问。
- **共享性**：静态变量在所有实例间共享。
- **类加载时初始化**：静态成员在类加载到内存时初始化，生命周期与类一致。
- **访问限制**：静态方法只能直接访问静态成员，非静态方法可访问所有成员。

---

## 🌟继承

在 Java 中，继承是面向对象编程的核心特性之一，它允许一个类（子类）继承另一个类（父类）的属性和方法，从而实现代码复用和功能扩展。以下是 Java 继承的语法规则和关键细节：

### 1. 基本语法

通过 `extends` 关键字声明继承关系，格式如下：

```java
// 父类（基类/超类）
class 父类名 {
    // 成员变量、方法等
}

// 子类（派生类），通过 extends 继承父类
class 子类名 extends 父类名 {
    // 可以添加子类自己的成员变量和方法
    // 也可以重写父类的方法（覆盖）
}
```

**示例**：

```java
// 父类：动物
class Animal {
    String name;
    void eat() {
        System.out.println(name + "在吃东西");
    }
}

// 子类：狗（继承自动物）
class Dog extends Animal {
    // 子类新增方法
    void bark() {
        System.out.println(name + "在叫：汪汪");
    }
}
```

### 2. 继承的核心特性

- **单继承限制**：**Java 只支持单继承**，即一个子类只能直接继承一个父类（避免多继承的歧义问题）。但可以通过多层继承间接继承多个类（如 `A extends B`，`B extends C`，则 `A` 间接继承 `C`）。

- **成员的继承规则**：

  - 子类继承父类中 **非 private** 的成员变量和方法（`public`、`protected` 及默认访问权限的成员，默认权限需在同一包内）。
  - 父类的 `private` 成员无法被子类直接访问，但可通过父类的 `public` 或 `protected` 方法间接访问。

- **构造方法的继承**：

  - 子类 **不能继承** 父类的构造方法，但子类构造方法中必须先**调用**父类的构造方法（通过 `super()` 实现），默认会隐式调用父类的无参构造。

  ```java
  class Parent {
      // 父类无参构造
      public Parent() {
          System.out.println("父类无参构造被调用");
      }
  }
  
  class Child extends Parent {
      public Child() {
          // 隐式调用 super()，无需手动写
          System.out.println("子类构造被调用");
      }
  }
  
  // 输出：
  // 父类无参构造被调用
  // 子类构造被调用
  ```

  - 若父类没有无参构造，子类必须显式通过 `super(参数)` 调用父类的有参构造，且 **`super()` 必须放在子类构造方法的第一行。**

  > 父类没有午餐构造的情况下，才必须要supper在第一行
  
  ```java
  public class test {
      class Parent {
          // 父类只有有参构造，没有无参构造
          public Parent(String name, int age) {
              System.out.println("父类有参构造被调用，name=" + name);
          }
      }
  
      class Child extends Parent {
          private String hobby;
          
          public Child() {
              // 调用本类的另一个构造器
              this("默认姓名", 18, "未知爱好");
          }
  
          public Child(String name, int age) {
              // 调用本类的三个参数构造器
              this(name, age, "未指定");
          }
  
          public Child(String name, int age, String hobby) {
              // 调用父类构造器
              super(name, age);
              this.hobby = hobby;
              System.out.println("子类构造被调用，爱好：" + hobby);
          }
      }
  }
  ```

> - `this` 和 `super` 都不能在静态方法（`static`）中使用，因为它们依赖于实例对象，而静态方法属于类本身。
> - 构造器中，`this(...)` 和 `super(...)` 不能同时出现，且都必须放在第一行。**this主要用于在当前类的一个构造器中调用另一个构造器（避免代码重复）**

### 3. 方法重写（Override）

子类可以重写父类的方法，以实现自己的逻辑，需满足以下规则：

- 方法名、参数列表（类型、顺序、数量）必须与父类完全一致。
- 返回值类型：父类返回类型为 `T`，子类可返回 `T` 或 `T` 的子类（协变返回类型）。
- 访问权限：子类重写的方法权限 **不能低于** 父类（如父类是 `protected`，子类可改为 `public`，但不能改为 `private`）。
- 不能重写父类的 `final` 方法（被 `final` 修饰的方法不可修改）。
- 用 `@Override` 注解可显式标记重写方法，帮助编译器检查语法正确性。

**示例**：

```java
class Animal {
    void makeSound() {
        System.out.println("动物发出声音");
    }
}

class Cat extends Animal {
    @Override // 显式标记重写
    void makeSound() {
        System.out.println("猫发出：喵喵"); // 重写父类方法
    }
}
```

### 4. `super` 关键字

在子类中，`super` 用于访问父类的成员或构造方法：

- `super.成员变量`：访问父类的成员变量（当子类与父类有同名变量时区分）。
- `super.方法名()`：调用父类的方法（当子类重写了父类方法时使用）。
- `super()`：调用父类的构造方法（必须在子类构造方法的第一行）。

**示例**：

```java
class Parent {
    String name = "父类名称";
    void print() {
        System.out.println("父类方法");
    }
}

class Child extends Parent {
    String name = "子类名称";
    void print() {
        super.print(); // 调用父类的print方法
        System.out.println("子类方法");
        System.out.println(super.name); // 访问父类的name变量
    }
}
```

### 5. `final` 与继承

- 被 `final` 修饰的类 **不能被继承**（如 `final class FinalClass { ... }`，任何类不能 `extends FinalClass`）。
- 被 `final` 修饰的方法 **不能被重写**，但可被继承使用。

### 总结

Java 继承通过 `extends` 实现单继承，核心是代码复用和扩展。子类可继承父类的非私有成员，通过重写方法定制行为，通过 `super` 关键字访问父类资源。合理使用继承能显著提高代码的可维护性和扩展性。

---

## 🔥权限修饰符

在 Java 中，权限修饰符用于控制类、成员变量、方法和构造方法的访问范围，共有 4 种，按访问权限从大到小依次为：`public`、`protected`、`default`（默认，无关键字）、`private`。它们的区别主要体现在访问范围上，具体如下：

### 1. `public`（公共的）

- **访问范围最广**，可以被以下范围访问：
  - 本类内部
  - 同一包中的其他类
  - 不同包中的子类
  - 不同包中的非子类（只要能访问到类本身）
- **适用场景**：希望被全局访问的类、方法或变量（如工具类的静态方法、对外暴露的接口方法）。

### 2. `protected`（受保护的）

- **访问范围次之**，可被以下范围访问：
  - 本类内部
  - 同一包中的其他类（无论是否为子类）
  - 不同包中的子类（通过子类对象访问，不能通过父类对象直接访问）✅
- **不允许**：不同包中的非子类访问。
- **适用场景**：父类中需要被子类继承和修改的成员（如模板方法中的钩子方法）。

### 3. `default`（默认，无修饰符）

- 当没有显式指定权限修饰符时，默认为此权限。
- **访问范围**：仅能被**本类内部**和**同一包中的其他类**访问。
- **不允许**：不同包中的类（包括子类）访问。
- **适用场景**：同一包内的类之间共享逻辑，但不希望被外部包访问（如包内工具类的辅助方法）。

### 4. `private`（私有的）

- **访问范围最小**，仅能在**本类内部**访问。
- **不允许**：任何外部类（包括同一包的类、子类）访问。
- **适用场景**：类的内部实现细节（如私有成员变量、内部工具方法），通常通过 `public` 或 `protected` 的方法（getter/setter）间接访问。

### 总结表格

| 修饰符      | 本类内部 | 同一包中其他类 | 不同包的子类 | 不同包的非子类 |
| ----------- | -------- | :------------- | ------------ | -------------- |
| `public`    | ✅        | ✅              | ✅            | ✅              |
| `protected` | ✅        | ✅              | ✅            | ❌              |
| `default`   | ✅        | ✅              | ❌            | ❌              |
| `private`   | ✅        | ❌              | ❌            | ❌              |

### 注意

- 类的权限修饰符只能是 `public` 或 `default`（外部类），内部类可以使用所有 4 种修饰符。
- 权限修饰符的设计核心是**封装**：通过限制访问范围，隐藏内部实现，只暴露必要的接口，提高代码的安全性和可维护性。

---

## 💧多态

在 Java 中，**多态（Polymorphism）** 是面向对象编程（OOP）的三大核心特性之一（另外两个是封装和继承），核心思想是 “**同一行为，不同实现**”。简单来说，就是**通过父类的引用指向子类的对象**时，调用方法会根据对象的实际类型执行对应的逻辑，而不是父类的逻辑。

#### 多态的核心表现

当父类引用指向子类对象时，调用被重写的方法会执行子类的实现，而非父类的原始实现。例如：

```java
// 父类
class Animal {
    public void sound() {
        System.out.println("动物发出声音");
    }
}

// 子类1
class Dog extends Animal {
    @Override // 重写父类方法
    public void sound() {
        System.out.println("狗汪汪叫");
    }
}

// 子类2
class Cat extends Animal {
    @Override // 重写父类方法
    public void sound() {
        System.out.println("猫喵喵叫");
    }
}

public class Test {
    public static void main(String[] args) {
        Animal animal1 = new Dog(); // 父类引用指向Dog对象
        Animal animal2 = new Cat(); // 父类引用指向Cat对象
        
        animal1.sound(); // 输出：狗汪汪叫（执行Dog的实现）
        animal2.sound(); // 输出：猫喵喵叫（执行Cat的实现）
    }
}
```

上述代码中，`animal1` 和 `animal2` 都是 `Animal` 类型的引用，但分别指向 `Dog` 和 `Cat` 对象。调用 `sound()` 方法时，实际执行的是子类重写后的逻辑，这就是多态的体现。

#### 多态的实现条件

1. **继承关系**：必须存在父类与子类（包括类继承 `extends` 或接口实现 `implements`）。
2. **方法重写**：子类必须重写（`override`）父类中的方法（注意：静态方法、私有方法、`final` 方法不能被重写）。
3. **父类引用指向子类对象**：通过 `父类类型 变量名 = new 子类类型()` 的方式定义对象。

#### 多态的作用

1. **提高代码灵活性**：同一方法调用可适配不同对象，无需为每个子类单独写逻辑。
2. **增强扩展性**：新增子类时，只需重写父类方法，无需修改调用方代码（符合 “开闭原则”）。
3. **简化代码逻辑**：通过父类统一管理不同子类对象，减少冗余代码。

#### 多态的注意事项

- 多态仅针对**方法**有效，对**属性**无效（访问属性时，仍以声明的父类类型为准）。
- 若要调用子类特有的方法（未在父类中定义），需通过**类型转换**（`(子类类型) 父类引用`）实现，但需避免 “ClassCastException”（类型转换异常）。

```java
Animal animal = new Dog();
// animal.fetch(); // 错误：父类引用无法直接调用子类特有方法
Dog dog = (Dog) animal; // 强制转换为Dog类型
dog.fetch(); // 正确：调用Dog的特有方法
```

总结：多态是 Java 中实现代码复用和灵活设计的重要机制，通过 “父类定义规范，子类实现细节” 的方式，让程序更易于维护和扩展。

---

## 以后就不做详细的笔记了

> Java 规定：**一个`.java`文件中，最多只能有一个`public`类，且这个`public`类的类名必须和文件名完全一致**。

---

## 👍抽象类

### 一、抽象类的定义

抽象类是被`abstract`关键字修饰的类，它是一种不能被实例化的类，主要用于作为其他类的父类，定义通用的属性和方法，同时可以包含未实现的抽象方法，由子类去具体实现。

**语法格式**：

```java
[访问修饰符] abstract class 类名 {
    // 成员变量
    // 构造方法（抽象类可以有构造方法，用于子类初始化）
    // 普通方法
    // 抽象方法（没有方法体）
    [访问修饰符] abstract 返回值类型 方法名(参数列表);
}
```

### 二、核心特性

1. **不能实例化**

   抽象类无法通过`new`关键字创建对象，例如：

   ```java
   abstract class Shape {}
   // Shape shape = new Shape(); // 编译错误：抽象类不能实例化
   ```

2. **可以包含抽象方法**

   抽象方法是**没有方法体**的方法（用`;`结尾），必须由子类重写（除非子类也是抽象类）。

   ```java
   abstract class Shape {
       public abstract double getArea(); // 抽象方法
   }
   ```

3. **可以包含普通方法和成员变量**

   抽象类允许有已实现的方法、成员变量、静态方法等，例如：

   ```java
   abstract class Shape {
       protected String color;
       
       public Shape(String color) {
           this.color = color;
       }
       
       public void showColor() { // 普通方法
           System.out.println("颜色：" + color);
       }
       
       public abstract double getArea(); // 抽象方法
   }
   ```

4. **子类必须实现抽象方法**

   如果子类不是抽象类，则必须重写父类所有的抽象方法：

   ```java
   class Circle extends Shape {
       private double radius;
       
       public Circle(String color, double radius) {
           super(color);
           this.radius = radius;
       }
       
       @Override
       public double getArea() { // 必须实现抽象方法
           return Math.PI * radius * radius;
       }
   }
   ```

5. **可以有构造方法**

   抽象类的构造方法用于子类初始化时调用（通过`super()`），但不能直接用于创建对象。

### 三、抽象类的使用场景

1. **定义模板**：抽象类可以定义一组通用的方法和属性，子类根据自身特性实现抽象方法（模板方法模式）。
2. **代码复用**：将多个子类的公共逻辑抽取到抽象类中，减少重复代码。
3. **限制实例化**：明确某个类只能作为父类，不能独立存在（如`Shape`、`Animal`等概念类）。

### 四、抽象类与接口的区别

| 特性     | 抽象类                | 接口                                                         |
| -------- | --------------------- | ------------------------------------------------------------ |
| 实例化   | 不能实例化            | 不能实例化                                                   |
| 方法     | 可包含抽象 / 普通方法 | Java 8 + 可包含默认 / 静态方法，其余为抽象方法，只有方法签名，无方法实现 |
| 成员变量 | 可包含各种变量        | 默认`public static final`                                    |
| 继承关系 | 单继承                | 多实现                                                       |
| 构造方法 | 可以有                | 不能有                                                       |

### 五、注意事项

1. **`abstract`关键字不能与`final`、`static`、`private`同时修饰方法**：

   - `final`方法不能被重写，与抽象方法矛盾；
   - `static`方法属于类，不能被子类重写；
   - `private`方法对子类不可见，无法重写。

2. **抽象类可以继承其他类**：

   抽象类可以继承普通类或另一个抽象类。

3. **抽象类可以实现接口**：

   抽象类实现接口时，可以不实现接口的方法（由子类实现）。

### 六、示例代码

```java
// 抽象类
abstract class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    public abstract void makeSound(); // 抽象方法
    
    public void eat() { // 普通方法
        System.out.println(name + "在吃东西");
    }
}

// 子类实现抽象方法
class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + "喵喵叫");
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        Animal cat = new Cat("加菲猫");
        cat.makeSound(); // 输出：加菲猫喵喵叫
        cat.eat();       // 输出：加菲猫在吃东西
    }
}
```

### 总结

抽象类是 Java 中实现抽象和继承的重要机制，通过抽象方法强制子类实现特定行为，同时通过普通方法提供通用逻辑，适用于需要定义模板但不允许实例化的场景。

---

## 🪵接口

在 Java 中，“接口（Interface）” 是一种特殊的引用类型，它定义了一组方法的规范（契约），但不提供方法的具体实现（Java 8 后支持默认方法和静态方法的实现）。接口是实现多态、解耦和代码规范的重要工具。

### 一、接口的定义

使用`interface`关键字定义接口，语法如下：

```java
[访问修饰符] interface 接口名 [extends 父接口1, 父接口2...] {
    // 常量（默认public static final）-- 公开静态常量
    // 抽象方法（默认public abstract）
    // 默认方法（Java 8+，用default修饰）
    // 静态方法（Java 8+，用static修饰）
    // 私有方法（Java 9+，用private修饰）
}
```

> - **`public`**：接口成员变量默认是公共的，可被所有实现类或外部类直接访问。
> - **`static`**：属于接口本身，而非实现类的实例，通过`接口名.变量名`即可访问（如`YourInterface.CATEGORY`）。
> - **`final`**：必须在声明时初始化，且值不可修改（常量），实现类无法重定义或改变其值。

```java
public interface Animal {
    // 常量（隐式public static final）
    String CATEGORY = "生物";

    // 抽象方法（隐式public abstract）
    void eat();
    void move();

    // 默认方法（有实现）
    default void sleep() {
        System.out.println("动物睡觉");
    }

    // 静态方法
    static void breathe() {
        System.out.println("动物呼吸");
    }
}
```

### 二、接口的核心特性

1. **抽象性**：

   - 接口中的抽象方法没有方法体，必须由实现类实现。
   - 接口不能被实例化（`new Animal()`会报错）。

2. **常量定义**：

   - 接口中的变量默认是`public static final`（常量），必须初始化。
   - 例如：`String CATEGORY = "生物";` 等价于 `public static final String CATEGORY = "生物";`。

3. **多继承**：

   - 接口可以继承多个父接口（用`,`分隔），弥补了 Java 类单继承的局限。

   ```java
   interface Flyable {
       void fly();
   }
   
   interface Swimmable {
       void swim();
   }
   
   interface Amphibious extends Flyable, Swimmable {
       // 继承了fly()和swim()方法
   }
   ```

4. **默认方法（Java 8+）**：

   - 用`default`修饰，允许接口提供方法的默认实现。
   - 实现类可以重写默认方法，也可以直接使用。

   > **访问范围**：仅能被**本类内部**和**同一包中的其他类**访问。

5. **静态方法（Java 8+）**：

   - 用`static`修饰，属于接口本身，通过接口名调用（`Animal.breathe()`）。

6. **私有方法（Java 9+）**：

   - 用`private`修饰，用于接口内部方法的复用，不能被实现类访问。

### 三、接口的实现：`implements`

类通过`implements`关键字实现接口，必须实现接口中所有抽象方法（除非是抽象类）。

```java
public class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }

    @Override
    public void move() {
        System.out.println("狗跑");
    }

    // 可选：重写默认方法，默认方法不是抽象方法，所以不强制被重写。但是如果被重写，重写的时候去掉default关键字
    @Override
    public void sleep() {
        System.out.println("狗趴着睡觉");
    }
}
```

#### 多接口实现

```java
public class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("鸭子飞");
    }

    @Override
    public void swim() {
        System.out.println("鸭子游泳");
    }
}
```

### 四、接口的作用

1. **定义规范（契约）**：

   - 接口规定了实现类必须提供的方法，保证代码的规范性和一致性。
   - 例如：`java.util.List`接口定义了列表的操作规范，`ArrayList`和`LinkedList`实现该接口并提供具体实现。

2. **实现多态**：

   - 接口引用可以指向实现类对象，实现 “接口编程”，提高代码灵活性。

   ```java
   Animal animal = new Dog();
   animal.eat(); // 调用Dog的eat()方法
   ```

3. **解耦**：

   - 接口隔离了方法定义与实现，降低模块间依赖，便于扩展和维护。

4. **弥补单继承限制**：

   - 一个类可以实现多个接口，实现类似 “多继承” 的效果。

### 五、接口与抽象类的区别

| 特性     | 接口                                              | 抽象类                   |
| -------- | ------------------------------------------------- | ------------------------ |
| 继承限制 | 类可实现多个接口                                  | 类只能继承一个抽象类     |
| 方法     | 默认抽象方法，支持默认 / 静态方法，私有（jdk 9+） | 可包含抽象方法和具体方法 |
| 变量     | 只能是 public static final 常量                   | 可包含各种变量           |
| 构造方法 | 无                                                | 有（不能实例化）         |
| 设计目的 | 定义规范（WHAT）                                  | 代码复用（HOW）          |

### 六、常见应用场景

- **定义 API 规范**：如`java.lang.Comparable`接口（定义比较规则）。
- **回调机制**：如`java.awt.event.ActionListener`（事件监听）。
- **标记接口**：如`java.io.Serializable`（仅标记可序列化，无方法）。
- **策略模式**：通过接口定义算法族，实现类替换不同算法。

### 总结

接口是 Java 中实现抽象和多态的核心机制，专注于**定义规范**，强调 “做什么” 而非 “怎么做”。合理使用接口可以提高代码的可扩展性、灵活性和规范性，是面向对象设计的重要工具。

---

## ❌接口、抽象类、现实类的代码区别

```java
package demo.check.getNewKnoledge;

import java.util.Random;

/**
 * 演示接口与抽象类的核心区别、JDK新特性及多继承/多实现机制
 * 包含：接口默认方法、静态方法、私有方法（JDK9+）；抽象类构造器；变量类型区别；多实现冲突解决等
 */
public class InterfaceVsAbstractDemo {

    // 1. 基础接口定义（JDK7及之前特性）
    interface BaseInterface {
        // 公开静态常量（默认public static final）
        String BASE_CONSTANT = "基础接口常量";

        // 抽象方法（默认public abstract）
        void abstractMethod();
    }

    // 2. JDK8+接口新特性：默认方法、静态方法
    interface Jdk8Interface extends BaseInterface {
        // 默认方法（可带实现，实现类可重写）
        default void defaultMethod() {
            System.out.println("JDK8默认方法实现");
            // 调用接口私有方法（JDK9+）
            privateMethod();
        }

        // 静态方法（属于接口，只能通过接口名调用）
        static void staticMethod() {
            System.out.println("JDK8接口静态方法");
            staticPrivateMethod(); // 调用静态私有方法
        }

        // JDK9+私有方法（接口内部复用逻辑）
        private void privateMethod() {
            System.out.println("JDK9接口私有实例方法");
        }

        // JDK9+静态私有方法
        private static void staticPrivateMethod() {
            System.out.println("JDK9接口私有静态方法");
        }
    }

    // 3. 接口多继承（接口可继承多个接口）
    interface SubInterface extends Jdk8Interface, AnotherInterface {
        // 继承多个接口的所有抽象方法、默认方法
        // 若父接口默认方法冲突，需在此重写解决
        @Override
        default void defaultMethod() {
            // 显式调用指定父接口的默认方法
            Jdk8Interface.super.defaultMethod();
            AnotherInterface.super.defaultMethod();
        }
    }

    // 辅助接口（用于多继承演示）
    interface AnotherInterface {
        default void defaultMethod() {
            System.out.println("AnotherInterface默认方法");
        }
        // 公开静态常量（默认public static final）
        String anotherConstant = "Another接口常量";
    }

    // 4. 抽象类定义（对比接口）
    abstract static class AbstractClass {
        // 抽象类可以有各种访问修饰符的成员变量
        String abstractField = "抽象类实例变量";
        static String staticField = "抽象类静态变量";
        private int privateField;

        // 抽象类构造器（接口无构造器）
        public AbstractClass() {
            this.privateField = new Random().nextInt(100);
        }

        // 抽象方法（必须子类实现）
        public abstract void abstractClassMethod();

        // 具体方法（可被子类重写）
        public void concreteMethod() {
            System.out.println("抽象类具体方法：" + privateField);
        }

        // 静态方法（属于抽象类）
        public static void staticClassMethod() {
            System.out.println("抽象类静态方法");
        }
    }

    // 5. 实现类：多实现+继承抽象类 。--》 这是一个静态内部类，属于外部类本身
    static class ConcreteClass extends AbstractClass implements SubInterface, AnotherInterface {

        // 实现接口抽象方法
        @Override
        public void abstractMethod() {
            System.out.println("实现BaseInterface的抽象方法");
        }

        // 实现抽象类抽象方法
        @Override
        public void abstractClassMethod() {
            System.out.println("实现抽象类的抽象方法");
        }

        // 可选：重写接口默认方法 -- 不是必须重写的核心原因是：冲突已经在子接口SubInterface中解决了
        @Override
        public void defaultMethod() {
            System.out.println("实现类重写默认方法");
            // 调用父接口默认方法
            SubInterface.super.defaultMethod();
        }

        // 自定义方法
        public void testVariables() {
            // 访问各类变量
            System.out.println("接口常量：" + BASE_CONSTANT); // 接口常量，// 公开静态常量（默认public static final）
            System.out.println("抽象类实例变量：" + abstractField); // 抽象类实例变量
            System.out.println("抽象类静态变量：" + AbstractClass.staticField); // 抽象类静态变量
            // System.out.println(privateField); // 无法访问抽象类私有变量
        }
    }

    public static void main(String[] args) {
        // 1. 接口特性演示
        System.out.println("=== 接口特性 ===");
        Jdk8Interface.staticMethod(); // 调用接口静态方法
        System.out.println("接口常量：" + BaseInterface.BASE_CONSTANT);

        // 2. 抽象类特性演示
        System.out.println("\n=== 抽象类特性 ===");
        AbstractClass.staticClassMethod(); // 调用抽象类静态方法

        // 3. 实现类演示
        System.out.println("\n=== 实现类特性 ===");
        ConcreteClass concrete = new ConcreteClass();
        concrete.abstractMethod(); // 实现接口方法
        concrete.abstractClassMethod(); // 实现抽象类方法
        concrete.defaultMethod(); // 重写默认方法
        concrete.concreteMethod(); // 继承抽象类具体方法
        concrete.testVariables(); // 变量访问测试

        // 4. 多态演示
        System.out.println("\n=== 多态演示 ===");
        BaseInterface base = new ConcreteClass();
        base.abstractMethod(); // 调用实现类方法
        // base.defaultMethod(); // JDK8+接口默认方法可通过接口引用调用
        Jdk8Interface jdk8 = (Jdk8Interface) base;
        jdk8.defaultMethod();
    }
}
```

#### 输出

```cmd
=== 接口特性 ===
JDK8接口静态方法
JDK9接口私有静态方法
接口常量：基础接口常量

=== 抽象类特性 ===
抽象类静态方法

=== 实现类特性 ===
实现BaseInterface的抽象方法
实现抽象类的抽象方法
实现类重写默认方法
JDK8默认方法实现
JDK9接口私有实例方法
AnotherInterface默认方法
抽象类具体方法：4  // 数字是任意的
接口常量：基础接口常量
抽象类实例变量：抽象类实例变量
抽象类静态变量：抽象类静态变量

=== 多态演示 ===
实现BaseInterface的抽象方法
实现类重写默认方法
JDK8默认方法实现
JDK9接口私有实例方法
AnotherInterface默认方法
```

#### 接口、抽象类、实现类的成员调用对比表

| 成员类型         | 接口                            | 抽象类                   | 实现类                         |
| ---------------- | ------------------------------- | ------------------------ | ------------------------------ |
| **静态变量**     | 接口名调用（`Interface.CONST`） | 抽象类名 / 子类名调用    | 实现类名调用                   |
| **静态方法**     | 仅接口名调用                    | 抽象类名 / 子类名调用    | 实现类名调用                   |
| **default 方法** | 接口引用 / 实例调用，可被重写   | 无此特性                 | 可重写，重写后实例调用优先     |
| **实例变量**     | 无（接口变量默认 static）       | 实例调用（`obj.field`）  | 继承抽象类的实例变量，实例调用 |
| **抽象方法**     | 实现类实例调用（重写后）        | 实现类实例调用（重写后） | 自身实例调用                   |

1. **`static`的本质是 “类级共享”**：

   无论接口还是抽象类，`static`成员均脱离实例存在，接口静态方法更强调 “接口专属功能”，抽象类静态方法则偏向 “工具方法复用”。

2. **`default`的本质是 “接口功能扩展”**：

   既保证接口的 “规范定义” 属性，又允许提供默认实现，解决了接口升级的兼容性问题；但多继承时需手动解决冲突，体现了 Java“单继承、多实现” 的设计平衡。

3. **调用逻辑的核心是 “归属关系”**：

   - 属于类 / 接口的成员（`static`），用类名 / 接口名调用；
   - 属于实例的成员（非`static`），用实例调用；
   - 接口`default`方法虽属接口，但需通过实例触发，重写后优先执行实现类逻辑。

> Q：继承，实现的类中对父类或者接口的成员变量的调用不需要写出类名对吗？
>
> A：**无冲突时可省略类名 / 接口名，有冲突或静态变量推荐显式指定类名 / 接口名**。

| 变量类型                 | 是否需要类名 / 接口名          | 最佳实践                 |
| ------------------------ | ------------------------------ | ------------------------ |
| 抽象类实例变量（非私有） | 无需，直接访问                 | 直接访问或加`this`       |
| 抽象类静态变量           | 可直接访问，冲突时必须加类名   | 加抽象类名访问（清晰）   |
| 接口静态常量             | 可直接访问，冲突时必须加接口名 | 加接口名访问（避免歧义） |
| 私有变量                 | 无法直接访问，与类名无关       | 通过`getter`方法访问     |

#### 多态特性结合接口调用规则

```java
BaseInterface base = new ConcreteClass(); 
```

* //父接口引用指向实现类对象：base是BaseInterface类型的引用变量，但实际指向的是ConcreteClass的实例。
* 编译时类型为`BaseInterface`，运行时类型为`ConcreteClass`。

```java
base.abstractMethod(); // 调用实现类方法
```

* 编译时：编译器检查`BaseInterface`是否声明了`abstractMethod()`（是），语法通过。
* 运行时：根据多态 “运行时类型决定调用对象” 的规则，实际调用`ConcreteClass`中重写的`abstractMethod()`实现。

```java
// base.defaultMethod(); // JDK8+接口默认方法可通过接口引用调用  
```

* 被注释的原因：
  - `BaseInterface`是基础接口（未定义`defaultMethod()`），其编译时类型中没有`defaultMethod()`方法，因此编译器会报错。
  - 即使`ConcreteClass`实现的`SubInterface`（继承自`Jdk8Interface`）有`defaultMethod()`，但编译时`base`的类型是`BaseInterface`，编译器 “看不到” 该方法。

```java
Jdk8Interface jdk8 = (Jdk8Interface) base;        
jdk8.defaultMethod();
```

* 这是**向下转型**（窄化转型）：
  - 将`BaseInterface`类型的`base`强制转换为`Jdk8Interface`类型（因为`ConcreteClass`实际实现了`Jdk8Interface`，转型安全）。
  - 转型后，`jdk8`的编译时类型变为`Jdk8Interface`，编译器可以识别该接口中定义的`defaultMethod()`

---

## 🧊枚举

在 Java 中，枚举（Enum）是一种特殊的数据类型，用于定义一组固定的常量集合。它不仅能替代传统的常量定义方式，还提供了类型安全、方法支持等特性，是 Java 5 及以上版本引入的重要特性。

### 一、枚举类的基本定义

枚举类通过`enum`关键字定义，默认继承自`java.lang.Enum`类（无需显式声明），且所有枚举常量都是该类的实例，默认被`public static final`修饰。

示例：基本枚举类

```java
// 定义枚举类表示季节
enum Season {
    SPRING, SUMMER, AUTUMN, WINTER // 枚举常量，逗号分隔，分号可选（无其他成员时可省略）
}
```

### 二、枚举类的核心特性

1. **类型安全**

   枚举常量是唯一的，且类型固定，避免了使用`int`或`String`常量时的类型错误（如传入非法值）。

   ```java
   Season season = Season.SPRING;
   // 错误示例：无法将int赋值给Season类型
   // Season season = 1; 
   ```

2. **内置方法**

   枚举类继承自`Enum`类，拥有以下常用方法：

   - `values()`：返回包含所有枚举常量的数组（遍历枚举常用）。
   - `valueOf(String name)`：根据名称获取枚举常量（名称必须完全匹配，否则抛`IllegalArgumentException`）。
   - `ordinal()`：返回枚举常量的声明顺序（从 0 开始）。
   - `name()`：返回枚举常量的名称（与`toString()`默认行为一致）。

   #### 示例：内置方法使用

   ```java
   public class EnumTest {
       public static void main(String[] args) {
           // 遍历所有枚举常量
           for (Season s : Season.values()) {
               System.out.println(s + " - 顺序：" + s.ordinal());
           }
           
           // 通过名称获取枚举常量
           Season summer = Season.valueOf("SUMMER");
           System.out.println(summer); // 输出：SUMMER
       }
   }
   ```

3. **自定义成员与方法**

   枚举类可包含字段、构造方法和自定义方法（需注意：枚举常量必须放在第一行，且末尾加分号）。

   #### 示例：带自定义成员的枚举类

   ```java
   enum Season {
       // 枚举常量（调用构造方法初始化）
       SPRING("春天", "温暖"),
       SUMMER("夏天", "炎热"),
       AUTUMN("秋天", "凉爽"),
       WINTER("冬天", "寒冷"); // 分号
   
       // 自定义字段
       private final String chineseName;
       private final String desc;
   
       // 构造方法（默认private，不能声明public）
       Season(String chineseName, String desc) {
           this.chineseName = chineseName;
           this.desc = desc;
       }
   
       // 自定义方法
       public String getChineseName() {
           return chineseName;
       }
   
       public String getDesc() {
           return desc;
       }
   
       // 重写toString()
       @Override
       public String toString() {
           return chineseName + "(" + desc + ")";
       }
   }
   
   // 测试
   public class EnumTest {
       public static void main(String[] args) {
           Season spring = Season.SPRING;
           System.out.println(spring.getChineseName()); // 输出：春天
           System.out.println(spring); // 输出：春天(温暖)
       }
   }
   ```

4. **实现接口**

   枚举类可以实现接口，所有枚举常量会默认实现接口方法，也可让每个常量单独实现（通过匿名内部类）。

   #### 示例：枚举实现接口

   ```java
   interface Behavior {
       void show();
   }
   
   enum Season implements Behavior {
       SPRING {
           @Override
           public void show() {
               System.out.println("春天来了");
           }
       }, /// 这里是，
       SUMMER {
           @Override
           public void show() {
               System.out.println("夏天到了");
           }
       };
   }
   
   // 测试
   public class EnumTest {
       public static void main(String[] args) {
           Season.SPRING.show(); // 输出：春天来了
       }
   }
   ```

### 三、枚举类的应用场景

1. **定义固定常量集合**：如季节、星期、状态码（成功 / 失败）等。

2. **枚举单例模式**：利用枚举的特性实现线程安全的单例（推荐方式）。

   ```java
   enum Singleton {
       INSTANCE;
       public void doSomething() {
           System.out.println("单例操作");
       }
   }
   
   // 使用
   Singleton.INSTANCE.doSomething();
   ```

3. **switch 语句**：枚举常量可用于 switch 表达式，增强代码可读性。

   ```java
   Season season = Season.SUMMER;
   switch (season) {
       case SPRING:
           System.out.println("春游");
           break;
       case SUMMER:
           System.out.println("游泳");
           break;
       // ...
   }
   ```

### 四、枚举类与普通类的区别

- 枚举类默认继承`Enum`，不能继承其他类（但可实现接口）。
- 构造方法默认`private`，无法创建枚举类的实例（只能通过定义的常量）。
- 所有枚举常量都是`public static final`的实例。

### 总结

Java 枚举类是一种强类型的常量定义方式，

---

## 🦴 常用类

在 Java 中，`String`、`StringBuilder`、`StringJoiner`是处理字符串的常用类，它们各有特性和适用场景，下面分别详细解释：

### 1. String 类

`String`类用于表示不可变的字符串（字符串常量），一旦创建，其内容就无法修改。

#### 核心特性

- **不可变性**：字符串对象创建后，字符序列不可改变。任何看似修改字符串的操作（如`concat()`、`substring()`），**实际上都会创建新的`String`对象。**
- **常量池优化**：字符串常量会被存储在 JVM 的字符串常量池中，相同内容的字符串会复用常量池中的对象，节省内存。
- **线程安全**：由于不可变，多线程环境下访问`String`对象不会出现线程安全问题。

#### 常用方法

- `length()`：返回字符串长度。
- `charAt(int index)`：获取指定索引的字符。
- `equals(Object obj)`：比较字符串内容是否相等（区分大小写）。
- `equalsIgnoreCase(String anotherString)`：忽略大小写比较内容。
- `concat(String str)`：拼接字符串（创建新对象）。
- `substring(int beginIndex)`：截取子串。
- `indexOf(String str)`：查找子串首次出现的索引。
- `replace(char oldChar, char newChar)`：替换字符（创建新对象）。

#### 示例

```java
String s = "Hello";
s = s.concat(" World"); // 创建新的String对象，原对象"Hello"不变，因为没有被引用被JVM回收了
// s不再指向原来的"Hello"对象，而是指向新创建的"Hello World"对象。
System.out.println(s); // 输出：Hello World
```

#### 适用场景

- 字符串内容不频繁修改的场景（如常量定义、少量拼接）。
- 多线程环境下的字符串操作。

### 2. StringBuilder 类

`StringBuilder`是可变的字符序列，用于高效处理字符串的动态拼接、修改等操作，避免频繁创建新对象。

#### 核心特性

- **可变性**：底层通过**字符数组**存储数据，支持动态扩容，修改操作直接在原对象上进行，不创建新对象。
- **非线程安全**：方法未加同步锁，单线程环境下效率高。
- **效率高**：相比`String`的拼接（`+`运算符），`StringBuilder`避免了大量临时对象的创建，性能更优。

#### 常用方法

- `append(...)`：拼接字符串（支持各种数据类型）。
- `insert(int offset, ...)`：在指定位置插入内容。
- `delete(int start, int end)`：删除指定区间的字符。
- `replace(int start, int end, String str)`：替换指定区间的字符。
- `reverse()`：反转字符串。
- `toString()`：转换为`String`对象。

#### 示例

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // 直接修改原对象，无新对象创建
sb.insert(5, ",");   // 插入逗号
System.out.println(sb.toString()); // 输出：Hello, World
```

#### 适用场景

- 单线程环境下频繁修改字符串（如循环拼接、动态生成字符串）。

### 3. StringJoiner 类

`StringJoiner`是 Java 8 新增的类，用于便捷地拼接字符串并指定分隔符，还支持前缀和后缀，简化了分隔符拼接的场景。

#### 核心特性

- **分隔符支持**：创建时指定分隔符，拼接元素时自动添加分隔符，无需手动处理。
- **前缀 / 后缀**：可指定拼接结果的前缀和后缀。
- **底层依赖`StringBuilder`**：内部通过`StringBuilder`实现拼接，保证效率。

#### 常用方法

- `StringJoiner(CharSequence delimiter)`：构造方法，指定分隔符。
- `StringJoiner(CharSequence delimiter, CharSequence prefix, CharSequence suffix)`：指定分隔符、前缀、后缀。
- `add(CharSequence newElement)`：添加拼接元素。
- `merge(StringJoiner other)`：合并另一个`StringJoiner`。
- `toString()`：返回拼接后的字符串。

#### 示例

```java
// 指定分隔符为逗号，前缀为"["，后缀为"]"
StringJoiner sj = new StringJoiner(", ", "[", "]");
sj.add("Apple").add("Banana").add("Orange");
System.out.println(sj.toString()); // 输出：[Apple, Banana, Orange]
```

#### 适用场景

- 需要按固定分隔符拼接多个元素（如集合转字符串、CSV 格式生成）。
- 需要添加统一前缀 / 后缀的拼接场景。

### 三者对比

| 特性     | String             | StringBuilder   | StringJoiner       |
| -------- | ------------------ | --------------- | ------------------ |
| 可变性   | 不可变             | 可变            | 可变（底层 SB）    |
| 线程安全 | 是                 | 否              | 否                 |
| 适用操作 | 少量修改 / 查询    | 频繁修改 / 拼接 | 分隔符拼接         |
| 效率     | 低（频繁创建对象） | 高              | 高（简化拼接逻辑） |

### 总结

- **String**：适合字符串内容固定的场景，利用常量池优化内存。
- **StringBuilder**：适合单线程下频繁修改字符串的场景，追求效率。
- **StringJoiner**：适合按分隔符拼接元素的场景，代码更简洁。

实际开发中，优先根据操作类型选择：简单拼接用`+`（编译期优化为`StringBuilder`），循环拼接用`StringBuilder`，分隔符拼接用`StringJoiner`（或`String.join()`）。

---

## 🐛 内部类

在 Java 中，**内部类（Inner Class）** 是定义在另一个类（外部类）内部的类。它是 Java 面向对象编程的重要特性之一，主要用于实现代码的封装、提高代码的内聚性，以及解决 Java 单继承的限制。

### 内部类的分类

Java 中的内部类主要分为以下四类：

#### 1. 成员内部类（Member Inner Class）

- 定义在外部类的成员位置（与成员变量、方法同级），**没有`static`修饰。**
- 可以访问外部类的所有成员（包括私有成员），但外部类访问内部类成员需要通过内部类实例。
- **创建内部类实例时，必须先创建外部类实例。**

```java
public class OuterClass {
    private String outerField = "外部类字段";

    // 成员内部类
    public class InnerClass {
        public void show() {
            System.out.println("访问外部类字段：" + outerField);
        }
    }

    public static void main(String[] args) {
        // 创建外部类实例
        OuterClass outer = new OuterClass();
        // 创建内部类实例
        OuterClass.InnerClass inner = outer.new InnerClass();
        inner.show(); // 输出：访问外部类字段：外部类字段
    }
}
```

#### 2. 静态内部类（Static Inner Class）

- 用`static`修饰的成员内部类，也称为 “嵌套类”。
- 只能访问外部类的静态成员（静态变量、静态方法），不能访问非静态成员。
- 创建实例时不需要外部类实例，可以直接通过`外部类名.内部类名`创建。

```java
public class OuterClass {
    private static String staticOuterField = "外部类静态字段";

    // 静态内部类
    public static class StaticInnerClass {
        public void show() {
            System.out.println("访问外部类静态字段：" + staticOuterField);
        }
    }

    public static void main(String[] args) {
        // 直接创建静态内部类实例
        OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
        inner.show(); // 输出：访问外部类静态字段：外部类静态字段
    }
}
```

#### 3. 局部内部类（Local Inner Class）

- 定义在方法、代码块或构造器内部的类，**作用域仅限于当前方法或代码块。**
- 不能使用`public`、`private`、`static`等修饰符。
- 可以访问外部类的成员，但访问方法内的局部变量时，该变量必须是`final`或**effectively final**（Java 8+）。

```java
public class OuterClass {
    private String outerField = "外部类字段";

    public void outerMethod() { // -------- 
        final String localVar = "方法内局部变量";

        // 局部内部类
        class LocalInnerClass {
            public void show() {
                System.out.println("访问外部类字段：" + outerField);
                System.out.println("访问方法内局部变量：" + localVar);
            }
        }

        // 创建局部内部类实例并调用方法
        LocalInnerClass inner = new LocalInnerClass();
        inner.show();
    } // --------

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.outerMethod();
        // 输出：
        // 访问外部类字段：外部类字段
        // 访问方法内局部变量：方法内局部变量
    }
}
```

#### 4. 匿名内部类（Anonymous Inner Class）

- 没有类名的局部内部类，通常用于创建一个类的实例并同时实现其方法（常用于接口或抽象类的快速实现）。
- 必须继承一个父类或实现一个接口，且只能创建一个实例。
- 语法上通过`new 父类/接口() { ... }`定义。

**示例（实现接口）：**

```java
interface Greeting {
    void sayHello();
}

public class OuterClass {
    public static void main(String[] args) {
        // 匿名内部类实现Greeting接口
        Greeting greeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello from Anonymous Inner Class!");
            }
        };
        greeting.sayHello(); // 输出：Hello from Anonymous Inner Class!
    }
}
```

**示例（继承类）：**

```java
abstract class Animal {
    abstract void makeSound();
}

public class OuterClass {
    public static void main(String[] args) {
        // 匿名内部类继承Animal类
        Animal cat = new Animal() {
            @Override
            void makeSound() {
                System.out.println("Meow!");
            }
        };
        cat.makeSound(); // 输出：Meow!
    }
}
```

### 内部类的特点

1. **封装性**：内部类可以访问外部类的私有成员，而外部类外部无法直接访问内部类，增强了代码的封装性。
2. **代码组织**：将逻辑上相关的类放在一起，使代码更紧凑、可读性更高。
3. **解决单继承限制**：通过内部类可以间接实现 “多继承”（例如，一个类的多个内部类分别继承不同的类）。
4. **匿名内部类简化代码**：无需显式定义类，直接创建实例并实现方法，适合快速实现简单的接口或抽象类。

### 注意事项

- 成员内部类不能包含静态成员（除非是`static final`常量）。
- 局部内部类和匿名内部类访问局部变量时，变量需为`final`或 effectively final（Java 8 + 自动推断）。
- 内部类编译后会生成独立的字节码文件，命名格式为`外部类名$内部类名.class`（匿名内部类为`外部类名$数字.class`）。

内部类在 Java GUI 编程（如 Swing 事件处理）、集合框架（如迭代器实现）等场景中广泛使用，是 Java 中灵活且强大的特性。

---

## 🍉 包装类

Java 包装类（Wrapper Class）是为了**让基本数据类型具备对象特性而设计的类**，每个基本数据类型都有对应的包装类。它们主要用于需要对象的场景（如集合框架、泛型等），同时提供了丰富的方法来操作基本数据类型。

### 基本数据类型与包装类的对应关系

| 基本数据类型 | 包装类（java.lang 包） |
| ------------ | ---------------------- |
| byte         | Byte                   |
| short        | Short                  |
| int          | Integer                |
| long         | Long                   |
| float        | Float                  |
| double       | Double                 |
| char         | Character              |
| boolean      | Boolean                |

### 包装类的主要特性

#### 1. **不可变性**

包装类的对象一旦创建，其值不能被修改（类似 String 类）。例如：

```java
Integer a = new Integer(10);
a = 20; // 实际上是创建 （指向）了新的 Integer 对象，原对象未被修改
```

#### 2. **自动装箱与拆箱**

- **装箱（Boxing）**：基本数据类型自动转换为包装类对象。
- **拆箱（Unboxing）**：包装类对象自动转换为基本数据类型。

这是 Java 5 引入的特性，简化了代码：

```java
// 自动装箱：int → Integer
Integer num = 10;  // => Integer num = Integer.valueOf(10);

// 自动拆箱：Integer → int
int n = num; // => int n = num.intValue();
```

#### 3. **常量与静态方法**

包装类提供了常用常量和工具方法：

- **常量**：如 `Integer.MAX_VALUE`（int 最大值）、`Double.NaN`（非数字）。
- **类型转换**：如 `Integer.parseInt("123")`（字符串转 int）、`Double.valueOf("3.14")`（字符串转 Double）。
- **进制转换**：如 `Integer.toBinaryString(10)`（十进制转二进制）。

#### 4. **对象.equals () 与 ==**

- **==**：比较对象地址（或基本数据类型的值）。
- **equals()**：比较包装类对象的**值**（需注意 `null` 避免空指针）。

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true（缓存池范围）

Integer c = 128;
Integer d = 128;
System.out.println(c == d); // false（超出缓存池）
System.out.println(c.equals(d)); // true（比较值）
```

### 包装类的缓存机制

Byte、Short、Integer（-128~127）、Long、Character（0~127）等包装类会缓存常用值，避免频繁创建对象。超出范围则会新建对象。

### 常用场景

1. **集合框架**：集合（如 `ArrayList`）只能存储对象，需用包装类。

   ```java
   List<Integer> list = new ArrayList<>();
   list.add(10); // 自动装箱
   ```

   

2. **泛型**：泛型不支持基本数据类型，需用包装类。

   ```java
   Map<String, Double> map = new HashMap<>();
   ```

   

3. **方法参数 / 返回值**：需要接收 `null` 时，用包装类（基本类型不能为 `null`）。

### 总结

包装类是基本数据类型的 “对象化” 扩展，解决了基本类型无法参与面向对象操作的问题，同时提供了丰富的工具方法。自动装箱 / 拆箱简化了代码，但需注意缓存机制和 `==` 与 `equals()` 的区别。

---

## 🍎 常用API

### 1. Math 类

`java.lang.Math` 是包含基本数学运算方法的工具类，所有方法均为静态方法，无需实例化。

**核心方法**：

- `abs()`：绝对值（支持 int、long、float、double）
- `max()`/`min()`：最大值 / 最小值
- `pow(a, b)`：a 的 b 次方
- `sqrt()`：平方根
- `random()`：生成`[0.0, 1.0)`的随机 double 值
- `round()`：四舍五入（float→int，double→long）
- `sin()`/`cos()`/`tan()`：三角函数（参数为弧度）

**示例**：

```java
Math.abs(-5); // 5
Math.pow(2, 3); // 8.0
Math.random(); // 随机数如0.345
```

### 2. System 类

`java.lang.System` 提供系统级操作，所有属性和方法均为静态。

**核心功能**：

- **输入输出**：`System.out`（标准输出）、`System.in`（标准输入）、`System.err`（错误输出）
- **系统信息**：`System.getProperty("os.name")`（获取操作系统名称）、`System.getenv()`（环境变量）
- **数组拷贝**：`System.arraycopy(src, srcPos, dest, destPos, length)`
- **时间相关**：`System.currentTimeMillis()`（当前时间戳，毫秒）、`System.nanoTime()`（纳秒级计时）
- **退出程序**：`System.exit(int status)`（status=0 表示正常退出）

**示例**：

```java
long now = System.currentTimeMillis(); // 当前时间戳
int[] a = {1,2,3};
int[] b = new int[3];
System.arraycopy(a, 0, b, 0, 3); // 数组拷贝
// 从源数组 a 的索引 0 开始，拷贝 3 个元素（即 1,2,3），放到目标数组 b 的索引 0 开始的位置，
```

### 3. Runtime 类

`java.lang.Runtime` 代表 Java 程序的运行时环境，每个 JVM 进程只有一个实例（单例），通过`Runtime.getRuntime()`获取。

**核心方法**：

- `exec()`：执行系统命令（如启动外部程序）
- `availableProcessors()`：获取 CPU 核心数
- `maxMemory()`：JVM 最大可用内存（字节）
- `totalMemory()`：JVM 当前总内存
- `freeMemory()`：JVM 空闲内存
- `gc()`：建议 JVM 执行垃圾回收（仅建议，不保证立即执行）

**示例**：

```java
Runtime runtime = Runtime.getRuntime();
System.out.println("CPU核心数：" + runtime.availableProcessors());
runtime.exec("notepad.exe"); // 打开记事本（Windows）
```

### 4. Object 类

`java.lang.Object` 是所有 Java 类的根类（默认父类），提供通用方法，所有类均可重写。

**核心方法**：

- `equals(Object obj)`：比较对象是否相等（**默认比较地址，建议重写为值比较**）
- `hashCode()`：返回对象哈希码（需与`equals`一致重写）
- `toString()`：返回对象字符串表示（默认格式：类名 @哈希码，建议重写）
- `getClass()`：返回对象的运行时类（`Class`对象）
- `clone()`：创建对象副本（需实现`Cloneable`接口）
- `wait()`/`notify()`/`notifyAll()`：线程通信（需在同步代码块中使用）
- `finalize()`：对象被 GC 回收前调用（已过时，推荐使用`Cleaner`）

**示例**：

```java
class Person {
    String name;
    @Override
    public String toString() {
        return "Person{name='" + name + "'}";
    }
}
Person p = new Person();
p.name = "Alice";
System.out.println(p); // 输出：Person{name='Alice'}
```

### 5. BigInteger 类

`java.math.BigInteger` 用于表示任意大小的整数（突破基本类型`long`的限制），支持高精度运算。

**核心方法**：

- 构造：`new BigInteger(String val)`（需传入数字字符串）
- 运算：`add()`/`subtract()`/`multiply()`/`divide()`（加减乘除）
- 比较：`compareTo()`（返回 - 1/0/1 表示小于 / 等于 / 大于）
- 其他：`mod()`（取模）、`pow()`（幂运算）、`gcd()`（最大公约数）

**示例**：

```java
BigInteger a = new BigInteger("12345678901234567890");
BigInteger b = new BigInteger("98765432109876543210");
BigInteger sum = a.add(b); // 大数相加
```

### 6. BigDecimal 类

`java.math.BigDecimal` 用于高精度十进制运算（解决浮点数`double`/`float`的精度丢失问题，如金融计算）。

**核心要点**：

- 构造：推荐使用`new BigDecimal(String val)`（避免`double`构造的精度问题）
- 运算：`add()`/`subtract()`/`multiply()`/`divide()`（需指定舍入模式）
- 舍入模式：`RoundingMode.HALF_UP`（四舍五入）、`RoundingMode.DOWN`（截断）等
- 精度控制：`setScale(int scale, RoundingMode mode)`（设置小数位数和舍入方式）

**示例**：

```java
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
BigDecimal sum = a.add(b); // 0.3（无精度丢失）
BigDecimal div = a.divide(b, 2, RoundingMode.HALF_UP); // 0.50（保留2位小数）
```

### 总结

| 类         | 核心用途       | 关键特性                      |
| ---------- | -------------- | ----------------------------- |
| Math       | 基本数学运算   | 静态方法，支持基础计算        |
| System     | 系统级操作     | 静态方法，输入输出 / 系统信息 |
| Runtime    | JVM 运行时环境 | 单例，进程 / 内存管理         |
| Object     | 所有类的父类   | 通用方法（equals/toString）   |
| BigInteger | 超大整数运算   | 无精度限制，支持大数运算      |
| BigDecimal | 高精度小数运算 | 避免精度丢失，适用于金融计算  |

这些类覆盖了 Java 中基础工具、系统交互、高精度计算等核心场景，是开发中高频使用的 API。

---

