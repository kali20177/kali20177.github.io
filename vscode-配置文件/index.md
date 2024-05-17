# VSCode 配置文件和文档生成


## 1. Json 文件

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，最初由 Douglas Crockford 提出，作为 JavaScript 中对象字面量的一种表示方式，但已经成为一种独立于语言的格式。JSON 格式易于阅读和编写，同时也易于解析和生成。

优势：

- 易读性
- 易解析性
- 轻量级
- 支持复杂数据结构

使用场景：

- 配置文件 
- API 响应
- 日志数据
- Web 开发

基本语法：

- 对象（Object）：使用花括号 {} 表示，键值对之间使用冒号分隔，键值对之间使用逗号分隔。例如：{"name": "John", "age": 30}
- 数组（Array）：使用方括号 [] 表示，数组元素之间使用逗号分隔。例如：[1, 2, 3, 4, 5]
- 字符串（String）：使用双引号 "" 表示。例如："Hello, World"
- 数字（Number）：可以是整数或浮点数。例如：10 或 3.14
- 布尔值（Boolean）：true 或 false
- 空值（null）：null

这个 JSON 表示一个包含个人信息的对象，包括姓名、年龄、是否是学生、所修课程和地址等。

```json
{
    "name": "John",
    "age": 30,
    "isStudent": false,
    "courses": ["Math", "Science", "History"],
    "address": {
    "street": "123 Main St",
    "city": "Anytown"
    }
}
```

大部分 VSCode 配置文件全部在 `.vscode` 文件夹下。一般有 `launch.json`，`tasks.json`，`setting.json` 和 `xxx.code-snippets` 等。JSON 是 Visual Studio Code 中大多数配置文件的首选格式，在 VSCode 中允许使用注释，带有自动提示。

## 2. 代码片段 Snippets

`xxx.code-snippets` 文件用于项目的自定义代码片段。例如对于 C/C++ 项目可设置如下定义：

```json
//custom.code-snippets
{
    "Create Header Guard": {
        "prefix": "ifndef",
        "scope": "c",
        "body": [
            "#ifndef ${1:HEADER_NAME_H_}",
            "#define ${1:HEADER_NAME_H_}",
            "",
            "// Your code here",
            "",
            "#endif // ${1:HEADER_NAME_H_}"
        ],
        "description": "Create a header guard"
    },

    "Class Declaration": {
        "prefix": "class",
        "scope": "cpp",
        "body": [
            "class ${1:ClassName} {",
            "public:",
            "\t${1:ClassName}() {",
            "\t\t// Constructor code here",
            "\t}",
            "\t~${1:ClassName}() {",
            "\t\t// Destructor code here",
            "\t}",
            "};"
        ],
        "description": "Class declaration template"
    },
    "For Loop": {
        "prefix": "for",
        "scope": "c, cpp",
        "body": [
            "for (int ${1:i} = 0; ${1:i} < ${2:condition}; ++${1:i}) {",
            "\t// Your code here",
            "}"
        ],
        "description": "For loop template"
    },
    "Doxygen Comment": {
        "prefix": "comment",
        "scope": "c, cpp",
        "body": [
            "/**",
            " * @brief ${1:brief description}",
            " * @date ${CURRENT_YEAR}.${CURRENT_MONTH}.${CURRENT_DATE}",
            " * @param ${2:param_name} ${3:Param description}",
            " * @param ${4:param_name} ${5:Param description}",           
            " * @return ${6:return_type} ${7:Return description}",
            " */"
        ],
        "description": "Doxygen style comment"
    }
}
```

编辑一条 snippet 包含下面几个部分：

1. "prefix" 表示输入哪些字符后弹出提示
2. "scope" 设置在哪些类型的文件中生效
3. "body" 代码片段内容
4. "description" 相当于注释，描述这个片段的作用

> 符号 `${digital}` 代表产生片段后，光标的插入点，后面的数字代表按下 tab 后光标跳转的顺序。
> 如果是 `${digital:word}` 则表示跳转到此处时 word 会被选中。
>
> `${CURRENT_YEAR}.${CURRENT_MONTH}.${CURRENT_DATE}` 能自动填充时间

![h_class_snippet](/VSCode_cofig/h_class_snippet.gif)

可以使用 [snippet-generator](https://snippet-generator.app/?description=&tabtrigger=&snippet=&mode=vscode) 辅助生成代码片段。

## 3. 任务 Tasks

`tasks.json`` 机制旨在解决开发过程中的自动化任务管理问题。

- 自定义任务的集成：通过 tasks.json，开发者可以集成各种自定义的任务，比如编译、运行测试、代码质量检查等，以及任何其他可以通过命令行执行的任务。这样，开发者可以在 Visual Studio Code 中统一管理和执行这些任务，而无需离开编辑器。
- 跨平台的任务配置：tasks.json 支持在一个配置文件中定义跨平台的任务，因为它可以使用不同的命令和参数来适应不同的操作系统。这样，开发者可以在不同的操作系统上使用相同的配置文件来定义任务，而不需要为每个操作系统都单独配置任务。
- 问题匹配器的集成：tasks.json 支持配置问题匹配器，这使得开发者可以定义任务输出的格式，并将输出中的错误、警告等信息映射到 Visual Studio Code 中的问题列表，从而更轻松地查看和处理任务输出中的问题。

例如将圈复杂度检查工具 lizard 集成到 VSCode 的任务中。

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "lizard",
            "type": "shell",
            "command": "lizard \"${file}\"",
            "group": {
                "kind": "none",
                "isDefault": true
            },
            "presentation": {
                "reveal": "always",
                "panel": "new"
            },
            "problemMatcher": []
        }
    ]
}
```

编辑一个 task 包含下面几个部分：

1. label：任务的标签，用于在 Visual Studio Code 中显示任务的名称。
2. type：任务的类型，通常是 "shell"，表示该任务是通过 shell 命令来执行的。
3. command：要执行的命令，可以包含变量（如 ${file}）来表示文件路径等信息。
4. group：任务分组信息，包括 kind 和 isDefault 两个属性。kind 属性用于指定任务的分组类型，例如 "build"、"test" 等，而 isDefault 属性用于指示该任务是否为默认任务。
5. presentation：任务的呈现（presentation）信息，包括 reveal 和 panel 两个属性。reveal 属性指定任务执行时是否自动显示输出，而 panel 属性指定任务输出的面板类型。
6. problemMatcher：问题匹配器信息，用于定义任务输出中错误、警告等信息的匹配规则。

任务编辑完成后，使用快捷键 `Ctrl+Shift+P` 输入 tasks，选择"运行任务"，即可看到刚才设置的任务。

![lizard_task](/VSCode_cofig/lizard_task.gif)

## 4. 调试 launch

VSCode 的调试配置信息被放置于 `launch.json` 中。C/C++ gdb 的配置：

```json
{
  "configurations": [
    {
      "name": "(gdb) 启动",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/demo",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}/build/",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "miDebuggerPath": "gdb",
      "setupCommands": [
          {
              "description": "为 gdb 启用整齐打印",
              "text": "-enable-pretty-printing",
              "ignoreFailures": true
          },
          {
              "description": "将反汇编风格设置为 Intel",
              "text": "-gdb-set disassembly-flavor intel",
              "ignoreFailures": true
          }
      ]
  }
  ],
  "version": "2.0.0"
}
```

## 5. 工作区设置 settings

`settings.json` 是 Visual Studio Code 中用于配置编辑器外观，行为和功能的文件。例如编写本文时的配置：

```json
{
    // 不从下列目录中搜索关键词
    "search.exclude": {
        "themes/":true,
        "public/":true
    },
    // 不显示的文件/文件夹
    "files.exclude": {
        "themes/":true,
        "public/":true
    },
    // 目录布局方式
    "workbench.tree.enableStickyScroll":true
}
```

## 6. Doxygen 注释生成

安装 VSCode 插件 Doxygen Documentation Generator。在设置中搜索 doxygen 修改 author 和 email 字段。输入 `/**` 回车可自动产生符合 doxygen 的注释。生成的注释会根据后面的函数或者文件位置来判断。

![doxygen comment](/VSCode_cofig/doxygen_comment.gif)

## 7. VSCode Portable Mode

VSCode 也提供解压即用的版本 [Portable Mode](https://code.visualstudio.com/docs/editor/portable)，这个版本解压后进入解压的顶层目录，创建 data 文件夹，这样打开 VSCode 后，安装的插件和配置文件都会保存在这个文件夹中，更新时只需要替换其他部分即可。

缺点是不能正常使用右键菜单，可以通过修改注册表解决：

```
Windows Registry Editor Version 5.00
    
[HKEY_CLASSES_ROOT\*\shell\VSCode]
@="Open with Code"
"Icon"="E:\\VSCode\\Code.exe"
    
[HKEY_CLASSES_ROOT\*\shell\VSCode\command]
@="\"E:\\VSCode\\Code.exe\" \"%1\""
    
Windows Registry Editor Version 5.00
    
[HKEY_CLASSES_ROOT\Directory\shell\VSCode]
@="Open with Code"
"Icon"="E:\\VSCode\\Code.exe"
    
[HKEY_CLASSES_ROOT\Directory\shell\VSCode\command]
@="\"E:\\VSCode\\Code.exe\" \"%V\""
    
Windows Registry Editor Version 5.00
    
[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode]
@="Open with Code"
"Icon"="E:\\VSCode\\Code.exe"
    
[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode\command]
@="\"E:\\VSCode\\Code.exe\" \"%V\""
```

## 8. VSCode Profile

VSCode Profile 功能允许用户创建自定义设置的集合，包括插件、快捷键、设置等，可以快速切换项目或语言环境并与他人共享。

点击：管理（左下角齿轮）--> 配置文件 --> 创建配置文件，根据提示命名和选择需要导出的系列设置，可以保存为 xxx.code-profile 文件。

## 参考

1. [VSCode Snippets：提升开发幸福感的小技巧](https://juejin.cn/post/7076609496046370847)
2. [Snippets in Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets)
3. [Integrate with External Tools via Tasks](https://code.visualstudio.com/docs/editor/tasks)
4. [使用 Doxygen 构建文档](https://www.bookstack.cn/read/CMake-Cookbook/content-chapter12-12.1-chinese.md)
5. [Doxygen 笔记](https://doxygen-note.readthedocs.io/zh-cn/latest/tutorial/intro.html)
6. [Quick setup to use Doxygen with CMake](https://vicrucann.github.io/tutorials/quick-cmake-doxygen/)
7. [Profiles in Visual Studio Code](https://code.visualstudio.com/docs/editor/profiles)

