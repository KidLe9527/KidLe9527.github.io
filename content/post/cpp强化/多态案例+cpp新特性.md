> 案例描述:
> 电脑主要组成部件为CPU(用于计算)，显卡(用于显示)，内存条（用于存储),将每个零件封装出抽象基类，并且提供不同的厂商生产不同的零件，例如Intel厂商和Lenovo厂商创建电脑类提供让电脑工作的函数，并且调用每个零件工作的接口,测试时组装三台不同的电脑进行工作.

```cpp
#include<iostream>
#include<memory>  // --- 修改 --- 引入智能指针头文件
using namespace std;

// --- 修改 --- 新增品牌接口
class Brand {
public:
    virtual string getBrand() = 0;
};

// 抽象零件类（合并简化）
class CPU : public Brand {  // --- 修改 --- 继承Brand
public:
    virtual void calculate() = 0;
};

class VideoCard : public Brand {  // --- 修改 --- 继承Brand
public:
    virtual void display() = 0;
};

class Memory : public Brand {  // --- 修改 --- 继承Brand
public:
    virtual void storage() = 0;
};

// 电脑类（简化内存管理）
class Computer {
public:
    // --- 修改 --- 使用智能指针
    Computer(shared_ptr<CPU> cpu, shared_ptr<VideoCard> vc, shared_ptr<Memory> mem) 
        : m_cpu(cpu), m_vc(vc), m_mem(mem) {}

    void work() {
        cout << "【品牌信息】CPU:" << m_cpu->getBrand() 
             << " 显卡:" << m_vc->getBrand() 
             << " 内存:" << m_mem->getBrand() << endl;  // --- 修改 --- 输出品牌
        m_cpu->calculate();
        m_vc->display();
        m_mem->storage();
    }

    // --- 修改 --- 移除手动析构（智能指针自动管理）

private:
    shared_ptr<CPU> m_cpu;     // --- 修改 --- 智能指针
    shared_ptr<VideoCard> m_vc;
    shared_ptr<Memory> m_mem;
};

// 具体零件实现（简化重复代码）
class IntelCPU : public CPU {
public:
    string getBrand() override { return "Intel"; }  // --- 修改 --- 实现品牌接口
    void calculate() override { cout << "Intel CPU 计算中..." << endl; } // override, C++11特性,是对基类虚函数的重写
};

class IntelVideoCard : public VideoCard {
public:
    string getBrand() override { return "Intel"; }
    void display() override { cout << "Intel 显卡 渲染中..." << endl; }
};

class IntelMemory : public Memory {
public:
    string getBrand() override { return "Intel"; }
    void storage() override { cout << "Intel 内存 存储中..." << endl; }
};

class LenovoCPU : public CPU {
public:
    string getBrand() override { return "Lenovo"; }
    void calculate() override { cout << "Lenovo CPU 计算中..." << endl; }
};

class LenovoVideoCard : public VideoCard {
public:
    string getBrand() override { return "Lenovo"; }
    void display() override { cout << "Lenovo 显卡 渲染中..." << endl; }
};

class LenovoMemory : public Memory {
public:
    string getBrand() override { return "Lenovo"; }
    void storage() override { cout << "Lenovo 内存 存储中..." << endl; }
};

// 测试函数（简化组装逻辑）
void test01() {
    // --- 修改 --- 使用智能指针组装电脑
    auto computer1 = make_shared<Computer>(
        make_shared<IntelCPU>(), 
        make_shared<IntelVideoCard>(), 
        make_shared<IntelMemory>()
    );
    computer1->work();
    cout << "------分割线------" << endl;

    auto computer2 = make_shared<Computer>(
        make_shared<LenovoCPU>(), 
        make_shared<IntelVideoCard>(), 
        make_shared<LenovoMemory>()
    );
    computer2->work();
}

int main() {
    test01();
    return 0;
}
```

