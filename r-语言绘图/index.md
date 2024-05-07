# R 语言绘图


## 需要安装的包

安装命令：

```text
install.packages('ggplot2', repos = 'https://mirrors.tuna.tsinghua.edu.cn/CRAN')
```

* readxl
* gridExtra
* ggsci
* ggpubr

## 散点图绘制

如果需要绘制多组散点图，可以先把数据放在一个 excel 表中，整体导入数据框。每列数据名称最好不要使用中文。

```R
library(ggplot2)
library(readxl)

df <- read_xlsx("data.xlsx")
p <- ggplot(df) + geom_point(aes(x = Vol1, y = Disp1), color="#5D567E") + geom_point(aes(x = Vol2, y = Disp2), color="#C25E6C") + labs(x="vol",y="disp",title="compare")
```

效果：

![img](/R/Rplot.png)

颜色选择可以访问 [colorspace](https://mycolor.space/) 网站使用生成的配色。

