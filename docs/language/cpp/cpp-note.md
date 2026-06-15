# cpp note

## Rvalue Reference

> C++11 

| 类别              | 特征               | 例子                                                    | 核心理解                     |
| --------------- | ---------------- | ----------------------------------------------------- | ------------------------ |
| **左值 (Lvalue)** | 有名字、有内存地址、生命周期长。 | int a; std::string s;                                 | 能够持久存在的对象，**不能**随意窃取其资源。 |
| **右值 (Rvalue)** | 临时的、无名的、即将销毁的    | 字面量 "hello"（其实也是有内存地址的，存放在只读数据区）, 函数返回值 func(), x + y | **将死之身**，可以安全地窃取其资源。     |

右值引用，**T&&**

- 一种新的引用类型，专门用来绑定到右值上
- 目的：让编译器能够区分传进来的是可能需要被继续使用的左值还是可以窃取资源的右值

> 只要有名字就是左值，即使一个变量的类型是右值引用 `T&&`，只要它在函数体内有名字，它就在表达式中表现为左值
>
> ```C++
> void func(std::string&& str) {
>     // str 的【类型】是：右值引用 (std::string&&)
>     // str 的【值类别】是：左值 (Lvalue) —— 因为它叫 "str"
>     
>     std::string s1 = str;            // 调用拷贝构造 (深拷贝)
>     std::string s2 = std::move(str); // 调用移动构造 (窃取资源)
> }
> ```

!!! success
    右值引用 (T&&) 和右值这个概念，在很大程度上就是给编译器看的一个“特殊的类型标签”，目的是为了指挥编译器走另一条路。而在底层左值引用 (T&) 和右值引用 (T&&) 本质都是指针

## Binding

绑定是引用变量去和值建立联系的“动作”，在 C++（特别是涉及引用的语境）中，“绑定”指的就是**把一个“名字”（引用变量）和一块“内存”（实际的对象）连起来的过程。**

可以把它想象成 **“系绳子”**。

### 左值引用绑定

```C++
int a = 10;      // a 是一只名叫“a”的狗（实际对象）
int& ref = a;    // ref 是一根绳子。
                 // “绑定”就是把绳子 ref 系在了狗 a 的脖子上。
                 // 以后拽 ref 这根绳子，就是在拽 a 这只狗。
```

### 右值引用绑定

```C++
// "hello" 是一个临时的字符串对象，它本来没有名字，是个流浪狗。
// 如果没人理它，这一行结束它就“死”了（析构）。
std::string&& r_ref = "hello"; 

// 此时发生“绑定”：
// r_ref 这根绳子，把这个临时的流浪狗拴住了。
// 只要绳子 r_ref 还在，这条狗就不会死（生命周期延长）。
```

> 非 const 左值引用不能绑定到右值，因为 C++ 认为非 const 左值引用可能会修改它绑定的对象
>
> ```C++
> // void func(std::string& s)
> func("hello"); // 报错！
> ```

## Move Semantics

> C++11

`std::move` 本质是进行强制类型转换，强制将一个**左值**转换成**右值引用**类型，本质上可以理解为“贴标签”

- **没有 std::move**：变量 a 身上贴着“贵重物品，小心轻放”的标签（左值）。
- **用了 std::move**：相当于你把 a 原来的标签撕了，贴上一张“**废品，由于随意处置**”的标签（转为右值引用类型）。

编译器看到“废品”标签，就放心地把它交给了回收站（移动构造函数）去处理，而不是交给复印店（拷贝构造函数）

“移动语义”的实现是由移动构造函数（移动拷贝）来完成的，而在绝大多数场景下，移动构造函数都会由编译器自动生成：
```cpp
  class UserInfo {
  public:
      std::string name;          // 标准库 string，懂移动
      std::vector<int> history;  // 标准库 vector，懂移动
      int age;                   // 基础类型，直接拷贝（等同于移动）

      // 【关键】什么都不用写！
      // 编译器会自动生成：
      // UserInfo(UserInfo&&) { 
      //     name.move(); 
      //     history.move(); 
      //     age = other.age; 
      // }
  };

  void test() {
      UserInfo u1;
      // ... 填数据 ...

      // 这一行依然会极其高效！
      // 编译器自动生成的移动构造函数会去“偷” u1.name 和 u1.history 的资源
      UserInfo u2 = std::move(u1); 
  }
```

> C++11 后，C++98 的 **Rule of three **(Desctructor/Copy Constructor/Copy Assignment O,,   perator) --> C++11 的 **Rule of five** (plus Move Constructor/ Move Assignment Operator) / **Rule of zero**

> **Rule of zero:** 如果你的类不需要手动管理原始资源（如 `new` 出来的指针、文件句柄），那么你就不要去写那 5 个特殊成员函数（析构、拷贝/移动构造、拷贝/移动赋值），全部交给编译器自动生成。

!!! success
    在 Modern C++ 风格下，大部分场景下，我们都可以利用标准库容器，让编译器帮我们把写特殊成员函数的脏活都自动干完

## std::string_view

> C++17 <string_view>

`std::string_view`是一个字符串的“快照”或“窗口”，它只包含一个指针和一个长度，不拥有内存，也不进行拷贝

本质上是一个极其轻量的结构体，只包含两个成员变量:

```C++
// 伪代码实现
class string_view {
    const char* data_; // 指向字符串的起始位置
    size_t size_;      // 字符串的长度
    // ... 加上一堆类似 string 的只读成员函数 (find, substr, etc.)
};
```

在 C++17 之前，当我们编写一个接收字符串的函数时，通常有两种写法，但都有缺点：

**使用 const char**:(C 风格)

```C++
void func(const char* str);
```

- **优点**：高效，无拷贝。
- **缺点**：丢失了长度信息（需要 strlen，O(N)），且无法方便地利用 std::string 的丰富 API（如 find, substr)

**使用 const std::string& **(C++98/11 标准写法):

```C++
void func(const std::string& str);
```

- **优点**：API 丰富，对于已经存在的 std::string 对象来说很高效（引用传递）。
- **缺点**：当你传入 **字符串字面量**（"hello"）或 **子串** 时，它会发生昂贵的**内存分配**
  - 比如 `func("hello world")`：为了让引用类型的形参能够绑定，编译器会隐式的创建一个临时对象把 `const char*` 转为 `std::string`，那么就需要
    - 在堆上分配内存
    - 把"hello world"拷贝进去
    - 函数结束后再释放临时对象的内存
  - 比如 std:string 的 API substr():`long_str.substr(0.5)`, substr() 会做深拷贝返回一个新的 std:string 对象

C++17 引入的 `std::string_view` 在这些场景在就会变得非常经济

## constexpr

> C++11

`constexpr` 关键字用于声明编译时常量表达式，可以将本来需要在运行时执行的计算移到编译阶段，编译器在编译阶段就直接计算结果并将结果硬编码到指令中，节省了运算开销

- **`constexpr` 变量**：声明变量的值必须在编译时确定
- **`constexpr` 函数**：声明一个函数可以在常量表达式中使用，并且如果传入的参数是编译时常量，它本身的值也将在编译时计算出来

有点类似“内联”，但存在一点区别：

- **inline (内联)**：是**“复制粘贴”**。它关注的是**执行效率**（省去函数调用的开销）。
- **constexpr (常量表达式)**：是**“提前计算”**。它关注的是**计算时机**（把运行时要做的事挪到编译时做）。

比如我们计算 add(3,4)：

`inline int add(int a, int b) { return a + b }`

- **编译器的行为**：编译器把 a + b 这行代码“剪切”下来，贴到 main 函数里
- **最终的机器码**：程序里依然会有 ADD（加法）指令
- **运行时**：程序跑起来后，**CPU** 依然要执行一次 3 + 4 的运算

`constexpr int add(int a, int b) { return a + b }`

- **编译器的行为**：编译器在编译阶段直接算出了结果 7
- **最终的机器码**：程序里没有加法指令，只有一个立即数 7 (MOV EAX, 7)
- **运行时**：程序跑起来后，**CPU** 什么都不用算，直接拿结果

> 几个关键字拜访的类型一般是：static constexpr const Type
>
> - **static**：管生命周期或者说内存中位置
> - **constexpr**：管编译期计算
> - **const**：管数据内容只读
> - **Type**: 管变量数据类型

## enum class

> c++11

C++11 引入了 enum class (限定作用域枚举)，解决了老式 enum 的几乎所有痛点，例如

- 全局作用域污染：enum class 使用枚举值时应当按照“枚举名:枚举值”的方式使用
- 避免隐式转换

==应该成为写代码的默认选择==

Usage：

`enum class 枚举名 : 底层类型（可选） {枚举值...};`

example:

```C++
enum class Status: char { // 指定底层用 char 存储 ，省内存
  Ok = 0,
  Error,
  Unknown
};

Status s = Status::Error; // 像使用类一样，类名::成员变量

int val = static_cast<int>(s) // enum class 不能隐式转 int
```

## std::optional

> C++17

`std::optional<T>` 的取值范围是 T 的所有值 + 空 (nullopt)

!!! success
    推荐用在函数返回值上，用来表示返回值可能为空

```C++
std::optional<User> findUser(int id) {
  if (id < 0) {
  	return {}
    // 也可以 return std::nullopt
  }
	else {
    ...
    return user[id]; // 自动包装成 optional<User>
  }
}
```

- .hasvalue() 若非空返回 true

- .value() 返回容器内存储值的引用

  > C++ 标准库为了让我们写代码更方便（Syntactic Sugar），允许我们像使用指针一样直接对 std::optional 使用 -> 操作符，这样我们就不用每次都啰嗦地写 .value() 然后再去调用容器内存储值的方法或对象了

## using

> C++11

using Identifier = Type

用于给类型起别名，可以完全替代 C 风格的 `typedef` ，它的可读性更强，且支持模板

例如：

```C++
// --- 使用 using 极其优雅 ---
template <typename Val>
using MyMap = std::map<std::string, Val>;

// 用的时候：
MyMap<int> inventory;
MyMap<float> prices;
```

## Range-based for loop

> C++11

```C++
std::vector<int> nums = {1, 2, 3, 4, 5};

//     (1)    (2) (3)  (4)
for ( auto     &   x : nums ) {
    x = x * 2;
}
```

在 for 循环中，auto 有三种常见的写法，区别在于**拷贝**和**读写权限**。

==写法 A==：for (auto x : nums) —— 【值传递 / 拷贝】

- **含义**：把容器里的元素**复制**一份给 x。
- **后果**：
  1. **修改无效**：你在循环里修改 x，改的只是副本，**原容器 nums 里的数据不会变**。
  2. **性能损耗**：如果容器里存的是 std::string 或大结构体，每次循环都要进行一次**深拷贝**，效率很低。
- **场景**：只读访问基础类型（int, char, bool）。

```C++
for (auto x : nums) {
    x = 100; // nums 里的数据没变，只改了 x 这个临时工
}
```

==写法 B==：for (auto &x : nums) —— 【引用传递 / 读写】

- **含义**：x 是容器里元素的**别名（引用）**。x 就是那个元素本身。
- **后果**：
  1. **可以修改**：你在循环里修改 x，**原容器 nums 里的数据也会跟着变**。
  2. **高效**：没有拷贝发生，只是起个绰号。
- **场景**：需要修改容器内的元素，或者遍历大对象（为了避免拷贝）。

```C++
for (auto &x : nums) {
    x = 100; // nums 里的所有元素都变成了 100
}
```

==写法 C==：for (const auto &x : nums) —— 【常量引用 / 只读高效】

- **含义**：x 是元素的引用，但**被锁住了（const）**，不能修改。
- **后果**：
  1. **不能修改**：保证数据安全，防止手误改掉数据。
  2. **高效**：没有拷贝。
- **场景**：==这是最常用的写法！==当你只想读取容器里的数据（比如打印、查找），且数据类型较大（如 string, vector）时。

## if with initializer
> c++17

C++17 允许你在 if 圆括号内用分号分隔“初始化”和“条件判断”，就像 for 循环的前半部分一样。

语法：
```cpp
if (初始化语句; 条件表达式) {
    // ...
}
```

示例：
```cpp
#include <iostream>
#include <map>

int main() {
    std::map<int, std::string> m = {{1, "one"}};

    // 【C++17 写法】
    // 分号左边：定义并初始化变量 it
    // 分号右边：判断条件
    if (auto it = m.find(1); it != m.end()) {
        std::cout << "Found: " << it->second << std::endl;
        // it 在这里有效
    } 
    else {
        // it 在 else 里也有效！
        std::cout << "Not found" << std::endl;
    }
    
    // it 在这里失效了，不会污染外部作用域
}
```

## noexcept

> C++11

noexcept 可用于函数的 specifier 中，在编译期声明该函数不会抛出异常，如果抛出异常直接终止程序，从而让编译器能够进行更为激进的优化

在 C++ 中，为了支持异常处理，编译器必须生成大量额外的处理代码

*   没有 `noexcept` 时：
    编译器必须假设 `Fun()` 可能会抛出异常。如果 `Fun()` 被 `Caller()` 调用，编译器必须在 `Caller()` 里记录：“如果在调用 `Fun()` 时发生了异常，我需要先析构 `Caller()` 里已经创建的局部变量 A、B、C... 然后再把异常往上传。” 这个过程叫 **栈展开（Stack Unwinding）**。这需要生成额外的指令和查表操作，阻碍了指令重排等优化。

*   有了 `noexcept` 时：
    编译器知道 `Fun()` 绝对不会抛出异常返回给调用者。因此：
    1.  无需准备后路：调用者不需要为这次调用准备复杂的栈展开路径。
    2.  控制流简化：对于编译器来说，这个函数调用只有“成功返回”这一条路，没有“抛出异常”的隐形分支。这使得控制流图（CFG）更简单，编译器敢于进行更激进的指令重排、死代码消除等优化。

## std::chrono

> C++11 `<chrono>`

它通过强类型系统（Type Safety），解决了 C 语言时代用“裸整数”表示时间所带来的单位混淆和精度问题

主要包括三个概念

- 时长（duration）：`std::chrono::duration`
- 时间点（timepoint）：`std::chrono:time_point`
- 时钟（clock）：`std::chrono::system_clock`
  - `std::chrono::system_clock`：系统时钟
  - `std::xhrono::steady_clock`：秒表，单调递增，在计算 benchmark 时需要用这个

**Example**:

```cpp
#include <iostream>
#include <chrono>
#include <thread>

void heavy_work() {
    std::this_thread::sleep_for(std::chrono::milliseconds(200)); // 模拟耗时
}

int main() {
    // 1. 记录开始时间 (使用 steady_clock)
    auto start = std::chrono::steady_clock::now();

    heavy_work();

    // 2. 记录结束时间
    auto end = std::chrono::steady_clock::now();

    // 3. 计算时长 (时间点相减)
    // duration_cast 用于将纳秒精度的结果强制转换为我们想要的毫秒
    auto diff = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "耗时: " << diff.count() << " ms" << std::endl;
    
    return 0;
}
```

## Multithreading

> C++11

传统的C++（C++11标准之前）中并没有引入线程这个概念，在C++11出来之前，如果我们想要在C++中实现多线程，需要借助操作系统平台提供的API，比如Linux的<pthread.h>，或者windows下的<windows.h> 。

C++11提供了语言层面上的多线程，包含在头文件中。它解决了跨平台的问题，提供了管理线程、保护共享数据、线程间同步操作、原子操作等类。C++11 新标准中引入了5个头文件来支持多线程编程

- `<thread>`
  - **thread**
    - constructor
    - join
    - detach
  - **this_thread**
    - get_id
    - yield
    - sleep_for
    - sleep_until
- `<mutex>`
  - mutex
  - lock_gaurd
  - unique_lock
- `<atomic>`
- `<condition_variable>`

### \<thread\>

`std::thread` 是现代 C++ 处理并发的标准方式，它封装了底层操作系统的线程 API（如 pthread 或 Win32 线程），提供了一个跨平台的执行流管理工具。

| 核心操作         | 语义                                                 | 后果                                      | 工业界场景                         |
| :--------------- | :--------------------------------------------------- | :---------------------------------------- | :--------------------------------- |
| **`t.join()`**   | **等待**。主线程阻塞，直到子线程执行完毕。           | 同步执行，确保子线程资源被回收。          | 计算任务、必须获取结果的场景。     |
| **`t.detach()`** | **分离**。子线程驻留后台，独立运行，主线程不再管它。 | 线程生命周期可能超过 `std::thread` 对象。 | 服务端 Socket 处理、后台监控任务。 |

一旦创建了线程，必须在 `std::thread` 对象析构前显式调用 `join()` 或 `detach()`。如果不调用，程序会直接 `std::terminate`（崩溃）

```cpp
void task() { /* ... */ }

void example() {
    std::thread t(task);
    // 这里必须二选一：
    // t.join(); 
    // t.detach(); 
} // 若未处理，t 析构时会崩溃
```

`std::thread` 的构造函数会**拷贝**所有传入的参数。

- **按值传递（默认）**：最安全，每个线程拥有自己的数据副本，避免竞态。
- **按引用传递**：必须使用 `std::ref` 包装。

```cpp
void worker(int n, std::string& s) { s += std::to_string(n); }

int main() {
    int val = 42;
    std::string msg = "Score: ";
    
    // 报错：std::thread 默认尝试拷贝所有参数，无法直接绑定到 string&
    // std::thread t(worker, val, msg); 
    
    // 正确：使用 std::ref 显式声明传递引用
    std::thread t(worker, val, std::ref(msg)); 
    t.join();
}
```

**std::this_thread** 命名空间中包含了许多操作当前正在运行的线程的函数

| 函数              | 功能                                | 例子                                                    |
| :---------------- | :---------------------------------- | :------------------------------------------------------ |
| **`get_id()`**    | 获取当前线程的唯一标识符（TID）。   | `LOG(INFO) << std::this_thread::get_id();`              |
| **`sleep_for()`** | 让当前线程阻塞指定时间。            | `std::this_thread::sleep_for(std::chrono::seconds(1));` |
| **`yield()`**     | 主动让出 CPU 时间片，重新参与调度。 | 用于忙等待（Busy-waiting）的优化。                      |

`std::thread` 对象是**不可拷贝**的，但可以**移动**。这意味着你可以把一个创建好的线程所有权“转让”给另一个变量或存入容器（如 `std::vector`）

```cpp
std::thread t1(task);
std::thread t2 = std::move(t1); // t1 现在为空，t2 接管任务
```

!!! success
    **工业界避坑指南：**
    在 `detach` 场景下，绝对不要向线程传递局部变量的**指针或引用**。因为主线程可能已经执行结束并销毁了局部变量，而后台线程仍在尝试访问它，这将导致典型的 **Use-After-Free** 崩溃。

### \<mutex\>

mutex 头文件主要声明了与互斥量(mutex)相关的类。mutex提供了4种互斥类型，如下表所示。

| 类型                       | 说明                |
| -------------------------- | ------------------- |
| std::mutex                 | 最基本的 Mutex 类。 |
| std::recursive_mutex       | 递归 Mutex 类。     |
| std::time_mutex            | 定时 Mutex 类。     |
| std::recursive_timed_mutex | 定时递归 Mutex 类。 |

std::mutex 是 C++11 中最基本的互斥量，std::mutex 对象提供了独占所有权的特性——即不支持递归地对 std::mutex 对象上锁，而 std::recursive_lock 则可以递归地对互斥量对象上锁

#### lock & unlock

mutex 的常见操作有：

- lock()：资源上锁
- unlock()：解锁资源

现代 C++ 中不推荐使用，因为若 mtx.lock 和 mtx.unlock 中抛出异常，导致控制流离开当前代码块导致无法解锁，导致**死锁（Deadlock）**，整个程序卡死

Example:

```cpp
#include <iostream>  // std::cout
#include <thread>  // std::thread
#include <mutex>  // std::mutex

std::mutex mtx;  // mutex for critical section
void print_block (int n, char c) 
{
// critical section (exclusive access to std::cout signaled by locking mtx):
    mtx.lock();
    for (int i=0; i<n; ++i) 
    {
       std::cout << c; 
    }
    std::cout << '\n';
    mtx.unlock();
}
int main ()
{
    std::thread th1 (print_block,50,'');//线程1：打印*
    std::thread th2 (print_block,50,'$');//线程2：打印$

    th1.join();
    th2.join();
    return 0;
}
```

#### lock_guard

现代 C++ 90% 的简单情况，更推荐使用 lock_guard，

创建 lock_guard 对象时，它将尝试获取提供给它的互斥锁的所有权，当控制流离开 lock_guard 对象的作用域时，lock_guard 析构并自动释放互斥锁

简单而言即为：

- 创建时即加锁，作用域结束自动析构并解锁，无需手工解锁
- 不能中途复制，必须等作用域结束才解锁

Example:

```cpp
std::mutex mtx;
{
    std::lock_guard<std::mutex> guard(mtx); // 上锁
    count++;
    // ... 做点简单的事 ...
} // 自动解锁
```

#### unique_lock

unique_lock 是 lock_guard 的超集，相较于 lock_guard 而言功能更加丰富，一般而言需要配合 condition_variable 使用

主要特点为：

- 可以延迟上锁
- 可以中途解锁

Example:
```cpp
std::mutex mtx;
{
    std::unique_lock<std::mutex> lock(mtx); // 上锁
    
    // 操作共享数据
    count++;

    lock.unlock(); // 手动解锁！(lock_guard 做不到)
    
    // 做一些不需要锁的耗时操作 (比如 IO)
    // 这样不会阻塞其他线程太久
    
    lock.lock(); // 再锁回来
    // 继续操作共享数据
} // 自动解锁
```

### \<conditoin_variable\>

condition_variable 提供了一种机制，允许线程在某些条件不满足时挂起，直到其他线程通知它们条件已经满足

相比低级同步原语（例如互斥锁）更加方便和安全，也能够解决“忙等待”的问题。通常搭配 unique_lock 一起使用

主要包括这几个方法：

- void wait(unique_lock<mutex>& lock)
  - 通常在 unique_lock 创建语句和 unlock 解锁（不过 unique_lock 会在离开作用域时自动解锁）中间调用
  - wait 内部的工作流程是：
    - 入睡时自动解锁
    - 醒来时自动加锁
- void notify_one() noexcept;
- void notify_all() noexcept

下面是一个使用`condition_variable`的简单示例，展示了生产者-消费者问题的基本实现

```cpp
#include <iostream>
#include <condition_variable>
#include <mutex>
#include <queue>
#include <thread>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> product;

void producer(int id) {
    for (int i = 0; i < 5; ++i) {
        std::unique_lock<std::mutex> lck(mtx);
        product.push(id * 100 + i);
        std::cout << "Producer " << id << " produced " << product.back() << std::endl;
        cv.notify_one();
        lck.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lck(mtx);
        cv.wait(lck, []{ return !product.empty(); });
        if (!product.empty()) {
            int prod = product.front();
            product.pop();
            std::cout << "Consumer " << id << " consumed " << prod << std::endl;
        }
        lck.unlock();
    }
}

int main() {
    std::thread producers[2];
    std::thread consumers[2];

    for (int i = 0; i < 2; ++i) {
        producers[i] = std::thread(producer, i + 1);
    }

    for (int i = 0; i < 2; ++i) {
        consumers[i] = std::thread(consumer, i + 1);
    }

    for (int i = 0; i < 2; ++i) {
        producers[i].join();
        consumers[i].join();
    }

    return 0;
}
```

## lambda expression

> C++11

Lambda 表达式就是一个“匿名函数”。它允许你在代码中原地（on-the-spot）定义一个临时的函数，而不需要专门去写一个单独的 void functionName() { ... }

语法规则：

```cpp
[ 捕获列表 ] ( 参数列表 ) -> 返回值类型 { 函数体 }
```

- **[] (捕获列表)**：最灵魂的部分，它决定了 Lambda 内部能否使用外部的局部变量
  - **[x] (按值捕获)**：把变量 x **拷贝**一份进 Lambda，内部修改不影响外部
  - **[&x] (按引用捕获)**：把 x 的**引用**传进去，内部修改会影响外部
  - **[=]**：按值捕获外部**所有**可见的局部变量
  - **[&]**：按引用捕获外部**所有**可见的局部变量
- **() (参数列表)**：跟普通函数一样。如果没有参数，可以省略
- **-> 返回值**：指定返回类型（如 -> int）。通常可以省略，编译器会自动推导。
- **{} (函数体)**：具体的逻辑代码

Example：
```cpp
int main() {
    int clientSock = 10;
    
    // 原地定义逻辑
    std::thread t([clientSock]() {
        LOG(INFO) << "Handling client on socket: " << clientSock;
        // 这里可以直接写 recv/send
    });
    
    t.detach();
}
```
