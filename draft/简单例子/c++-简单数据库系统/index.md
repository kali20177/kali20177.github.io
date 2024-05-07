# [未完成] C++ 简单数据库系统


## 1. 说明

本文是阅读知乎网友教程 [如何用 C++ 实现一个简易数据库](https://www.zhihu.com/column/c_1472652536327389184) 的笔记。

## Rspec

一种基于 Ruby 的测试框架，可以用来测试 C++ 程序输出是否符合预期。

```ruby
describe XXX do
  it XXX do
       ......
  end
end
```

> 结构体偏移
>
>  ```cpp
>  #include <stddef.h>
>  size_t offsetof(type, member);
>  ``
>
> 宏定义：
> ```cpp
> #define size_of_attribute(Struct, Attribute) sizeof(((Struct *)0)->Attribute)
> ```
>
> **序列化**:把对象转化为可传输的字节序列过程称为序列化。
>
> **反序列化**：把字节序列还原为对象的过程称为反序列化。
>
> 序列化的目的是为了对象可以跨平台存储，和进行网络传输。

