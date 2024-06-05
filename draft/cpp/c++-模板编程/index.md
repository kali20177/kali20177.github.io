# [未完成] C++ 模板


> SFINAE: Substitution Failure Is Not An Error，替换失败不是错误

## 函数模板

从简单例子开始：

```cpp
template<typename T>
T max(const T& a, const T& b) {
	return a > b ? a : b;
}
```

如果调用时传入两个不同类型值，例如 `max(1, 2.1);`，可以使用如下方式修改。

### 方式一

直接通过**默认模板参数**指定一部分类型的实例化。

### 方式二

```cpp
template<typename T1, typename T2, typename RT =
	decltype(true ? T1{} : T2{})>   // 不求值语境
RT max(const T1& a, const T2& b) {
	return a > b ? a : b;
}
```

其中三目运算符的部分是提取 T1, T2 的公共类型，且其中的 true, false 不影响推断出的类型。T1{} 和 T2{} 是临时的构造对象。例如：

```cpp
using Tx = decltype(true ? 1 : 1.2);
using Ty = decltype(false ? int{} : double{});
```

上面的表达式返回的类型都是 double。

或者直接使用 a, b，不使用临时对象，但是这种情况下 cv 限定符失效。

### 方式三

不需要定义 RT 类型名，后置返回类型 (C++14)，结合 auto 实现：

```cpp
template<typename T1, typename T2>   // a, b 不是临时对象
auto max(const T1& a, const T2& b) -> decltype(true ? a : b) {
	return a > b ? a : b;
}
```

> [!NOTE]  
> C++14 返回值推导中，auto 只是占位符。

### 方式四

C++ 20，全部使用 auto 推导：

```cpp
auto max(const auto& a, const auto& b) {
	return a > b ? a : b;
}
```

## 万能引用

万能引用，又叫转发引用。接受左值表达式形参就推导为左值表达式，接受右值表达式就被推导为右值表达式。

```cpp
template<typename Ty>
constexpr Ty&& forward(Ty& Arg) noexcept {
	return static_cast<Ty&&>(Arg);
}
```

```cpp
int a = 10;
::forward<int>(a);     // 无引用折叠 int&&
::forward<int&>(a);    // int&
::forward<int&&>(a);   // int&&
```

## 变量模板（C++14）


## Concepts（C++20）

给模板参数添加约束或者概念，例如定义一个模板函数，只有某个类及其派生类使用：

```cpp
#include <concepts>
#include <type_traits>
// 基类 A
class A {};
// B 派生自 A 
class B : public A {};
// 不相关的类 C
class C {};

// 定义一个 concept，检查 T 是否是 A 的派生类（包括 A 自己）
template<typename T>
concept IsDerivedFromA = std::is_base_of<A, T>::value;

// 使用 concept 约束的函数模板
template<IsDerivedFromA T>
void myFunction(const T& obj) {
    std::cout << "Called with a type derived from A\n";
}

int main() {
    A a;
    B b;
    C c;
    myFunction(a); // OK
    myFunction(b); // OK
    // myFunction(c); // 错误，C 不是 A 的派生类
    return 0;
}
```

## 相关技巧

### trait 

一种仿 rust traits 的多态：

```cpp
struct Foo() {
    void print() const { std::cout << __PRETTY_FUNCTION__ << "\n"; }
};
struct Bar() {
    void print() const { std::cout << __PRETTY_FUNCTION__ << "\n"; }
};

template <typename T>
struct print_traits
{
    using print_return_t = decltype(std::declval<T>().print());
};

template <typename T, typename traits = print_traits<T>>
auto print(const T &obj) noexcept -> typename traits::print_return_t
{
    return obj.print();
}
```

print_traits 是一个模板结构体，它接受一个类型 T，内部定义一个类型别名 print_return_t，使用 decltype 来推导 T 类型的 print 成员函数的返回类型。

- std::declval<T>() 返回一个类的右值引用，不需要实际创建对象，且对于无默认构造函数的类有效
- `-> typename traits::print_return_t`，表示函数的返回类型是 traits 中的 print_return_t 类型

## 参考

1. [现代 C++ 编程（一）：多态](https://toby-shi-cloud.github.io/posts/cpppolymorphism.html)
2. [理解 std::declval](https://stdrc.cc/post/2020/09/12/std-declval/)
3. [理解 std::declval 和 decltype](https://juejin.cn/post/7021306822350864414)
