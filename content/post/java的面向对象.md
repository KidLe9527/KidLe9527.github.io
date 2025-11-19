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