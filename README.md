#!/usr/bin/env bash
# Установка VapourSynth через apt (Ubuntu 22.04+)

# 1. Добавь репозиторий с актуальной версией
sudo add-apt-repository -y ppa:djcj/vapoursynth
sudo apt update

# 2. Установи системный VapourSynth + Python-биндинги
sudo apt install -y \
    vapoursynth \
    python3-vapoursynth \
    libvapoursynth-dev

# 3. Проверка
python3 -c "import vapoursynth; print('VapourSynth:', vapoursynth.__version__)"
