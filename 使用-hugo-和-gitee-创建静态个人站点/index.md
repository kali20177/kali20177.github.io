# 使用 hugo 和 gitee 创建静态个人站点


## 1. 安装

通过应用商店下载 WSL，使用包管理器安装 Hugo 即可。

## 2. 创建项目

```bash
hugo new site mysite     #项目名称 mysite
```

进入 `mysite` 目录，结构和功能如下：

```bash
  mysite
    ├── archetypes # markdown文章的模版
    ├── config.toml # 配置文件
    ├── content # 网站内容，主要保存文章
    ├── data # 生成网站可用的数据文件，可用在模版中
    ├── layouts # 生成网站时可用的模版
    ├── public # 通过hugo命令生成的静态文件，主要发布这个
    ├── resources # 通过hugo命令一起生成的资源文件
    ├── static # 静态文件，比如文章中的图片/视频文件、缩略图等
    └── themes # 存放hugo主题
```

## 3. 挑选主题

搜索 Hugo 主题，在主题网站挑选合适主题后，git clone 主题项目到路径 `mysite/themes` 下。

## 4. 根据主题文档进行操作

一般需要将主题提供的 config.toml 文件，按照作者提示编辑修改后，覆盖 hugo 初始默认的配置文件。

## 5. 编辑文章

```bash
hugo new posts/first.md
```

文章会被创建到目录 `content/posts` 下，使用 markdown 语法编辑。

## 6. markdown 部分语法

### 6.1 表格

设置表格对齐方式：
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

### 6.2 强调格式

| 方式 | 效果 |
| :-----:| :----: |
| \*abc\* |*abc* |
| \*\*abc\*\* | **abc** |
| \*\*\*abc\*\*\* |***abc*** |
| \~\~abc\~\~ |~~abc~~ |

## 7. 启用 hugo server 查看效果

```bash
hugo server -D
```

![hugo server D](/hugo_gitee/hugo_server.png)

点击链接能够在浏览器中查看效果。

## 8. 图像

* 配置文件的图像放在主题文件夹的 `static` 中
* 文章的图像放在根目录的 `static` 文件夹中，或者使用图床链接

## 9. gitee 配置

由于 github 访问问题，建议部署到 gitee 确保站点能被正常访问。

在 gitee 上新建一个仓库，仓库名称和 gitee  的用户名相同。生成的个人站点链接将会是`gitee用户名.gitee.io` 。

## 10. 编译

执行以下命令编译站点：

```bash
hugo --theme=theme_name --baseURL="https://gitee用户名.gitee.io" -D
```

能够看到在项目文件夹中生成了 `public` 文件夹。

## 11. 将 public 推到 gitee 仓库

```bash
cd public
git init
git add .
git commit -m "test"
git remote add origin '仓库地址'
git push -u origin master
```

## 12. 在 gitee 仓库服务中开启 gitee pages 服务

点击服务 --> Git pages，按照流程开通，点击部署。打开网页查看效果：

![hugo 效果](/hugo_gitee/hugo_res.png)

## 13. 后续维护

存放源码的仓库被托管到 github 后，themes 文件夹下的项目由于有 .git 目录，会被设置成子模块。clone 仓库到本地后需要下载主题到 theme 文件夹。

```bash
git clone https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

或者从一开始下载时就一同下载子模块

```bash
​git clone --recursive https://github.com/xxx
```

再或者直接删除主题项目的 .git 目录，直接 push 所有文件到仓库。

## 参考

1. [Hugo - 10 分钟搭建 & 部署个人网站/博客，简历中的博客网站怎么建](https://www.bilibili.com/video/BV1x64y117PX?spm_id_from=333.999.0.0)
2. [Hugo + Markdown + MathJax](https://blog.lrtfm.com/2020/05/2020-05-03-hugo-markdown-mathjax)
3. [Markdown 表格：对齐方式，表格内换行、加粗、斜体，合并单元格](https://blog.csdn.net/lixiaoting94/article/details/103876601)

