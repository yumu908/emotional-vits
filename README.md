# Emotional VITS

[![Hugging Face Spaces](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue)](https://huggingface.co/spaces/innnky/nene-emotion)

在线demo ↑↑↑  [bilibili demo](https://www.bilibili.com/video/BV1Vg411h7of)

数据集无需任何情感标注，通过[情感提取模型](https://github.com/audeering/w2v2-how-to) 提取语句情感embedding输入网络，实现情感可控的VITS合成

## 模型结构

+ 相对于原版VITS仅修改了TextEncoder部分
  ![image-20221029104949567](resources/out.png)

## 模型的优缺点介绍

该模型缺点：

+ 推理时需要指定一个音频作为情感的**参考音频**才能够合成音频，而模型本身**并不知道**“激动”、“平静”这类表示情绪的词语对应的情感特征是什么。
+ 对于只有一个角色的模型，可以通过**预先筛选**的方式，即手动挑选几条“激动”、“平静”、“小声”之类的音频，手动实现情感文本->情感embedding的对应关系 （这个过程可以用[聚类算法](emotion_clustering.ipynb)
  简化筛选）
+ 对于有**多个角色**的模型，上述预筛选的方式有**局限性**，因为例如同样对于“平静”这一个情感而言，不同角色对应的情感embedding可能会不同，导致建立情感文本->情感embedding的映射关系很繁琐，很难通过一套统一的标准去描述不同角色之间的相似情感

该模型的优点：

+ **任何**普通的TTS数据集均可以完成情感控制。**无需**手动打情感标签。
+ 由于在训练时候并没有指定情感的文本与embedding的对应关系，所有的情感特征embedding均在一个连续的空间内
+ 因此理论上对于任意角色数据集中出现的情感，推理时均可以通过该模型实现合成，只需要输入目标情感音频对应的embedding即可，而不会受到情感分类数量限制

## 快速挑选各个情感对应的音频

可以使用 **聚类算法** 自动对音频的情感embedding进行分类，大致上可以区分出**情感差异较大**的各个类别，具体使用请参考 [emotion_clustering.ipynb](emotion_clustering.ipynb)

## Pre-requisites

0. Python >= 3.6
1. Clone this repository
2. Install python requirements. Please refer [requirements.txt](requirements.txt)
3. prepare datasets
4. Build Monotonic Alignment Search and run preprocessing if you use your own datasets.

```sh
# Cython-version Monotonoic Alignment Search
cd monotonic_align
python setup.py build_ext --inplace

# Preprocessing (g2p) for your own datasets. Preprocessed phonemes for nene have been already provided.
python preprocess.py --text_index 2 --filelists filelists/train.txt filelists/val.txt --text_cleaners japanese_cleaners


```

1. extract emotional embeddings, this will generate *.emo.npy for each wav file.

```sh
python emotion_extract.py --filelists filelists/train.txt filelists/val.txt
```

## Training Exmaple

```sh

# nene
python train_ms.py -c configs/nene.json -m nene

# if you are fine tuning pretrained original VITS checkpoint ,
python train_ms.py -c configs/nene.json -m nene --ckptD /path/to/D_xxxx.pth --ckptG /path/to/G_xxxx.pth

```

## Inference Example

See [inference.ipynb](inference.ipynb) or use [MoeGoe](https://github.com/CjangCjengh/MoeGoe)

## 如果需要同时支持 中、英、日（C/J/E） 三种语言的 VITS 预训练模型，目前有以下几个最火的优秀选择

1. Plachta 的中英日三语 VITS 模型（最经典）
   模型项目：Plachta/VITS-Umamusume-voice-synthesizer
   Hugging Face 搜索名：Plachta/VITS-Umamusume-voice-synthesizer (或者在 Hugging Face 搜索 VITS Umamusume)
   特点：这是目前社区内使用最广泛的动漫中英日三语模型，支持通过语言标签（如 [ZH] 代表中文，[JA] 代表日语，[EN] 代表英语）来控制生成的语言，发音准确。
2. MeloTTS (由 MyShell 开源，极速且高质量)
   模型项目：MeloTTS
   Hugging Face 搜索名：myshell-ai/MeloTTS
   特点：非常快速且对 CPU 友好的 TTS 架构（由 VITS 改良而来），原生支持中文、英文、日语、西班牙语、法语和韩语。
3. GPT-SoVITS（中英日克隆效果最好）
   项目名：GPT-SoVITS
   Hugging Face 搜索名：RVC-Boss/GPT-SoVITS
   特点：目前中文社区最强大的跨语言零样本声音克隆模型。只需要提供 5 秒的参考音频，就可以让模型用非常自然的中、英、日三语进行配音。
   💡 替换与配置提示
   如果您下载了中英日三语模型，在配置模型时需要注意：

Text Cleaners 必须修改：
三语模型通常需要配合 chinese_cleaners、japanese_cleaners 的混合体，或者由项目作者专门定制的 c_j_e_cleaners（例如 vtubers 配置文件中的 text_cleaners 需要修改为支持多语言的名称）。
符号表（Symbols）匹配：
多语言模型的音素符号表（symbols）会比单语言模型庞大得多，加载模型时必须使用该模型自带的 symbols.py 或在其 config.json 中定义的符号表，否则会报维度不匹配的错误。

如果您是指 **想听听合成效果的示范音频** ，或者 **需要用于提取情感的示范音频** ：

### 1. 哪里可以听示范音频（网页端）

赛马娘多语言模型（包括您目前打开配置的  **Agnes Digital / 爱丽数码** ）在 Hugging Face Space 上有官方的在线试听和合成 Demo：

+ **在线试听 Demo 地址** ：[Plachta/VITS-Umamusume-voice-synthesizer](https://huggingface.co/spaces/Plachta/VITS-Umamusume-voice-synthesizer)
+ 在这个网页里，您可以直接输入中英日三种语言，选择角色并生成音频，听一听实际的合成效果。

---

### 2. 情感提取的“示范音频”（可以直接用您身边的任意音频）

因为这个项目使用的是 **情感可控的 VITS（Emotional VITS）** ，它不需要特定的“官方示范音频”来提取情感。

+ 代码中的 `emotion_extract.py` 能够 **从任意 `.wav` 文件中提取情感** 。
+ 只要是您自己录制的、或者下载的任何 WAV 音频（比如愤怒的、高兴的、哭泣的声音），都可以直接作为情感参考输入！
