# [未完成] C++ 简单 Log


## 1. 说明

学校课题组祖传代码缺乏日志系统，监控过程信息非常麻烦，想自己实现一个简易的 Log 模块实现基本 Log 功能，顺带学习单例模式。代码参考自网络公开的博客和教程，并给出了链接。

## 2. 日志级别

项目本身要求不是很高，只考虑实现 info, warning, error 三个等级的日志信息。

## 3. 日志功能

* 获取时间
* 输出信息到日志文件或终端

## 4. 普通方法实现

### 4.1 时间获取

```cpp
std::string getCurrentTime()
{
    const time_t now = time(nullptr);
    char str[26];
    ctime_s(str, sizeof str, &now);
    return str;
}
```

### 4.2 logger 类

```cpp
class Log
{
public:
    const int LogLevelError = 0;
    const int LogLevelWaring = 1;
    const int LogLevelInfo = 2;

private:
    int m_LogLevel = LogLevelInfo; // 默认
public:
    void SetLevel(int level)
    {
        m_LogLevel = level;
    }

    void Error(const char* message)
    {
        if (m_LogLevel >= LogLevelError)
            std::cout << getCurrentTime() << "[Error]:" << message << std::endl;
    }

    void Warning(const char* message)
    {
        if (m_LogLevel >= LogLevelWaring)
            std::cout << getCurrentTime() << "[Warning]:" << message << std::endl;
    }

    void Info(const char* message)
    {
        if (m_LogLevel >= LogLevelInfo)
            std::cout << getCurrentTime() << "[information]:" << message << std::endl;
    }
};

int main()
{
    Log log;
    log.SetLevel(log.LogLevelWaring);
    log.Warning("Warning");
    log.Error("Error");
    log.Info("info");
}
```

## 5. 单例模式

设计需求：某个类只需要一个实例化对象。

典型场景：

1. 系统日志由一个日志管理器记录；
2. 线程池；
3. 全局游戏的窗口；

要求：

1. 将构造函数非 public;
2. 使用一个静态成员函数保存唯一实例；
3. 实现一个 public 方法，获取唯一实例；
4. 禁用移动构造和拷贝构造；

```cpp
class Singleton {
public:
    static Singleton& GetInstance() {
        static Singleton inst;
        return inst;
    }
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;

    // ...
private:
    Singleton() {
        // ...
    }
    // ...
};
```

参考陈硕老师 muduo 库中的 noncopyable 类，构造单例模式禁用拷贝构造和赋值操作时可以继承一个 noncopyable 类，对于阅读代码的人更友好。

```cpp
class noncopyable
{
public:
    noncopyable(const noncopyable&) = delete;
    void operator=(const noncopyable&) = delete;

protected:
    noncopyable() = default;
    ~noncopyable() = default;
};
```

## 参考

1. [C++ 新标准 004_不简单的“单例模式”](https://www.bilibili.com/video/BV1nP4y1J7HH?share_source=copy_web)

