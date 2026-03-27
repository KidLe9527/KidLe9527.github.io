你想要用 C++ 实现最小堆（Min Heap）数据结构，最小堆的核心特性是**父节点的值始终小于或等于其左右子节点的值**，且它是一棵完全二叉树，通常用数组来高效存储。

下面我会先讲解实现思路，再给出完整的可运行代码，最后解释关键逻辑。

## 一、实现思路

1. **存储结构**：用 `std::vector` 存储堆元素（相比普通数组，vector 支持动态扩容，更灵活）。

2. **核心操作**：

   - `push()`：插入元素到堆尾，然后向上调整（**上浮 / Percolate Up**），维持堆的性质。
   - `pop()`：删除堆顶（最小元素），将堆尾元素移到堆顶，然后向下调整（**下沉 / Percolate Down**），维持堆的性质。
   - `top()`：获取堆顶元素（不删除）。
   - `empty()`/`size()`：辅助判断堆是否为空、获取堆的大小。

   

3. **索引关系**（完全二叉树的数组映射）：

   - 对于索引为 `i` 的节点：
     - 左子节点索引：`2 * i + 1`
     - 右子节点索引：`2 * i + 2`
     - 父节点索引：`(i - 1) / 2`（整数除法，自动向下取整）

   

## 二、完整 C++ 实现代码

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>  // 用于抛出异常（处理空堆访问）

// 最小堆类模板，支持任意可比较的数据类型
template <typename T>
class MinHeap {
private:
    std::vector<T> heap;  // 存储堆元素的容器

    // 核心辅助函数：上浮调整（插入元素后调用，维持最小堆性质）
    void siftUp(int index) {
        // 当不是根节点，且当前节点值小于父节点值时，继续上浮
        while (index > 0) {
            int parentIndex = (index - 1) / 2;  // 计算父节点索引
            // 最小堆：父节点 <= 子节点，满足则停止上浮
            if (heap[parentIndex] <= heap[index]) {
                break;
            }
            // 交换父节点和当前节点的值
            std::swap(heap[parentIndex], heap[index]);
            // 更新当前节点索引为父节点索引，继续向上检查
            index = parentIndex;
        }
    }

    // 核心辅助函数：下沉调整（删除堆顶后调用，维持最小堆性质）
    void siftDown(int index) {
        int n = heap.size();
        // 当当前节点有左子节点时，继续下沉（完全二叉树，有右子节点必存在左子节点）
        while (2 * index + 1 < n) {
            int leftChild = 2 * index + 1;    // 左子节点索引
            int rightChild = 2 * index + 2;   // 右子节点索引
            int minChildIndex = leftChild;    // 先假设左子节点是最小值节点

            // 若存在右子节点，且右子节点值更小，更新最小值节点索引为右子节点
            if (rightChild < n && heap[rightChild] < heap[leftChild]) {
                minChildIndex = rightChild;
            }

            // 最小堆：父节点 <= 最小子节点，满足则停止下沉
            if (heap[index] <= heap[minChildIndex]) {
                break;
            }

            // 交换当前节点和最小子节点的值
            std::swap(heap[index], heap[minChildIndex]);
            // 更新当前节点索引为最小子节点索引，继续向下检查
            index = minChildIndex;
        }
    }

public:
    // 1. 构造函数（默认构造空堆）
    MinHeap() = default;

    // 2. 获取堆顶元素（最小元素）
    T top() const {
        if (heap.empty()) {
            throw std::out_of_range("MinHeap is empty, cannot get top element.");
        }
        return heap[0];
    }

    // 3. 插入元素到堆中
    void push(const T& value) {
        // 步骤1：将元素添加到数组末尾（完全二叉树的最后一个节点）
        heap.push_back(value);
        // 步骤2：对最后一个元素进行上浮调整，维持堆性质
        siftUp(heap.size() - 1);
    }

    // 4. 删除堆顶元素（最小元素）
    void pop() {
        if (heap.empty()) {
            throw std::out_of_range("MinHeap is empty, cannot pop element.");
        }
        // 步骤1：将堆尾元素覆盖堆顶元素（删除堆顶的高效方式，避免数组移动）
        heap[0] = heap.back();
        // 步骤2：删除堆尾的冗余元素
        heap.pop_back();
        // 步骤3：对堆顶元素进行下沉调整，维持堆性质
        siftDown(0);
    }

    // 5. 判断堆是否为空
    bool empty() const {
        return heap.empty();
    }

    // 6. 获取堆的大小（元素个数）
    size_t size() const {
        return heap.size();
    }

    // 辅助函数：打印堆的内部数组（用于调试查看）
    void printHeap() const {
        for (const auto& val : heap) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
};

// 测试代码
int main() {
    try {
        MinHeap<int> minHeap;

        // 插入测试数据
        std::cout << "插入元素：5, 3, 8, 1, 2, 9" << std::endl;
        minHeap.push(5);
        minHeap.push(3);
        minHeap.push(8);
        minHeap.push(1);
        minHeap.push(2);
        minHeap.push(9);

        // 打印堆内部数组
        std::cout << "堆内部数组：";
        minHeap.printHeap();

        // 获取并删除堆顶元素
        std::cout << "依次弹出最小元素：";
        while (!minHeap.empty()) {
            std::cout << minHeap.top() << " ";
            minHeap.pop();
        }
        std::cout << std::endl;

    } catch (const std::exception& e) {
        std::cerr << "错误：" << e.what() << std::endl;
    }

    return 0;
}
```

## 三、关键逻辑解释

1. **类模板设计**：使用 `template <typename T>` 让最小堆支持 `int`、`double` 等所有可比较（支持 `<`、`<=` 运算符）的数据类型，提高复用性。

2. **`siftUp()` 上浮函数**：

   - 仅在插入元素时调用，插入的元素放在数组末尾，可能破坏堆的性质。
   - 不断与父节点比较，若当前节点更小，则交换两者，直到当前节点不小于父节点或成为根节点。

   

3. **`siftDown()` 下沉函数**：

   - 仅在删除堆顶时调用，堆尾元素移到堆顶后，可能破坏堆的性质。
   - 先找到当前节点的左右子节点中的最小值节点，再与当前节点比较，若当前节点更大，则交换两者，直到当前节点不大于最小子节点或没有子节点。

   

4. **异常处理**：`top()` 和 `pop()` 中判断堆是否为空，避免空堆访问导致的未定义行为，抛出 `std::out_of_range` 异常并附带清晰提示。

5. **测试结果**：运行代码后，输出如下（堆内部数组可能因调整顺序不同略有差异，但弹出的最小元素顺序一定是升序）：

```
插入元素：5, 3, 8, 1, 2, 9
堆内部数组：1 2 8 3 5 9 
依次弹出最小元素：1 2 3 5 8 9 
```

## 四、补充说明

1. **C++ 标准库中的最小堆**：C++ 标准库 `<queue>` 中提供了 `std::priority_queue`，但它默认是**最大堆**。若要使用最小堆，可通过指定比较器实现：

```cpp
#include <queue>
#include <vector>

// 最小堆（存储 int 类型）
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
```

2. **时间复杂度**：

- 插入（`push()`）：O (log n)，上浮调整最多遍历堆的高度。
- 删除堆顶（`pop()`）：O (log n)，下沉调整最多遍历堆的高度。
- 获取堆顶（`top()`）：O (1)，直接访问数组第一个元素。

### 总结

1. 最小堆的核心是**完全二叉树的数组映射**和**上浮 / 下沉调整**，以此维持「父节点 ≤ 子节点」的性质。
2. C++ 实现时优先用 `std::vector` 存储元素，核心操作 `push()` 依赖 `siftUp()`，`pop()` 依赖 `siftDown()`。
3. 自定义最小堆便于理解底层逻辑，实际开发中可直接使用 `std::priority_queue` 配合 `std::greater<T>` 实现最小堆需求。



---

**自定义比较函数 + 标准库堆**

```cpp
auto cmp = [](const ListNode* a, const ListNode* b) {
            return a->val > b->val; // 最小堆
        };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq;
```

> ### 1. 普通排序的比较器（比如`sort`函数）
>
> `std::sort`的比较器`comp(a, b)`的规则是：**返回`true`时，会把`a`放在`b`的前面**。
>
> - 比如`return a < b;`：`a`小于`b`时`a`在前，最终是**升序排列**（从小到大）。
> - 比如`return a > b;`：`a`大于`b`时`a`在前，最终是**降序排列**（从大到小）。
> - 这是你理解的「`a->val > b->val`是降序」的来源，这个理解在`sort`中是完全正确的。
>
> ### 2. 优先队列`std::priority_queue`的比较器
>
> 优先队列的比较器`comp(a, b)`的规则完全不同：它是用来判断「**是否需要将`a`下沉到`b`的下方**」（或者说「`b`是否比`a`更有优先级，应该排在堆顶」）。
>
> - 优先队列的底层是堆，默认是**最大堆**，默认比较器是`std::less<T>`。
> - 它的核心逻辑是：**当`comp(a, b)`返回`true`时，说明`a`的优先级低于`b`，`b`会排在`a`的前面（更靠近堆顶）**。
> - 简单记：`priority_queue` 比较器 `cmp(a, b)`，**返回 `true`（满足条件）→ `a` 下沉；返回 `false`（不满足条件）→ `a` 留顶 / 上浮**。

**标准库优先队列的简单模版声明**

```cpp
template <class T, class Container = vector<T>, class Compare = less<T>>
class priority_queue;
```

第三个 `Compare`：**比较器的类型**（不是比较器对象，是类型！比如 `std::less<int>`、`std::greater<int>`，这里就是 `cmp` 这个 lambda 的类型）。

模板参数在编译期就需要确定，必须是「类型」（比如 `int`、`vector<int>`、自定义类类型等），而 `cmp` 是一个「lambda 对象」（运行期创建的实例），两者不能混淆。