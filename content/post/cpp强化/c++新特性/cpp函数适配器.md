## 函数适配器

在 C++ 中，**函数对象适配器（Function Object Adapters）** 是用于修改、组合或扩展现有函数对象（包括内建函数对象、自定义仿函数、普通函数、Lambda 等）行为的工具，它们封装在 `<functional>` 头文件中，是实现函数式编程的重要组件。

适配器的核心作用是**适配现有可调用对象**，使其满足特定接口要求（如参数数量、参数类型、返回值类型等），或通过组合实现更复杂的逻辑。

### 一、常见的函数对象适配器分类

#### 1. **绑定适配器（Binding Adapters）**

用于将函数对象的参数绑定到固定值，或调整参数顺序，主要包括：

- `std::bind`（C++11 起）：通用绑定器，可绑定参数到固定值或重排参数顺序。
- 旧式适配器（C++98，已被 `std::bind` 取代）：`std::bind1st`（绑定第一个参数）、`std::bind2nd`（绑定第二个参数）。

**示例**：

```cpp
#include <iostream>
#include <functional>
using namespace std;

int add(int a, int b) { return a + b; }

int main() {
    // 绑定 add 的第一个参数为 10，生成新的函数对象（只接受一个参数）
    auto add10 = bind(add, 10, placeholders::_1);
    cout << add10(5) << endl;  // 10 + 5 = 15

    // 绑定内建函数对象 plus<int> 的第二个参数为 20
    auto add20 = bind(plus<int>(), placeholders::_1, 20);
    cout << add20(3) << endl;  // 3 + 20 = 23
    return 0;
}
```

- `placeholders::_1` 是占位符，表示调用时传入的第一个参数。

  - **参数绑定列表**：用 `std::placeholders::_n`（占位符）表示调用时传入的第 `n` 个参数，或直接绑定固定值。

    ```cpp
    int add(int a, int b) { return a + b; }
    // 交换参数顺序（_2是第二个调用参数，_1是第一个）
    auto swap_add = bind(add, placeholders::_2, placeholders::_1);
    cout << swap_add(5, 10) << endl;  // 10+5=15
    ```


#### 2. **否定适配器（Negation Adapters）**

用于对函数对象的返回值取反，主要包括：

- `std::not1`（C++98）：对**一元函数对象**取反（要求函数对象定义 `argument_type` 别名）。
- `std::not2`（C++98）：对**二元函数对象**取反（要求函数对象定义 `first_argument_type`/`second_argument_type` 别名）。
- C++11 后可结合 `std::not_fn`（更通用）。

**示例**：

```cpp
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    // 查找第一个不大于 3 的元素（!greater<int>(x,3)）
    auto it = find_if(v.begin(), v.end(), not1(bind(greater<int>(), placeholders::_1, 3)));
    cout << *it << endl;  // 输出 1

    // C++17 std::not_fn 更简洁
    auto it2 = find_if(v.begin(), v.end(), not_fn(bind(greater<int>(), placeholders::_1, 3)));
    return 0;
}
```

#### 3. **函数指针适配器（Function Pointer Adapters）**

用于将普通函数指针转换为函数对象（C++11 前需要，C++11 后可直接使用函数指针）：

- `std::ptr_fun`（C++98，C++17 弃用）：将函数指针适配为函数对象，使其能与其他适配器结合。

**示例**：

```cpp
#include <iostream>
#include <functional>
#include <algorithm>
#include <vector>
using namespace std;

bool is_even(int x) { return x % 2 == 0; }

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    // C++98：用 ptr_fun 适配函数指针，结合 not1
    auto it = find_if(v.begin(), v.end(), not1(ptr_fun(is_even)));
    cout << *it << endl;  // 输出 1（第一个奇数）

    // C++11 后可直接用函数指针，无需 ptr_fun
    auto it2 = find_if(v.begin(), v.end(), not_fn(is_even));
    return 0;
}
```

#### 4. **成员函数适配器（Member Function Adapters）**

用于将类的成员函数适配为函数对象，主要包括：

- `std::mem_fn`（C++11 起）：适配成员函数指针，支持对象、指针、智能指针调用。
- 旧式适配器：`std::mem_fun`（适配成员函数指针，用于对象指针）、`std::mem_fun_ref`（适配成员函数指针，用于对象引用）。

**示例**：

```cpp
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>
using namespace std;

struct Person {
    string name;
    void print() const { cout << name << endl; }
};

int main() {
    vector<Person> people = {{"Alice"}, {"Bob"}, {"Charlie"}};
    // 用 mem_fn 适配成员函数 print，遍历调用
    for_each(people.begin(), people.end(), mem_fn(&Person::print));//实体，或者说是引用

    vector<Person*> ptrs = {new Person{"Dave"}, new Person{"Eve"}};
    for_each(ptrs.begin(), ptrs.end(), mem_fn(&Person::print));  // 支持指针
    return 0;
}
```

### 二、适配器的核心价值

1. **接口适配**：将不符合算法要求的可调用对象（如参数数量不符、返回值不符）转换为符合要求的形式。
2. **逻辑组合**：通过绑定、否定等操作，组合现有函数对象实现复杂逻辑（如 `not1(bind(greater<int>(), _1, 3))` 表示 “不大于 3”）。
3. **兼容性**：旧式适配器（如 `bind1st`、`ptr_fun`）保证了对早期 C++ 代码的兼容，新式适配器（如 `std::bind`、`std::not_fn`）更灵活通用。

### 三、C++11 后的变化

C++11 引入 Lambda 表达式和更通用的适配器（如 `std::bind`、`std::not_fn`）后，旧式适配器（`bind1st`、`ptr_fun`、`mem_fun` 等）逐渐被取代，因为 Lambda 更简洁直观：

```cpp
// 用 Lambda 替代适配器组合
auto it = find_if(v.begin(), v.end(), [](int x) { return !(x > 3); });
```

### 总结

函数对象适配器是 C++ 中连接函数对象、普通函数、成员函数与标准算法的 “桥梁”，通过修改或组合可调用对象的行为，满足泛型算法的接口要求。虽然 C++11 后 Lambda 表达式在很多场景下更易用，但适配器在处理复杂函数组合或兼容旧代码时仍有其价值。

---



#### 补充

> C++ 中的函数对象适配器用于修改、组合现有可调用对象（函数对象、普通函数、成员函数等）的行为，使其适配算法的接口要求。以下是常用适配器的语法、作用及示例：
>
> ### 一、绑定适配器：`std::bind`（C++11 起）
>
> #### 语法：
>
> ```cpp
> std::bind(可调用对象, 参数绑定列表);
> ```
>
> - **参数绑定列表**：用 `std::placeholders::_n`（占位符）表示调用时传入的第 `n` 个参数，或直接绑定固定值。
> - **作用**：绑定参数到固定值、调整参数顺序，将多元函数转为一元 / 少元函数。
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> using namespace std;
> 
> int add(int a, int b) { return a + b; }
> 
> int main() {
>     // 绑定第一个参数为10，第二个参数为占位符（调用时传入）
>     auto add10 = bind(add, 10, placeholders::_1);
>     cout << add10(5) << endl;  // 10+5=15
> 
>     // 交换参数顺序（_2是第二个调用参数，_1是第一个）
>     auto swap_add = bind(add, placeholders::_2, placeholders::_1);
>     cout << swap_add(5, 10) << endl;  // 10+5=15
>     return 0;
> }
> ```
>
> ### 二、旧式绑定适配器：`bind1st`/`bind2nd`（C++98，C++17 弃用）
>
> #### 语法：
>
> ```cpp
> std::bind1st(二元函数对象, 固定值);  // 绑定为第一个参数
> std::bind2nd(二元函数对象, 固定值);  // 绑定为第二个参数
> ```
>
> - **要求**：二元函数对象需继承 `std::binary_function<Arg1, Arg2, Result>`（定义参数类型别名）。
> - **作用**：将二元函数对象转为一元函数对象。
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> #include <vector>
> #include <algorithm>
> using namespace std;
> 
> // 自定义二元函数对象（继承binary_function）
> class MyAdd : public binary_function<int, int, void> {
> public:
>     void operator()(int a, int b) const {
>         cout << a + b << " ";
>     }
> };
> 
> int main() {
>     vector<int> v = {1,2,3};
>     // bind2nd：绑定第二个参数为100，容器元素作为第一个参数
>     for_each(v.begin(), v.end(), bind2nd(MyAdd(), 100));  // 101 102 103
>     return 0;
> }
> ```
>
> ### 三、取反适配器：`not1`/`not2`（C++98）
>
> #### 语法：
>
> ```cpp
> std::not1(一元函数对象);  // 对一元函数对象的返回值取反
> std::not2(二元函数对象);  // 对二元函数对象的返回值取反
> ```
>
> - **要求**：
>   - `not1`：一元函数对象需继承 `std::unary_function<Arg, Result>`；
>   - `not2`：二元函数对象需继承 `std::binary_function<Arg1, Arg2, Result>`。
> - **作用**：反转函数对象的布尔返回值。
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> #include <vector>
> #include <algorithm>
> using namespace std;
> 
> // 一元函数对象（继承unary_function）
> class GreaterThan5 : public unary_function<int, bool> {
> public:
>     bool operator()(int v) const { return v > 5; }
> };
> 
> int main() {
>     vector<int> v = {1,6,3,7};
>     // not1：取反，查找第一个≤5的元素
>     auto it = find_if(v.begin(), v.end(), not1(GreaterThan5()));
>     cout << *it << endl;  // 输出1
> 
>     // not2：对二元函数取反（less<int>()升序→降序）
>     sort(v.begin(), v.end(), not2(less<int>()));
>     for (int x : v) cout << x << " ";  // 7 6 3 1
>     return 0;
> }
> ```
>
> ### 四、函数指针适配器：`ptr_fun`（C++98，C++17 弃用）
>
> #### 语法：
>
> ```cpp
> std::ptr_fun(普通函数指针);
> ```
>
> - **作用**：将普通函数指针转为函数对象，使其能与其他适配器结合。
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> #include <vector>
> #include <algorithm>
> using namespace std;
> 
> bool isEven(int x) { return x % 2 == 0; }
> 
> int main() {
>     vector<int> v = {1,2,3,4};
>     // ptr_fun适配函数指针，not1取反→查找第一个奇数
>     auto it = find_if(v.begin(), v.end(), not1(ptr_fun(isEven)));
>     cout << *it << endl;  // 输出1
>     return 0;
> }
> ```
>
> ### 五、成员函数适配器：`mem_fn`（C++11 起）
>
> #### 语法：
>
> ```cpp
> std::mem_fn(类成员函数指针);
> ```
>
> - **作用**：适配类的成员函数指针，支持通过对象、指针、智能指针调用。
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> #include <vector>
> #include <algorithm>
> using namespace std;
> 
> struct Person {
>     string name;
>     void print() const { cout << name << endl; }
> };
> 
> int main() {
>     vector<Person> people = {{"Alice"}, {"Bob"}};
>     vector<Person*> ptrs = {&people[0], &people[1]};
> 
>     // 适配成员函数，遍历调用
>     for_each(people.begin(), people.end(), mem_fn(&Person::print)); // 对象调用
>     for_each(ptrs.begin(), ptrs.end(), mem_fn(&Person::print));     // 指针调用
>     return 0;
> }
> ```
>
> ### 六、旧式成员函数适配器：`mem_fun`/`mem_fun_ref`（C++98，C++17 弃用）
>
> #### 语法：
>
> ```cpp
> std::mem_fun(成员函数指针);      // 容器存对象指针时使用
> std::mem_fun_ref(成员函数指针);  // 容器存对象实体时使用
> ```
>
> #### 示例：
>
> ```cpp
> #include <iostream>
> #include <functional>
> #include <vector>
> #include <algorithm>
> using namespace std;
> 
> struct Person {
>     void show() { cout << "Person" << endl; }
> };
> 
> int main() {
>     vector<Person> v1 = {Person(), Person()};
>     vector<Person*> v2 = {&v1[0], &v1[1]};
> 
>     for_each(v1.begin(), v1.end(), mem_fun_ref(&Person::show)); // 实体
>     for_each(v2.begin(), v2.end(), mem_fun(&Person::show));     // 指针
>     return 0;
> }
> ```
>
> ### 总结
>
> | 适配器        | 作用                            | 适用场景             |
> | ------------- | ------------------------------- | -------------------- |
> | `bind`        | 绑定参数、调整顺序              | C++11+ 通用绑定      |
> | `bind1st/2nd` | 绑定二元函数的参数              | C++98 二元函数转一元 |
> | `not1/not2`   | 反转函数对象的布尔返回值        | 一元 / 二元函数取反  |
> | `ptr_fun`     | 普通函数指针转函数对象          | C++98 适配普通函数   |
> | `mem_fn`      | 适配成员函数指针                | C++11+ 成员函数适配  |
> | `mem_fun/ref` | 适配成员函数（区分指针 / 实体） | C++98 成员函数适配   |
>
> C++11 后推荐使用 **`bind`、`mem_fn` 和 Lambda 表达式**，旧式适配器（如 `bind1st`、`ptr_fun`）仅用于兼容旧代码。

---



#### 解释`mem_fun_ref(&Person::show)`

> 在 `for_each(v1.begin(), v1.end(), mem_fun_ref(&Person::show))` 中，`mem_fun_ref(&Person::show)` 是**成员函数适配器**，作用是将类的成员函数指针（`&Person::show`）适配成可被 `for_each` 调用的一元函数对象。以下是详细拆解：
>
> ### 1. `&Person::show`：成员函数指针
>
> `&Person::show` 是指向 `Person` 类的**非静态成员函数**的指针，其类型为 `void (Person::*)()`（表示 “属于 `Person` 类、无参数、返回 `void` 的成员函数指针”）。
>
> 成员函数指针与普通函数指针的区别：
>
> - 调用成员函数时，必须绑定到**具体的对象 / 指针**（因为成员函数隐含 `this` 指针）；
> - 普通函数指针可直接调用，成员函数指针需通过对象 / 指针间接调用。
>
> ### 2. `mem_fun_ref`：成员函数适配器（适配对象实体）
>
> `mem_fun_ref` 是 C++ 标准库中的适配器（定义在 `<functional>`），专门用于：
>
> - 将**成员函数指针**转换为**可调用的函数对象**；
> - 适配场景：当容器中存储的是**对象实体**（如 `vector<Person>`）时，遍历元素时会以**对象引用**的形式传递给适配器。
>
> 其核心作用是：🔥
>
> - 生成一个一元函数对象，该对象的 `operator()` 接收一个 `Person` 对象（或引用），并调用其 `show` 成员函数；
> - 自动处理 `this` 指针的绑定（将遍历到的对象作为 `this` 指向的实例）。
>
> ### 3. 适配后的函数对象如何被 `for_each` 调用
>
> `for_each` 遍历容器时，会对每个元素执行以下逻辑：
>
> ```cpp
> // 伪代码：for_each的内部逻辑
> for (auto it = v1.begin(); it != v1.end(); ++it) {
>     // 适配器生成的函数对象调用：传入当前元素（对象实体）
>     适配后的函数对象(*it); 
> }
> ```
>
> 而 `mem_fun_ref(&Person::show)` 生成的函数对象，其 `operator()` 等价于：
>
> ```cpp
> void operator()(Person& obj) const {
>     obj.show(); // 调用对象的show成员函数
> }
> ```
>
> 因此，`for_each` 遍历到每个 `Person` 对象时，会通过适配器自动调用该对象的 `show` 方法。
>
> ### 4. `mem_fun_ref` 与 `mem_fun` 的区别
>
> - **`mem_fun_ref`**：用于容器存储**对象实体**（如 `vector<Person>`），接收对象引用；
> - **`mem_fun`**：用于容器存储**对象指针**（如 `vector<Person*>`），接收对象指针。
>
> 例如：
>
> ```cpp
> vector<Person*> v2; // 存储指针
> for_each(v2.begin(), v2.end(), mem_fun(&Person::show)); 
> // mem_fun生成的函数对象：void operator()(Person* ptr) { ptr->show(); }
> ```
>
> ### 5. 为什么需要适配器？
>
> `for_each` 要求第三个参数是**一元可调用对象**（接收一个参数），但成员函数指针本身不是可调用对象（需要绑定 `this`）。`mem_fun_ref` 相当于 “包装器”，将成员函数指针转换成符合 `for_each` 要求的一元函数对象，实现 “遍历对象并调用其成员函数” 的逻辑。
>
> ### 总结
>
> `mem_fun_ref(&Person::show)` 的完整含义：
>
> 将 `Person` 类的 `show` 成员函数指针，适配成一个**一元函数对象**；当 `for_each` 遍历容器中的 `Person` 对象实体时，该函数对象会接收对象引用，并自动调用对象的 `show` 成员函数。
>
> 本质是解决 “成员函数指针无法直接作为算法参数” 的问题，实现成员函数与 STL 算法的适配。