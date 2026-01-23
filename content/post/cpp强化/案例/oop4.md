1. 编写一个学生和教师数据输入和显示程序。学生数据有编号、姓名、班号和成绩，教师数据有编号、姓名、职称和部门。要求将编号、姓名输入和显示设计成一个类Person，并作为学生类Student和教师类Teacher的基类。最终在主函数中进行测试。

```cpp
#include <iostream>
#include <string>

class Person {
protected: // 保护成员，允许派生类访问
    int id_;
    std::string name_;
public:
    Person(int id, const std::string& name) : id_(id), name_(name) {}

    virtual void input() {
        std::cout << "输入编号: ";
        std::cin >> id_;
        std::cout << "输入姓名: ";
        std::cin >> name_;
    }

    virtual void display() const {
        std::cout << "编号: " << id_ << ", 姓名: " << name_;
    }

    virtual ~Person() = default; // 虚析构函数,确保派生类对象通过基类指针删除时正确析构
};

class Student : public Person {
private:
    std::string class_no_;
    float score_;
public:
    Student(int id = 0, const std::string& name = "", const std::string& class_no = "", float score = 0.0f)
        : Person(id, name), class_no_(class_no), score_(score) {}
    
    void input() override {
        Person::input(); // 调用基类输入
        std::cout << "输入班号: ";
        std::cin >> class_no_;
        std::cout << "输入成绩: ";
        std::cin >> score_;
    }

    void display() const override {
        Person::display(); // 调用基类显示
        std::cout << ", 班号: " << class_no_ << ", 成绩: " << score_ << std::endl;
    }
};

class Teacher : public Person {
private:
    std::string title_;
    std::string department_;
public:
    Teacher(int id = 0, const std::string& name = "", const std::string& title = "", const std::string& department = "")
        : Person(id, name), title_(title), department_(department) {}
  
    void input() override {
        Person::input(); // 调用基类输入
        std::cout << "输入职称: ";
        std::cin >> title_;
        std::cout << "输入部门: ";
        std::cin >> department_;
    }

    void display() const override {
        Person::display(); // 调用基类显示
        std::cout << ", 职称: " << title_ << ", 部门: " << department_ << std::endl;
    }
};

int main() {
    Student student;
    Teacher teacher;

    std::cout << "请输入学生信息:" << std::endl;
    student.input();
    std::cout << "请输入教师信息:" << std::endl;
    teacher.input();

    std::cout << "\n学生信息:" << std::endl;
    student.display();
    std::cout << "教师信息:" << std::endl;
    teacher.display();

    return 0;
}
```

2. 分别定义Teacher（教师）类和Cadre（干部）类，采用多继承方式由这两个类派生出新类Teacher_Cadre（教师兼干部）。最终在主函数中进行测试。要求：

（1）在两个基类中都包含姓名、年龄、性别、地址、电话等数据成员。

（2）在Teacher类中还包含数据成员titile（职称），在Cadre类中还包含数据成员post（职务），在Teacher_Cadre类中还包含数据成员wages（工资）。

（3）对两个基类中的姓名、年龄、性别、地址、电话等数据成员用相同的名字，在引用这些数据成员时，指定作用域。

（4）在类体中声明成员函数，在类外定义成员函数。

（5）在派生类Teacher_Cadre的成员函数show中调用Teacher类中的display函数，输出姓名、年龄、性别、职称、地址、电话，然后再用cout语句输出职务与工资。

```cpp
#include <iostream>
#include <string>

class Teacher {
protected:
    std::string name_;
    int age_;
    std::string gender_;
    std::string address_;
    std::string phone_;
    std::string title_; // 职称
public:
    void display() const;
    void input();
};

class Cadre {
protected:
    std::string name_;
    int age_;
    std::string gender_;
    std::string address_;
    std::string phone_;
    std::string post_; // 职务
public:
    void display() const;
    void input();
};

class Teacher_Cadre : public Teacher, public Cadre {
private:
    double wages_; // 工资
public:
    void show() const;
    void input();
};

void Teacher::input() {
    std::cout << "教师信息输入：" << std::endl;
    std::cout << "输入教师姓名: ";
    std::cin >> name_;
    std::cout << "输入教师年龄: ";
    std::cin >> age_;
    std::cout << "输入教师性别: ";
    std::cin >> gender_;
    std::cout << "输入教师地址: ";
    std::cin >> address_;
    std::cout << "输入教师电话: ";
    std::cin >> phone_;
    std::cout << "输入教师职称: ";
    std::cin >> title_;
}

void Teacher::display() const {
    std::cout << "教师信息：" << std::endl;
    std::cout << "姓名: " << name_ << ", 年龄: " << age_ << ", 性别: " << gender_ << std::endl;
    std::cout << "地址: " << address_ << ", 电话: " << phone_ << ", 职称: " << title_ << std::endl;
}

void Cadre::input() {
    std::cout << "干部信息输入：" << std::endl;
    std::cout << "输入干部姓名: ";
    std::cin >> name_;
    std::cout << "输入干部年龄: ";
    std::cin >> age_;
    std::cout << "输入干部性别: ";
    std::cin >> gender_;
    std::cout << "输入干部地址: ";
    std::cin >> address_;
    std::cout << "输入干部电话: ";
    std::cin >> phone_;
    std::cout << "输入干部职务: ";
    std::cin >> post_;
}

void Cadre::display() const {
    std::cout << "干部信息：" << std::endl;
    std::cout << "姓名: " << name_ << ", 年龄: " << age_ << ", 性别: " << gender_ << std::endl;
    std::cout << "地址: " << address_ << ", 电话: " << phone_ << ", 职务: " << post_ << std::endl;
}

void Teacher_Cadre::input() {
    Teacher::input(); // 输入教师信息
    // Cadre::input();   // 输入干部信息
    std::cout << "输入职务: ";
    std::cin >> post_;
    std::cout << "输入工资: ";
    std::cin >> wages_;
}

void Teacher_Cadre::show() const {
    Teacher::display(); // 调用Teacher类的display函数
    std::cout << "职务: " << post_ << ", 工资: " << wages_ << std::endl;
}

int main() {
    Teacher_Cadre tc;
    tc.input();
    tc.show();
    return 0;
}   
```

3. 写一个程序，定义抽象基类Shape，
由它派生出5个派生类：Circle，Square，Rectangle，Trapezoid，Triangle。
用虚函数分别计算几种图形面积，并求它们的和。要求使用基类指针数组，使它的每一个元素指向一个派生类对象。
最终在主函数中进行测试。

```cpp
#include <iostream>
#include <cmath>
#include <vector>
const double PI = 3.1415926;

// 抽象基类 Shape
class Shape {
public:
    virtual double area() const = 0; // 纯虚函数，计算面积
    virtual ~Shape() = default; // 虚析构函数
};

// 圆形类 Circle
class Circle : public Shape {
private:
    double radius_;// 半径
public:
    Circle(double radius = 0) : radius_(radius) {}
    double area() const override {
        return PI * radius_ * radius_;
    }
};

// 正方形类 Square
class Square : public Shape {
private:
    double side_;// 边长
public:
    Square(double side = 0) : side_(side) {}
    double area() const override {
        return side_ * side_;
    }
};  

// 矩形类 Rectangle
class Rectangle : public Shape {
private:
    double length_;// 长
    double width_;// 宽
public:
    Rectangle(double length = 0, double width = 0) : length_(length), width_(width) {}
    double area() const override {
        return length_ * width_;
    }
};

// 梯形类 Trapezoid
class Trapezoid : public Shape {
private:
    double base1_;// 上底
    double base2_;// 下底
    double height_;// 高
public:
    Trapezoid(double base1 = 0, double base2 = 0, double height = 0) 
        : base1_(base1), base2_(base2), height_(height) {}
    double area() const override {
        return (base1_ + base2_) * height_ / 2.0;
    }
};  

// 三角形类 Triangle
class Triangle : public Shape {
private:
    double base_;// 底
    double height_;// 高
public:
    Triangle(double base = 0, double height = 0) : base_(base), height_(height) {}
    double area() const override {
        return base_ * height_ / 2.0;
    }
};  

int main() {
    // 创建基类指针数组，指向不同派生类对象
    std::vector<Shape*> shapes;
    shapes.push_back(new Circle(5.0));          // 半径5的圆
    shapes.push_back(new Square(4.0));          // 边长4的正方形
    shapes.push_back(new Rectangle(3.0, 6.0));  // 长3宽6的矩形
    shapes.push_back(new Trapezoid(4.0, 6.0, 5.0)); // 上底4下底6高5的梯形
    shapes.push_back(new Triangle(4.0, 5.0));   // 底4高5的三角形

    double total_area = 0.0;
    for (const auto& shape : shapes) {
        total_area += shape->area(); // 计算总面积
    }

    std::cout << "各种图形面积之和: " << total_area << std::endl;

    // 释放内存
    for (auto& shape : shapes) {
        delete shape;
    }
    return 0;
}   
```

