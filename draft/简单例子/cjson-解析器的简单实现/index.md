# [未完成] Cjson 解析器的简单实现


## 1. json 格式

一种数据交换格式，源于 JavaScript，可用于任何编程语言。
json 是树状结构，包含六种类型：

- null;
- boolean
- number
- string
- array
- object

**任务要求：**

1. 把 json 解析为一个树状结构
2. 提供访问接口
3. 把数据结构转换成 json 文本

> **头文件设计注意**
> include 防范，避免重复声明。
>
> ```c
> #ifndef LEPTJSON_H__
> #define LEPTJSON_H__
> /* ... */
> #endif /* LEPTJSON_H__ */
> ```

因为 C 语言没有 C++ 的命名空间（namespace）功能，一般会使用项目的简写作为标识符的前缀。通常枚举值用全大写（如 LEPT_NULL），而类型及函数则用小写（如 lept_type）。

在 cJSON 库中定义了如下结构体表示一个 json 数据：

```c
/* The cJSON structure: */
typedef struct cJSON
{
    /* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
    struct cJSON *next;
    struct cJSON *prev;
    /* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
    struct cJSON *child;

    /* The type of the item, as above. */
    int type;

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw */
    char *valuestring;
    /* writing to valueint is DEPRECATED, use cJSON_SetNumberValue instead */
    int valueint;
    /* The item's number, if type==cJSON_Number */
    double valuedouble;

    /* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    char *string;
} cJSON;
```

> JSON 语法子集：
>
> ```text
> JSON-text = ws value ws
> ws = *(%x20 / %x09 / %x0A / %x0D)
> value = null / false / true 
> null  = "null"
> false = "false"
> true  = "true"
> ```

那么第一行的意思是，JSON 文本由 3 部分组成，首先是空白（whitespace），接着是一个值，最后是空白。

第二行告诉我们，所谓空白，是由零或多个空格符（space U+0020）、制表符（tab U+0009）、换行符（LF U+000A）、回车符（CR U+000D）所组成。

第三行是说，我们现时的值只可以是 null、false 或 true，它们分别有对应的字面值（literal）。

