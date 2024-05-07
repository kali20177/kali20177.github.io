# Llama.cpp 简单体验


## 编译

建议在 WSL2 上操作，可以直接调用 N 卡加速，且无需复杂的配置依赖过程，首先安装 CUDA TOOLKIT 和 ccache：

```bash
sudo apt install nvidia-cuda-toolkit ccache
nvcc --version
```

下载 llama.cpp 并编译：

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama
cmake -B build -DLLAMA_CUDA=ON
cmake --build build --config Release
```

## 导入模型

在 llama 中新建文件夹存放中文模型：

```bash
mkdir -p "zh-models/7B/" 
```

这里使用 GitHub 项目 Chinese-LLaMA-Alpaca-2 的模型，[下载位置](https://hf-mirror.com/hfl/chinese-llama-2-7b-gguf/tree/main)，选择 `ggml-model-q4_0.gguf` 下载，完成后移动到上面新建的文件夹。

在根目录新建下面的脚本文件：

```bash
#!/bin/bash

# temporary script to chat with Chinese Alpaca-2 model
# usage: ./chat.sh alpaca2-ggml-model-path your-first-instruction

SYSTEM_PROMPT='You are a helpful assistant. 你是一个乐于助人的助手。'

MODEL_PATH=$1
FIRST_INSTRUCTION=$2

./build/bin/main -m "$MODEL_PATH" \
-ngl 31 --color -i -c 4096 -t 8 --temp 0.5 --top_k 40 --top_p 0.9 --repeat_penalty 1.1 \
--in-prefix-bos --in-prefix ' [INST] ' --in-suffix ' [/INST]' -p \
"[INST] <<SYS>>
$SYSTEM_PROMPT
<</SYS>>

$FIRST_INSTRUCTION [/INST]"
```

使用 GPU 推理需要参数 `-ngl`，实测使用 RTX2060 的情况下 31 网上就已经无法正常输出结果了。运行脚本：

```bash
chmod +x chat.sh
./chat.sh zh-models/7B/ggml-model-q4_0.gguf '请列举5条文明乘车的建议'
```

实测下来大部分问题都在胡言乱语。
