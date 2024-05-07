# [未完成] C++ 类型


TODO: 本文内容归到 "语法" 和 "模板"

## 转换规则

### 隐式

### 显式

## 常用类型转换

### 1.1 数组转换为 vector

1. 方式一，定义 vector 时直接赋值：

    ```cpp
    vector<char> v(array, array + sizeof(array))
    ```

2. 方式二，先定义 vector，再使用 memcpy 将 array 的值拷贝到 vector:

    ```cpp
    vector<int> v(N)
    memcpy(&V[0], array, sizeof(array))
    ```

### 1.2 vector 转换为数组

使用 memcpy 将 vector 中的数据拷贝到数组中：

```cpp
vector<char> vec(10, 'c');
char charary[vec.size()];
//vector 全部转到数组
memcpy(charary, &vec[0], vec.size() * sizeof(vec[0]));
```

### 1.3 更通用的方法

使用模板和智能指针。

```cpp
#include <vector>
#include <memory>
#include <fmt/core.h>
#include <fmt/ranges.h>

// 非智能指针版本
// template <typename T>
// T* vectorToArray(const std::vector<T>& vec) {
//     T* arr = new T[vec.size()];
//     for (std::size_t i = 0; i < vec.size(); ++i) {
//         arr[i] = vec[i];
//     }
//     return arr;
// }

template <typename T>
std::unique_ptr<T[]> vectorToArray(const std::vector<T>& vec) {
    std::unique_ptr<T[]> arr(new T[vec.size()]);
    for (std::size_t i = 0; i < vec.size(); ++i) {
        arr[i] = vec[i];
    }
    return arr;
}

template <typename T>
std::vector<T> arrayToVector(const T* arr, std::size_t size) {
    std::vector<T> vec(arr, arr + size);
    return vec;
}

// C 风格的函数，接受数组和数组大小作为参数
void processArray(int* arr, std::size_t size) {
    for (std::size_t i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}

int main() {
    // 创建一个 vector
    std::vector<int> vec = {1, 2, 3, 4, 5};
    // 将 vector 转为数组
    auto arr = vectorToArray(vec);
    processArray(arr.get(), vec.size());

    fmt::print("{}\n", fmt::join(arr.get(), arr.get() + vec.size(), " "));
    // 再将数组转回 vector
    std::vector<int> vec2 = arrayToVector(arr.get(), vec.size());
    fmt::print("{}\n", vec2);
    return 0;
}
```

`std::unique_ptr<T[]>` 是 `std::unique_ptr` 的一个特化版本，用于管理动态分配的数组，T[] 表示元素类型为 T 的数组。get 方法用于获取被管理对象的实际指针，可以传给 C 风格函数或者 C 函数。

> std::unique_ptr 有两个版本：
>
> 1. 管理单个对象（例如以 new 分配）
> 2. 管理动态分配的对象数组（例如以 new[] 分配）

### string 和 const char* 



## 参考

1. [C++ 中 vector 和数组的互相转换](https://www.jianshu.com/p/fc15781df71f)
2. [c++ 中 vector 和数组的互相转换](http://bcoder.com/others/convert-between-vector-and-array-in-cplusplus)
3. [std::unique_ptr](https://zh.cppreference.com/w/cpp/memory/unique_ptr)

