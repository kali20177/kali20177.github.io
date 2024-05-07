# Whisper 语音模型使用


whisper 是 openai 开源的可离线使用的语音识别模型。目前的缺点是只能产生英文的字幕文件。

## Windows

Windows 上使用的优势是能调用 Nvidia 显卡加速。

下载：

1. [WhisperDesktop](https://github.com/Const-me/Whisper/releases/download/1.12.0/WhisperDesktop.zip)
2. ggml 语音模型，可以通过[国内镜像站点](https://hf-mirror.com/ggerganov/whisper.cpp/tree/main)下载，文件越大，识别效果越好，耗时和占用资源越多。

使用：

1. 解压 WhisperDesktop 后点击 WhisperDesktop.exe 运行，并选择下载好的模型文件。

    ![import ggml](/whisper/import_ggml.png)

2. 在 Advanced 中选择独显。

    ![nv](/whisper/nv.png)

3. 选择待识别的 mp3 文件，如果是视频文件可以先使用 ffmpeg 转换成 mp3 后再进行操作。选择产生 "SubRip subtitles"文件（SRT 字幕），点击 Transcribe 开始生成字幕。

    ![run](/whisper/run.png)

    > 图中的 "Translate" 选项是指视频是其他语言，需要转成英文，而非直接产生其他语言的字幕。

4. 最后产生 srt 字幕文件后，在 vlc 中选择并添加即可。实测使用 RTX2060 + ggml-large-v2 模型的情况下，产生一个多小时讲座的字幕花费 17 分钟左右。

    ![result](/whisper/result.png)

## MacOS 
