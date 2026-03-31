#!/usr/bin/env bash
# Скачивание TensorRT вручную

# 1. Скачай TensorRT с https://developer.nvidia.com/tensorrt
# Для CUDA 12.6 + Ubuntu 22.04: TensorRT-10.7.0.23.Linux.x86_64-gnu.cuda-12.6.tar.gz

cd ~/Downloads
# (Скачай архив через браузер или wget с NVIDIA Developer Account)

# 2. Распакуй
tar -xzvf TensorRT-10.*.tar.gz
sudo mv TensorRT-10.* /opt/tensorrt

# 3. Добавь в PATH
echo 'export PATH=/opt/tensorrt/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/opt/tensorrt/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc

# 4. Установи Python wheel
source ~/legendary-potato/venv/bin/activate
pip install /opt/tensorrt/python/tensorrt-*-cp312-*.whl

# 5. Проверка
trtexec --version
python3 -c "import tensorrt; print(tensorrt.__version__)"
