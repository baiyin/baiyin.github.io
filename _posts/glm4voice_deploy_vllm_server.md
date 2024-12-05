
基础环境 

```
OS: Ubuntu 20.04.5 LTS (Focal Fossa)
GPU: A100-SXM-80G * 1
CUDA: 11.8  驱动版本: 470.129.06 
```

最终版本 
```
Python: 3.11.10 
torch: 2.3.0
```

## 创建步骤

安装pyenv 
```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
# 编辑 ~/.bashrc，配置 pyenv 配置，保存退出
  export PYENV_ROOT="~/.pyenv"
  export PATH="$PYENV_ROOT/bin:$PATH"
  eval "$(pyenv init --path)"
  eval "$(pyenv init -)"
  eval "$(pyenv virtualenv-init -)"
# 保存退出 ~/.bashrc
source ~/.bashrc 
# pyenv 测试 
pyenv 
```

## 安装 Python 
```
# 需要先把安装 python 所需要的系统包给安装上 
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
export v=3.11.10; 
# 注意下载目录和上面 pyenv 配置中配置的目录一致
# 适用国内镜像地址
wget https://npmmirror.com/mirrors/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/; 
pyenv install $v 
# 验证 
pyenv versions 
# 设置全局 Python 版本
pyenv global 3.11.10
```

## 创建虚拟环境 
```
pyenv virtualenv 3.11.10 glm4voice_env 
pyenv activate glm4voice_env
```

## 安装
模型下载 
```
# modelscope 临时配在文件中 
import os
# 设置 MODELSCOPE_CACHE 环境变量
os.environ["MODELSCOPE_CACHE"] = "/vepfs/baiyin/.cache/modelscope"

```

安装依赖包
```
# 因为一般pip源不包含 torch 细分版本，建议从 pytorch 官网下载  
# 到 https://download.pytorch.org/whl/cu118/torch，
# 手动下载 torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torch-2.3.0+cu118-cp311-cp311-linux_x86_64.whl
# 创建 constraint.txt
# 写入 torch==2.3.0+cu118
pip install torchaudio==2.3.0+cu118 --index-url https://download.pytorch.org/whl/cu118 -c constraint.txt
# 在 constraint.txt 中添加
# torchaudio==2.3.0+cu118
# 到 https://download.pytorch.org/whl/cu118/torchvision
# 手动下载 torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl
pip install torchvision-0.18.0+cu118-cp311-cp311-linux_x86_64.whl -c constraint.txt
# 在 constraint.txt 中添加
# torchvision==0.18.0+cu118
# 到 
# 手动下载 vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl
pip install vllm-0.5.1+cu118-cp311-cp311-manylinux1_x86_64.whl -c constraint.txt
```

**注意: torch, torchaudio, torchvision 和 vllm 的版本必须匹配 cuda 版本和 python 版本。具体匹配关系可以问ChatGPT**

```
pip install -r requirements.txt -c constraints.txt
```

## 启动服务 

到 glm4voice 的dev分支下载 vllm_model_server.py 

python vllm_model_server.py --host localhost --model-path <glm-4-voice-9b的path> --port 10002 --dtype bfloat16 --device cuda:0





