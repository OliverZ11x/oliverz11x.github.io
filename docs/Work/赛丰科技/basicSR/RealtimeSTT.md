---
date created: 2025/5/30 11:10
date modified: 2025/5/30 15:16
---
# RealtimeSTT

1 G 显存左右，实时性高

## 关于项目

RealtimeSTT 监听麦克风并将语音转录为文本。

> **提示**：*请查看 [Linguflex](https://github.com/KoljaB/Linguflex)，这是 RealtimeSTT 的原始项目。它允许您通过语音控制环境，是目前功能最强大、最复杂的开源助手之一。*

它非常适合：

- **语音助手**
- 需要**快速准确**语音转文本转换的应用程序

![[IMG-2025-05-30-11-24-17.mp4]]

## 技术栈

该库使用：

- **语音活动检测**
  - [WebRTCVAD](https://github.com/wiseman/py-webrtcvad) 用于初始语音活动检测。
  - [SileroVAD](https://github.com/snakers4/silero-vad) 用于更准确的验证。
- **语音转文本**
  - [Faster_Whisper](https://github.com/guillaumekln/faster-whisper) 用于即时（GPU 加速）转录。
- **唤醒词检测**
  - [Porcupine](https://github.com/Picovoice/porcupine) 或 [OpenWakeWord](https://github.com/dscripka/openWakeWord) 用于唤醒词检测。

## 快速示例

### 打印所有说话内容：

```python  
from RealtimeSTT import AudioToTextRecorder  

def process_text(text):  
    print(text)  

if __name__ == '__main__':  
    print("等待提示'speak now'")  
    recorder = AudioToTextRecorder()  

    while True:  
        recorder.text(process_text)  
```  

### 输入所有说话内容：

```python  
from RealtimeSTT import AudioToTextRecorder  
import pyautogui  

def process_text(text):  
    pyautogui.typewrite(text + " ")  

if __name__ == '__main__':  
    print("等待提示'speak now'")  
    recorder = AudioToTextRecorder()  

    while True:  
        recorder.text(process_text)  
```  

*会将所有说话内容输入到您选择的文本框中*

### 功能

- **语音活动检测**：自动检测您何时开始和停止说话。
- **实时转录**：将语音实时转换为文本。
- **唤醒词激活**：可在检测到指定唤醒词时激活。

> **提示**：*查看 [RealtimeTTS](https://github.com/KoljaB/RealtimeTTS)，这是该库的输出对应工具，用于文本转语音功能。它们共同构成了围绕大型语言模型的强大实时音频包装器。*

## 技术栈

该库使用：

- **语音活动检测**
  - [WebRTCVAD](https://github.com/wiseman/py-webrtcvad) 用于初始语音活动检测。
  - [SileroVAD](https://github.com/snakers4/silero-vad) 用于更准确的验证。
- **语音转文本**
  - [Faster_Whisper](https://github.com/guillaumekln/faster-whisper) 用于即时（GPU 加速）转录。
- **唤醒词检测**
  - [Porcupine](https://github.com/Picovoice/porcupine) 或 [OpenWakeWord](https://github.com/dscripka/openWakeWord) 用于唤醒词检测。

*这些组件代表了前沿应用的“行业标准”，为构建高端解决方案提供了最现代和有效的基础。*

## 安装

```bash  
pip install RealtimeSTT  
```  

这将安装所有必要的依赖项，包括仅支持 CPU 的 PyTorch 版本。

尽管可以仅使用 CPU 安装运行 RealtimeSTT（此时使用“tiny”或“base”等小模型），但使用 CUDA 会获得更好的体验（请向下滚动）。

### Linux 安装

在安装 RealtimeSTT 之前，请执行：

```bash  
sudo apt-get update  
sudo apt-get install python3-dev  
sudo apt-get install portaudio19-dev  
```  

### MacOS 安装

在安装 RealtimeSTT 之前，请执行：

```bash  
brew install portaudio  
```  

### 支持 CUDA 的 GPU（推荐）

#### 更新 PyTorch 以支持 CUDA

根据您的具体 CUDA 版本，按照以下说明升级 PyTorch 安装以启用 GPU 支持。这对于希望通过 CUDA 功能提升 RealtimeSTT 性能的用户非常有用。

##### 对于 CUDA 11.8：

要更新 PyTorch 和 Torchaudio 以支持 CUDA 11.8，请使用以下命令：

```bash  
pip install torch==2.5.1+cu118 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu118  
```  

##### 对于 CUDA 12. X：

要更新 PyTorch 和 Torchaudio 以支持 CUDA 12. X，请执行以下操作：

```bash  
pip install torch==2.5.1+cu121 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121  
```  

将 `2.5.1` 替换为与您的系统和需求匹配的 PyTorch 版本。

### 可能需要的前置步骤

> **注意**：*要检查您的 NVIDIA GPU 是否支持 CUDA，请访问[官方CUDA GPU列表](https://developer.nvidia.com/cuda-gpus)。*
如果您以前未使用过 CUDA 模型，安装前可能需要执行一些额外步骤。这些步骤为 CUDA 支持和**GPU 优化**安装做准备。建议需要**更好性能**且拥有兼容 NVIDIA GPU 的用户执行以下步骤。要通过 CUDA 使用 RealtimeSTT 的 GPU 支持，请同时遵循以下步骤：

1. **安装 NVIDIA CUDA Toolkit**：
	- 选择 CUDA 11.8 或 CUDA 12. X Toolkit
		- 对于 12. X，访问 [NVIDIA CUDA Toolkit归档](https://developer.nvidia.com/cuda-toolkit-archive)并选择最新版本。
		- 对于 11.8，访问 [NVIDIA CUDA Toolkit 11.8](https://developer.nvidia.com/cuda-11-8-0-download-archive)。
	- 选择操作系统和版本。
	- 下载并安装软件。

2. **安装 NVIDIA cuDNN**：
	- 选择 CUDA 11.8 或 CUDA 12. X Toolkit
		- 对于 12. X，访问 [cuDNN下载](https://developer.nvidia.com/cudnn-downloads)。
			- 选择操作系统和版本。
			- 下载并安装软件。
		- 对于 11.8，访问 [NVIDIA cuDNN归档](https://developer.nvidia.com/rdp/cudnn-archive)。
			- 点击“Download cuDNN v 8.7.0 (November 28 th, 2022), for CUDA 11. X”。
			- 下载并安装软件。

3. **安装 ffmpeg**：

	> **注意**：*实际上可能不需要安装 ffmpeg 即可运行 RealtimeSTT* <sup> *感谢 jgilbert 2017 指出这一点*</sup>  
	您可以从 [ffmpeg网站](https://ffmpeg.org/download.html)下载适用于您操作系统的安装程序。  
	或使用软件包管理器：  
	- **在 Ubuntu 或 Debian 上**：

		```bash  
        sudo apt update && sudo apt install ffmpeg  
        ```  

	- **在 Arch Linux 上**：

		```bash  
        sudo pacman -S ffmpeg  
        ```  

	- **在 MacOS 上使用 Homebrew** ([https://brew.sh/](https://brew.sh/)):

		```bash  
        brew install ffmpeg  
        ```  

	- **在 Windows 上使用 Winget** [官方文档](https://learn.microsoft.com/en-us/windows/package-manager/winget/):

		```bash  
        winget install Gyan.FFmpeg  
        ```  

	- **在 Windows 上使用 Chocolatey** ([https://chocolatey.org/](https://chocolatey.org/)):

		```bash  
        choco install ffmpeg  
        ```  

	- **在 Windows 上使用 Scoop** ([https://scoop.sh/](https://scoop.sh/)):

		```bash  
        scoop install ffmpeg  
        ```  

## 快速开始

### 手动录制

录制的开始和停止由手动触发。

```python  
recorder.start()  
recorder.stop()  
print(recorder.text())  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

if __name__ == '__main__':  
    recorder = AudioToTextRecorder()  
    recorder.start()  
    input("按Enter键停止录制...")  
    recorder.stop()  
    print("转录结果: ", recorder.text())  
```  

### 自动录制

基于语音活动检测的录制。

```python  
with AudioToTextRecorder() as recorder:  
    print(recorder.text())  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

if __name__ == '__main__':  
    with AudioToTextRecorder() as recorder:  
        print("转录结果: ", recorder.text())  
```  

在循环中运行 recorder. Text 时，建议使用回调函数，允许异步运行转录：

```python  
def process_text(text):  
    print (text)  
    
while True:  
    recorder.text(process_text)  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

def process_text(text):  
    print(text)  

if __name__ == '__main__':  
    recorder = AudioToTextRecorder()  

    while True:  
        recorder.text(process_text)  
```  

### 唤醒词

检测到关键词后激活录制。将所需激活关键词的逗号分隔列表写入 wake_words 参数。您可以从以下列表中选择唤醒词：alexa, americano, blueberry, bumblebee, computer, grapefruits, grasshopper, hey google, hey siri, jarvis, ok google, picovoice, porcupine, terminator。

```python  
recorder = AudioToTextRecorder(wake_words="jarvis")  

print('说"Jarvis"然后说话。')  
print(recorder.text())  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

if __name__ == '__main__':  
    recorder = AudioToTextRecorder(wake_words="jarvis")  

    print('说"Jarvis"开始录制。')  
    print(recorder.text())  
```  

### 回调函数

您可以设置在不同事件触发时执行的回调函数（请参阅[配置](#configuration)）：

```python  
def my_start_callback():  
    print("录制开始！")  

def my_stop_callback():  
    print("录制停止！")  

recorder = AudioToTextRecorder(on_recording_start=my_start_callback,  
                               on_recording_stop=my_stop_callback)  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

def start_callback():  
    print("录制开始！")  

def stop_callback():  
    print("录制停止！")  

if __name__ == '__main__':  
    recorder = AudioToTextRecorder(on_recording_start=start_callback,  
                                   on_recording_stop=stop_callback)  
```  

### 输入音频块

如果您不想使用本地麦克风，请将 use_microphone 参数设置为 false，并使用以下方法提供 16 位单声道（采样率 16000）的原始 PCM 音频块：

```python  
recorder.feed_audio(audio_chunk)  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

if __name__ == '__main__':  
    recorder = AudioToTextRecorder(use_microphone=False)  
    with open("audio_chunk.pcm", "rb") as f:  
        audio_chunk = f.read()  

    recorder.feed_audio(audio_chunk)  
    print("转录结果: ", recorder.text())  
```  

### 关闭

您可以通过使用上下文管理器协议安全地关闭录制器：

```python  
with AudioToTextRecorder() as recorder:  
    [...]  
```  

或者手动调用 shutdown 方法（如果无法使用“with”语句）：

```python  
recorder.shutdown()  
```  

#### 独立示例：

```python  
from RealtimeSTT import AudioToTextRecorder  

if __name__ == '__main__':  
    with AudioToTextRecorder() as recorder:  
        [...]  
    # 或在不使用“with”时手动关闭  
    recorder.shutdown()  
```  
