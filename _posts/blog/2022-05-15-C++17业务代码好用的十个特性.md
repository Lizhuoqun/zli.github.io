---
layout: post
title: "C++17在业务代码中最好用的十个特性"
modified:
categories: blog
excerpt:
tags: [tech]
image:
  feature:
date: 2022-05-15

---
# C++17在业务代码中最好用的十个特性
自从步入现代C++时代开始，C++语言标准形成了三年一个版本的惯例：C++11标志着现代C++的开端，C++14在11的基础上查缺补漏，并未加入许多新特性，而C++17作为C++11后的第一个大版本，标志着现代C++逐渐走向成熟。WXG编译器升级到gcc7.5已有一段时间，笔者所在项目组也已经将全部代码升级到C++17。在使用了c++17一年多之后，笔者总结了C++17在业务代码中最好用的十个特性。

注1：本文只包含wxg的gcc7.5支持的特性，[Execution Policy](https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag_t), [File System](https://en.cppreference.com/w/cpp/filesystem)等暂不支持的特性不包含在内。

注2：本文只包含应用于业务逻辑的特性，[Fold Expression](https://en.cppreference.com/w/cpp/language/fold), [Mathematical Special Functions](https://en.cppreference.com/w/cpp/numeric/special_functions)等适用于元编程和科学计算的特性并不包含。

笔者将这些特性大体上分为三类：语法糖、性能提升和类型系统

## 语法糖
这里所说的语法糖，并不是严格意义上编程语言级别的语法糖，还包括一些能让代码更简洁更具有可读性的函数和库：

### [结构化绑定](https://en.cppreference.com/w/cpp/language/structured_binding)
c++17最便利的语法糖当属结构化绑定。结构化绑定是指将array、tuple或struct的成员绑定到一组变量*上的语法，最常用的场景是在遍历map/unordered_map时不用再声明一个中间变量了:
```cpp
// pre c++17
for(const auto& kv: map){
  const auto& key = kv.first;
  const auto& value = kv.second;
  // ...
}

// c++17
for(const auto& [key, value]: map){
  // ...
}
```

*: 严格来说，结构化绑定的结果并不是变量，c++标准称之为名字/别名，这也导致它们不允许被lambda捕获，但是gcc并没有遵循c++标准，所以以下代码在gcc可以编译，clang则编译不过
``` cpp
for(const auto& [key, value]: map){
    [&key, &value]{
        std::cout << key << ": " << value << std::endl;
    }();
}
```
在clang环境下，可以在lambda表达式捕获时显式引入一个引用变量通过编译
```cpp
for(const auto& [key, value]: map){
    [&key = key, &value = value]{
        std::cout << key << ": " << value << std::endl;
    }();
}
```
另外这条限制在c++20中已经被删除，所以在c++20标准中gcc和clang都可以捕获结构化绑定的对象了。

### [std::tuple的隐式推导](https://zh.cppreference.com/w/cpp/utility/tuple/deduction_guides)
在c++17以前，构造`std::pair/std::tuple`时必须指定数据类型或使用`std::make_pair/std::make_tuple`函数，c++17为`std::pair/std::tuple`新增了推导规则，可以不再显示指定类型。
```cpp
// pre c++17
std::pair<int, std::string> p1{3.14, "pi"s};
auto p1 = std::make_pair(3.14, "pi"s);

// c++17
std::pair p3{3.14, "pi"s};
```

### [if constexpr](https://en.cppreference.com/w/cpp/language/if)
if constexpr语句是编译期的if判断语句，在C++17以前做编译期的条件判断往往通过复杂[SFINAE](https://en.cppreference.com/w/cpp/language/sfinae)机制或模版重载实现，甚至嫌麻烦的时候直接放到运行时用if判断，造成性能损耗，if constexpr大大缓解了这个问题。比如我想实现一个函数将不同类型的输入转化为字符串，在c++17之前需要写三个函数去实现，而c++17只需要一个函数。
```cpp
// pre c++17
template <typename T>
std::string convert(T input){
    return std::to_string(input);
}

// const char*和string进行特殊处理
std::string convert(const char* input){
    return input;
}
std::string convert(std::string input){
    return input;
}
```

```cpp
// c++17
template <typename T>
std::string convert(T input) {
    if constexpr (std::is_same_v<T, const char*> ||
                  std::is_same_v<T, std::string>) {
        return input;
    } else {
        return std::to_string(input);
    }
}
```

### [if初始化语句](https://en.cppreference.com/w/cpp/language/if)
c++17支持在if的判断语句之前增加一个初始化语句，将仅用于if语句内部的变量声明在if内，有助于提升代码的可读性。且对于lock/iterator等涉及并发/RAII的类型更容易保证程序的正确性。
```cpp
// c++ 17
std::map<int, std::string> m;
std::mutex mx;
extern bool shared_flag; // guarded by mx
 
int demo()
{
    if (auto it = m.find(10); it != m.end()) { return it->second.size(); }
    if (char buf[10]; std::fgets(buf, 10, stdin)) { m[0] += buf; }
    if (std::lock_guard lock(mx); shared_flag) { unsafe_ping(); shared_flag = false; }
    if (int s; int count = ReadBytesWithSignal(&s)) { publish(count); raise(s); }
    if (const auto keywords = {"if", "for", "while"};
        std::ranges::any_of(keywords, [&tok](const char* kw) { return tok == kw; }))
    {
        std::cerr << "Token must not be a keyword\n";
    }
}
```

## 性能提升
### [std::shared_mutex](https://en.cppreference.com/w/cpp/thread/shared_mutex)
`shared_mutex`是c++的原生读写锁实现，有共享和独占两种锁模式，适用于并发高的读场景下，通过reader之前共享锁来提升性能。在c++17之前，只能自己通过独占锁和条件变量自己实现读写锁或使用c++14加入的性能较差的`std::shared_timed_mutex`。以下是通过`shared_mutex`实现的线程安全计数器：

```cpp
// c++17
class ThreadSafeCounter {
 public:
  ThreadSafeCounter() = default;
 
  // Multiple threads/readers can read the counter's value at the same time.
  unsigned int get() const {
    std::shared_lock lock(mutex_);
    return value_;
  }
 
  // Only one thread/writer can increment/write the counter's value.
  unsigned int increment() {
    std::unique_lock lock(mutex_);
    return ++value_;
  }
 
  // Only one thread/writer can reset/write the counter's value.
  void reset() {
    std::unique_lock lock(mutex_);
    value_ = 0;
  }
 
 private:
  mutable std::shared_mutex mutex_;
  unsigned int value_ = 0;
};
```

### [std::string_view](https://en.cppreference.com/w/cpp/string/basic_string_view)
`std::string_view`顾名思义是字符串的“视图”，类成员变量包含两个部分：字符串指针和字符串长度，std::string_view涵盖了std::string的所有只读接口。std::string_view对字符串不具有所有权，且兼容std::string和const char*两种类型。

c++17之前，我们处理只读字符串往往使用`const std::string&`，`std::string`有两点性能优势:

1. 兼容两种字符串类型，减少类型转换和内存分配。如果传入的是明文字符串`const char*`, `const std::string&`需要进行一次内存分配，将字符串拷贝到堆上，而`std::string_view`则可以避免。
2. 在处理子串时，`std::string::substr`也需要进行拷贝和分配内存，而`std::string_view::substr`则不需要，在处理大文件解析时，性能优势非常明显。

```cpp
// from https://stackoverflow.com/a/40129046
// author: Pavel Davydov

// string_view的remove_prefix比const std::string&的快了15倍
string remove_prefix(const string &str) {
  return str.substr(3);
}
string_view remove_prefix(string_view str) {
  str.remove_prefix(3);
  return str;
}

static void BM_remove_prefix_string(benchmark::State& state) {                
  std::string example{"asfaghdfgsghasfasg3423rfgasdg"};
  while (state.KeepRunning()) {
    auto res = remove_prefix(example);
    // auto res = remove_prefix(string_view(example)); for string_view
    if (res != "aghdfgsghasfasg3423rfgasdg") {
      throw std::runtime_error("bad op");
    }
  }
}
```
### [std::map/unordered_map try_emplace](https://en.cppreference.com/w/cpp/container/map/try_emplace)
在向`std::map/unordered_map`中插入元素时，我们往往使用`emplace`，`emplace`的操作是如果元素key不存在，则插入该元素，否则不插入。但是在元素已存在时，`emplace`仍会构造一次待插入的元素，在判断不需要插入后，立即将该元素析构，因此进行了一次多余构造和析构操作。c++17加入了`try_emplace`，避免了这个问题。同时try_emplace在参数列表中将key和value分开，因此进行原地构造的语法比`emplace`更加简洁

```cpp
std::map<std::string, std::string> m;
// emplace的原地构造需要使用std::piecewise_construct，因为是直接插入std::pair<key, value>
m.emplace(std::piecewise_construct,
           std::forward_as_tuple("c"),
           std::forward_as_tuple(10, 'c'));

// try_emplace可以直接原地构造，因为参数列表中key和value是分开的
m.try_emplace("c", 10, 'c')
```
同时，c++17还给`std::map/unordered_map`加入了`insert_or_assign`函数，可以更方便地实现插入或修改语义

## 类型系统
c++17进一步完备了c++的类型系统，终于加入了众望所归的类型擦除容器([Type Erasure](https://en.wikipedia.org/wiki/Type_erasure))和代数数据类型([Algebraic Data Type](https://en.wikipedia.org/wiki/Algebraic_data_type))

### [std::any](https://en.cppreference.com/w/cpp/utility/any)
`std::any`是一个可以存储任何可拷贝类型的容器，C语言中通常使用`void*`实现类似的功能，与`void*`相比，`std::any`具有两点优势：

1. `std::any`更安全：在类型T被转换成`void*`时，T的类型信息就已经丢失了，在转换回具体类型时程序无法判断当前的`void*`的类型是否真的是T，容易带来安全隐患。而`std::any`会存储类型信息，`std::any_cast`是一个安全的类型转换。
2. `std::any`管理了对象的生命周期，在`std::any`析构时，会将存储的对象析构，而`void*`则需要手动管理内存。
   
`std::any`应当很少是程序员的第一选择，在已知类型的情况下，`std::optional`, `std::variant`和继承都是比它更高效、更合理的选择。只有当对类型完全未知的情况下，才应当使用`std::any`，比如动态类型文本的解析或者业务逻辑的中间层信息传递。

### [std::optional](https://en.cppreference.com/w/cpp/utility/optional)
`std::optional<T>`代表一个可能存在的T值，对应Haskell中的`Maybe`和Rust/OCaml中的`option`，实际上是一种[Sum Type](https://en.wikipedia.org/wiki/Tagged_union)。常用于可能失败的函数的返回值中，比如工厂函数。在C++17之前，往往使用`T*`作为返回值，如果为`nullptr`则代表函数失败，否则`T*`指向了真正的返回值。但是这种写法模糊了所有权，函数的调用方无法确定是否应该接管`T*`的内存管理，而且`T*`可能为空的假设，如果忘记检查则会有SegFault的风险。

```cpp
// pre c++17
ReturnType* func(const std::string& in) {
    ReturnType* ret = new ReturnType;
    if (in.size() == 0)
        return nullptr;
    // ...
    return ret;
}

// c++17 更安全和直观
std::optional<ReturnType> func(const string& in) {
    ReturnType ret;
    if (in.size() == 0)
        return nullopt;
    // ...
    return ret;
}
```

### [std::variant](https://en.cppreference.com/w/cpp/utility/variant)
`std::variant<T, U, ...>`代表一个多类型的容器，容器中的值是制定类型的一种，是通用的Sum Type，对应Rust的`enum`。是一种类型安全的`union`，所以也叫做`tagged union`。与`union`相比有两点优势：

1. 可以存储复杂类型，而union只能直接存储基础的POD类型，对于如`std::vector`和`std::string`就等复杂类型则需要用户手动管理内存。
2. 类型安全，variant存储了内部的类型信息，所以可以进行安全的类型转换，c++17之前往往通过`union`+`enum`来实现相同功能。

通过使用`std::variant<T, Err>`，用户可以实现类似Rust的`std::result`，即在函数执行成功时返回结果，在失败时返回错误信息，上文的例子则可以改成:

```cpp
std::variant<ReturnType, Err> func(const string& in) {
    ReturnType ret;
    if (in.size() == 0)
        return Err{"input is empty"};
    // ...
    return {ret};
}
```

需要注意的是，c++17只提供了一个库级别的variant实现，没有对应的[模式匹配(Pattern Matching)](https://en.wikipedia.org/wiki/Pattern_matching)机制，而最接近的`std::visit`又缺少编译器的优化支持，所以在c++17中`std::variant`并不好用，跟Rust和函数式语言中出神入化的Sum Type还相去甚远，但是已经有许多围绕`std::variant`的提案被提交给c++委员会探讨，包括模式匹配，`std::expected`等等。

总结一下，c++17新增的三种类型给c++带来了更现代更安全的类型系统，它们对应的使用场景是：
 - `std::any`适用于之前使用`void*`作为通用类型的场景。
 - `std::optional`适用于之前使用`nullptr`代表失败状态的场景。
 - `std::variant`适用于之前使用`union`的场景。

## 总结
以上是笔者在生产环境中最常用的c++17特性，除了本文描述的十个特性外，c++17还添加了如[lambda值捕获*this](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture), [钳夹函数std::clamp()](https://en.cppreference.com/w/cpp/algorithm/clamp), [强制检查返回值[[nodiscard]]](https://en.cppreference.com/w/cpp/language/attributes/nodiscard)等非常易用的特性，本文篇幅有限不做赘述，欢迎有兴趣的读者自行探索。
