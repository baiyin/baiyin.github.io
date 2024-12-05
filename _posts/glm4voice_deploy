

基础环境 

```
OS: Ubuntu 20.04.5 LTS (Focal Fossa)
GPU: A100-SXM-80G
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

## 创建虚拟环境 
pyenv virtualenv 3.11.10 glm4voice_env 
pyenv activate glm4voice_env  







