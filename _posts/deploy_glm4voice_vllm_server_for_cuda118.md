
## 系统配置

```
OS: Ubuntu 20.04.5 LTS (Focal Fossa)
GPU: A100-SXM-80G * 1
CUDA: 11.8  驱动版本: 470.129.06 
```

最终版本 
```
Python: 3.11.10 
torch: 2.3.0+cu118
vllm: 5.0.1
```

## 搭建基础环境 

**安装pyenv**
```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
```

编辑 ~/.bashrc，尾部添加 pyenv 配置 
```log
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
```
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
```
# 一般pip源不包含torch的cuda细分版本，建议从 pytorch 官网下载  
# 到 https://download.pytorch.org/whl/cu118/torch，手动下载 torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torchaudio==2.3.0+cu118 --index-url https://download.pytorch.org/whl/cu118
# 到 https://download.pytorch.org/whl/cu118/torchvision，手动下载 torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl
```

新建constraint.txt，写入
```log
torch==2.3.0+cu118
torchaudio==2.3.0+cu118
torchvision==0.18.0+cu118
```
保存退出

安装 vllm 
```
# 手动下载 vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl
# 下载地址 https://github.com/vllm-project/vllm/releases/download/vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl
pip install vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl -c constraint.txt
```

**注意**: 
1. torch, torchaudio, torchvision 和 vllm 的版本必须匹配 cuda 版本和 python 版本。具体匹配关系可以问ChatGPT
2. 安装 vllm 和下面 requirements 时加上 -c contraints.txt 约束，否则 torch 会被替换，导致和 cuda 不兼容 

安装glm4voice依赖 
```bash
git clone --recurse-submodules https://github.com/THUDM/GLM-4-Voice
cd GLM-4-Voice
pip install -r requirements.txt -c constraints.txt

# 安装剩余依赖 
pip install accelerate 
sudo apt install ffmpeg 
```

## 启动服务 

这里用 vllm 加速版 

到 glm4voice的dev分支下载 vllm_model_server.py 
```
python vllm_model_server.py --host localhost --model-path <存放glm-4-voice-9b的path> --port 10000 --dtype bfloat16 --device cuda:0
```



