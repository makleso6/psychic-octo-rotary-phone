#!/usr/bin/env bash
# Установка TensorRT 10.x для CUDA 12.x на Ubuntu 22.04/24.04

# 1. Добавь NVIDIA репозиторий
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update

# 2. Установи TensorRT (полный пакет)
sudo apt install -y \
    tensorrt \
    tensorrt-libs \
    tensorrt-dev \
    python3-libnvinfer \
    python3-libnvinfer-dev \
    libnvinfer-bin

# 3. Проверка trtexec
trtexec --version
which trtexec  # Должен быть в /usr/bin/trtexec

# 4. Установи Python-биндинги в venv
source ~/legendary-potato/venv/bin/activate
pip install tensorrt
