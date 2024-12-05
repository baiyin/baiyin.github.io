## 基于 CUDA 11.8 的 GLM4Voice vLLM 服务部署

## 系统配置

```
OS: Ubuntu 20.04.5 LTS
GPU: A100-SXM-80G * 1
CUDA: 11.8  驱动版本: 470.129.06
```

最终版本

```
Python: 3.11.10
torch: 2.3.0+cu118
vllm: 5.0.1
```

安装过程挑战点

1. 系统 CUDA 版本 11.8 且不能升级，需要让 glm4voice 适配 cuda118 版本。
2. 当前 python 包默认支持 CUDA 版一般 12 以上，和 11.8 不兼容。**需要** 安装和 cuda11.8 对应的版本，小版本号也要对应。
3. 官方给出的 vllm for cuda11.8 安装示例有问题 
4. 安装 python 包依赖过程中容易替换掉前面已经安装的版本，需要注意安装顺序和约束 

## 搭建基础环境

**安装pyenv**

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
```

编辑 ~/.bashrc，尾部添加 pyenv 配置

```bash
export PYENV_ROOT="~/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

保存退出 ~/.bashrc

```bash
source ~/.bashrc
```

pyenv 测试

```bash
pyenv
```

**安装 Python**

```bash
# 安装 python 依赖的系统包
sudo apt update
sudo apt install -y \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    wget \
    curl \
    llvm \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev \
    git
# 开始安装 python
export v=3.11.10
# 用国内镜像地址
wget https://npmmirror.com/mirrors/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/
pyenv install $v
# 验证
pyenv versions # 提示 3.11.10 则表示安装成功
# 设置全局 Python 版本
pyenv global 3.11.10
```

**创建虚拟环境**

```bash
# 创建虚拟环境
pyenv virtualenv 3.11.10 glm4voice_env
# 激活虚拟环境
pyenv activate glm4voice_env
```

## 安装GLM4Voice

**模型下载**

国内 huggingface 太慢，用 modelscope 下载

安装 modelscope

```bash
pip install modelscope
```

下载模型

```python
from modelscope import snapshot_download

snapshot_download('ZhipuAI/glm-4-voice-9b')
snapshot_download('ZhipuAI/glm-4-voice-tokenizer')
snapshot_download('ZhipuAI/glm-4-voice-decoder')
```

**安装依赖包**

安装 torch, torchaudio, torchvision, xformers 

```bash
# 安装 torch 
# 一般pip源不包含torch的cuda细分版本，建议从 pytorch 官网下载
# 到 https://download.pytorch.org/whl/cu118/torch 手动下载 torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl

# 安装 torchaudio 
pip install torchaudio==2.3.0+cu118 --index-url https://download.pytorch.org/whl/cu118

# 安装 torchvision 
# 到 https://download.pytorch.org/whl/cu118/torchvision 手动下载 torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl

# 安装 xformers 
# 到 https://download.pytorch.org/whl/cu118/xformers 手动下载 xformers-0.0.26.post1+cu118-cp311-cp311-manylinux2014_x86_64.whl 
pip install xformers-0.0.26.post1+cu118-cp311-cp311-manylinux2014_x86_64.whl 
```

测试

```bash
# python 测试 torch 
>>> import torch
>>> import torchaudio
>>> import torchvision
>>> print("CUDA Available:", torch.cuda.is_available())
# True 表示安装成功 
```

```bash
# shell 测试 xformers 
python -m xformers.info
# 输出中包含
# pytorch.cuda: available 
# 则说明成功安装 cuda 版本 xformers 
```

**注意**

1. torch, torchaudio, torchvision, xformers 版本必须和 cuda 版本和 python 版本匹配。具体匹配关系问 ChatGPT 或 Google 
2. xformers 需要单独安装，否则通过 vllm 依赖安装的版本不支持 cuda 

新建constraints.txt，写入

```bash
torch==2.3.0+cu118
torchaudio==2.3.0+cu118
torchvision==0.18.0+cu118
xformers==0.0.26.post1+cu118
```

保存退出

安装 vllm

```bash
# 手动下载 vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl
# 下载地址 https://github.com/vllm-project/vllm/releases/download/v0.5.1/vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl
pip install vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl -c constraints.txt
```

**注意**:

1. vllm 的版本必须和系统 cuda 和 python 版本匹配。具体匹配关系问 ChatGPT 或 Google，或参考 https://docs.vllm.ai/en/latest/getting_started/installation.html
3. 安装 vllm 必须加 -c contraints.txt 约束，否则 torch 会被覆盖，导致和 cuda 不兼容

安装glm4voice依赖

```bash
git clone --recurse-submodules https://github.com/THUDM/GLM-4-Voice
cd GLM-4-Voice
pip install -r requirements.txt -c ../constraints.txt

# 安装剩余依赖
pip install accelerate
sudo apt install ffmpeg
```

**注意**

1. 安装 requirements 要加 -c contraints.txt 约束，否则 torch 会被替换导致和 cuda 不兼容

## 启动服务

这里用 vllm 加速版

1. 到 github [glm4voice 的 dev分支](https://github.com/THUDM/GLM-4-Voice/tree/dev)下载 vllm_model_server.py
2. 启动服务 

```bash
# glm-4-voice-9b 在 modelscope 指定的目录下 
python vllm_model_server.py --host localhost --model-path /path/to/the/glm-4-voice-9b --port 10000 --dtype bfloat16 --device cuda:0

python web_demo.py --tokenizer-path /path/to/the/glm-4-voice-tokenizer --model-path /path/to/the/glm-4-voice-9b --flow-path /path/to/the/glm-4-voice-decoder 
```
