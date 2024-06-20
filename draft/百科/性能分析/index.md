# 性能分析


## 火焰图

使用 Linux 的性能分析器 perf 采集性能数据。

```bash
sudo apt-get install linux-tools-common linux-tools-generic linux-tools-`uname -r`
```

查找需要监视的程序 PID 号，并使用 perf 记录：

```bash
# 获取程序进程号
ps aux | grep XXX
# 监视性能，每秒 99 次，持续 120 秒
sudo perf record -F 99 -p PID_NUM -g -- sleep 120
```

执行后会在当前目录下产生 `perf.data` 文件，无需等待被监视程序结束，perf 被中断也能生成结果文件。进一步和处理结果：

```bash
perf script > out.perf
```

默认情况下使用 perf 需要 sudo 权限，通过下列设置修改执行权限：

```bash
# 允许非特权用户进行内核分析和访问 CPU 事件
echo 0 | sudo tee /proc/sys/kernel/perf_event_paranoid
sudo sh -c 'echo kernel.perf_event_paranoid=0 >> /etc/sysctl.d/local.conf'
# 启用内核模块符号解析以供非特权用户使用
echo 0 | sudo tee /proc/sys/kernel/kptr_restrict
sudo sh -c 'echo kernel.kptr_restrict=0 >> /etc/sysctl.d/local.conf'
```

需要使用项目[FlameGraph](https://github.com/brendangregg/FlameGraph) 中的 perl 脚本生成 svg 文件，可视化性能数据。

```bash
# On-CPU CPU 火焰图
./stackcollapse-perf.pl out.perf > out.folded
# 或者 Off-CPU IO 火焰图
./stackcollapse.pl out.perf > out.folded
```

生成 svg 图像：

```bash
./flamegraph.pl out.folded > result.svg
```

对于两次采集的不同数据，可以使用 `difffolded.pl` 生成差异火焰图。

```bash
./difffolded.pl out1.folded out2.folded | ./flamegraph.pl > diff.svg
```

浏览器打开可以进行交互。

火焰图纵向深度表示调用栈的深度，自底向上。横向表示抽象数，如果一个函数越宽，代表采样时被命中的次数越多，该函数在程序实际执行时间越长。

如果顶层函数存在平顶，表示该函数可能存在性能瓶颈。

## 参考

1. [Linux Perf](https://weedge.github.io/perf-book-cn/zh/chapters/7-Overview-Of-Performance-Analysis-Tools/7-4_Linux_perf_cn.html)
2. [如何读懂火焰图？](https://ruanyifeng.com/blog/2017/09/flame-graph.html)
3. [火焰图性能分析](https://ifun.dev/post/flamegraph/)

