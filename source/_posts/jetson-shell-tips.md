---
title: Environment setting for Nvidia Jetson AGX
date: 2021-05-06
updated: {{date}}
tags: 
- Nvidia Jetson
---

## Install PyTorch v1.8 and torchvision v0.9.0



```bash
#!/bin/bash

cd ~

if [ ! -d "~/torch" ]; then
  curl -o torch.tar.gz 47.92.80.237/download/torch
  tar zxvf torch.tar.gz
fi

cd torch

# install torch v1.8.0 for python 3.6
echo $1 | sudo -S apt-get install -y python3-pip libopenblas-base libopenmpi-dev 
pip3 install Cython -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install numpy -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install torch-1.8.0-cp36-cp36m-linux_aarch64.whl

# install torchvision v0.9.0
echo $1 | sudo -S apt-get install -y libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev
cd torchvision
export BUILD_VERSION=0.9.0
python3 setup.py install --user
```

