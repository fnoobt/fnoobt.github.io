---
title: 本地音视频转文字
author: fnoobt
date: 2023-05-04 20:31:00 +0800
categories: [AI,AI教程]
tags: [ai,whisper]
---

[Whisper](https://github.com/openai/whisper) 是 OpenAI 开发的一个通用开源免费的语音识别模型。它是在一个大型的不同音频数据集上训练出来的，也是一个多任务模型，可以进行多语言语音识别(multilingual speech recognition)、语音翻译(speech translation)和语言识别(language identification)。由于官方版本的 Whisper 基本依赖于命令行运行，所以有了各种封装软件，比如 Buzz、WhisperDesktop 等

[WhisperDesktop](https://github.com/Const-me/Whisper) 可以在 Windows 上通过可视化页面调用 Whisper.cpp 项目所提供的 GGML 格式模型运行的 Whisper 版本，从而实现本地电脑的音视频转文字！

## 部署 WhisperDesktop

### 下载 WhisperDesktop

1. 到 [WhisperDesktop发布页面](https://github.com/Const-me/Whisper/releases) 下载最新版，一般文件名为 `WhisperDesktop.zip`。
2. 软件为绿色软件，下载的文件解压即可点击运行

### 下载语音模型

可以从官方[huggingface](https://huggingface.co/ggerganov/whisper.cpp/tree/main)处下载 GGML 格式的 Whisper 模型。模型越大对显存要求就也越大，速度也越慢，一般使用 medium 模型（ggml-medium.bin）就够了。

Whisper 有五种型号模型，其中四种为纯英文版本，可按需权衡速度和准确性进行选择。以下是 OpenAI 官方给出的各种模型的大小和性能比较（实际上现在模型大了很多）。

| 大小   | 参数   | 纯英文型号 | 多语言模型 | 必需的显存 | 相对速度 |
|--------|--------|------------|------------|------------|----------|
|  tiny  |  39 M  |   tiny.en  |    tiny    |    ~1 GB   |   ~32x   |
|  base  |  74 M  |   base.en  |    base    |    ~1 GB   |   ~16x   |
|  small |  244 M |  small.en  |    small   |    ~2 GB   |    ~6x   |
| medium |  769 M |  medium.en |   medium   |    ~5 GB   |    ~2x   |
|  large | 1550 M |     N/A    |    large   |   ~10 GB   |    1x    |

## 使用 WhisperDesktop
双击打开 WhisperDesktop.exe，加载刚刚下载的语音模型

选择视频文件、视频语言、输出文件格式和位置，点击<kbd>Transcribe</kbd>即可进行转换

输出格式(Output Format)
- None: 无
- Text file: 文本文件
- Text with timestamps: 带时间戳的文本文件
- SubRip subtitles：SubRip字幕（srt）
- WebVTT subtitles: WebVTT字幕（vtt）

> 注意：WhisperDesktop 翻译功能只支持翻译为英文，建议采用谷歌翻译等进行翻译。
{: .prompt-tip }

## 文本翻译（谷歌翻译）

1. 复制文本内容到新建的.docx文件中
2. 将该文件上传到[谷歌文档翻译](https://translate.google.com/?sl=auto&tl=zh-CN&op=docs)上传中进行翻译
3. 翻译完成后下载对应的.docx文件，复制内容即可。

****

本文参考

> 1. [Whisper模型、WhisperDesktop及模型下载、使用](https://www.bilibili.com/read/cv23285680/)
> 2. [Buzz/|WhisperDesktop介绍](https://www.youtube.com/watch?v=e0u1nwoosKY)