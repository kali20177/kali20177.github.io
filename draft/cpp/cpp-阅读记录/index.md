# [未完成] CPP_阅读记录


TODO: 本文内容拆分到其他主题

## C++ 语言的设计和演化

![cpp history](/CPP/CPP_history.png)

## C++ 之旅

TODO: 拆分到 "语法"

### chapter 15 指针和容器

多个指针可以指向同一个对象，**拥有所有权的指针负责删除他指向的对象**。没有所有权的指针**可以悬空**，所以没有所有权的指针允许成为野指针。

- unique_ptr 唯一所有权，析构函数负责销毁对象。和内置指针相比，unique_ptr 是一种没有时间或空间开销的轻量级机制。
- shared_ptr 共享所有权，最后一个共享的析构函数负责销毁对象。shared_ptr 提供了某种形式的垃圾回收，确保确实需要共享所有权时才使用。

tuple 中具有唯一类型的元素可以通过其类型被访问。

```cpp
tuple<string, int, double> t1 {"shark", 123, 3.14};

// 方式 1
string fish = get<0>(t1);
int count = get<1>(t1);
double price = get<2>(t1);

// 方式 2
auto fish = get<string>(t1);
auto count = get<int>(t1);
auto price = get<double>(t1);
```

## C++ Core Guidelines


## 视频参考

[隐式转换]

```cpp
class Entity 
{
private:
    std::string m_Name;
    int m_Age;
public: 
    // explicit Entity(const std::string& name) 
    Entity(const std::string& name) 
        : m_Name(name), m_Age(8)  { }
    
    // explicit Entity(int age) 
    Entity(int age) 
        : m_Name("Unknown"), m_Age(age)  { }
};

// 合法 一次隐式转换为 Entity, 通过 explicit 禁用
Entity a = std::string("maze");
Entity b = 22;          // 隐式转换
Entity b = (Entity)22;  // 显式转换
```
