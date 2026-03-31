#!/usr/bin/env bash
# setup/01_setup_env.sh - ИСПРАВЛЕННАЯ ВЕРСИЯ
set -euo pipefail

log() { echo -e "\033[1;32m[setup]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Проверка виртуального окружения ──────────────────────────────────────────
if [[ -z "${VIRTUAL_ENV:-}" ]]; then
    err "Запусти из venv:\n  source venv/bin/activate"
fi

log "Virtual env: $VIRTUAL_ENV ✓"

# ── Проверки базовых зависимостей ────────────────────────────────────────────
command -v nvidia-smi &>/dev/null || err "NVIDIA driver не найден"
command -v python3    &>/dev/null || err "Python 3 не найден"
command -v ffmpeg     &>/dev/null || err "ffmpeg не найден: sudo apt install ffmpeg"

CUDA_VER=$(nvidia-smi 2>/dev/null | grep -oP "CUDA Version: \K[\d.]+" || true)
log "CUDA: ${CUDA_VER:-"неизвестна"}"

# ── Python-пакеты ────────────────────────────────────────────────────────────
log "Установка Python-пакетов..."
pip install --upgrade pip wheel setuptools
pip install \
    numpy \
    tqdm \
    rich \
    opencv-python \
    Cython \
    'scenedetect[opencv]'

# ── VapourSynth ──────────────────────────────────────────────────────────────
log "Проверка VapourSynth..."
if ! python3 -c "import vapoursynth" 2>/dev/null; then
    log "Установка VapourSynth через apt..."
    if ! grep -q "djcj/vapoursynth" /etc/apt/sources.list.d/*.list 2>/dev/null; then
        sudo add-apt-repository -y ppa:djcj/vapoursynth
        sudo apt update
    fi
    sudo apt install -y vapoursynth python3-vapoursynth libvapoursynth-dev
fi

python3 -c "import vapoursynth; print('VapourSynth:', vapoursynth.__version__)" || \
    err "VapourSynth не установлен"

VS_PLUGIN_DIR=$(python3 -c "import vapoursynth as vs; print(vs.core.plugins())" 2>/dev/null | grep -oP "(?<=').*(?=')" | head -1 || echo "$HOME/.local/lib/vapoursynth")
log "VS plugin dir: $VS_PLUGIN_DIR"

# ── vsrepo (обновление базы плагинов) ────────────────────────────────────────
log "Установка vsrepo..."
pip install vsrepo

log "Обновление базы плагинов vsrepo..."
python3 -m vsrepo update || {
    log "  vsrepo update не удался, пробуем инициализацию вручную..."
    mkdir -p ~/.config/vsrepo
    python3 -m vsrepo upgrade-all 2>/dev/null || log "  (пропускаем)"
}

# Проверяем успешность обновления
if python3 -m vsrepo available 2>&1 | grep -q "ffms2"; then
    log "✓ База плагинов vsrepo обновлена"
    VSREPO_OK=1
else
    log "⚠️  vsrepo не работает корректно, будем устанавливать плагины вручную"
    VSREPO_OK=0
fi

# ── Установка плагинов VapourSynth ───────────────────────────────────────────
if [[ $VSREPO_OK -eq 1 ]]; then
    log "Установка плагинов через vsrepo..."
    python3 -m vsrepo install ffms2 || log "  ffms2: пропускаем"
    python3 -m vsrepo install bm3dcuda || log "  bm3dcuda: не найден в репозитории"
    python3 -m vsrepo install rife || log "  rife: не найден в репозитории"
else
    log "Установка плагинов вручную (vsrepo недоступен)..."
fi

# ── vs-mlrt (TensorRT inference для VapourSynth) ─────────────────────────────
log "Установка vs-mlrt..."

# Устанавливаем Python-обёртку
pip install vsmlrt || {
    log "  Установка vsmlrt через pip не удалась, пробуем из исходников..."
    pip install git+https://github.com/AmusementClub/vs-mlrt.git || \
        log "  ⚠️  vsmlrt Python package не установлен"
}

# Устанавливаем бинарный плагин
if [[ $VSREPO_OK -eq 1 ]]; then
    python3 -m vsrepo install mlrt || {
        log "  vs-mlrt плагин не найден через vsrepo, устанавливаем вручную..."
        install_vsmlrt_manual
    }
else
    install_vsmlrt_manual
fi

# Функция ручной установки vs-mlrt
install_vsmlrt_manual() {
    local MLRT_RELEASE="https://github.com/AmusementClub/vs-mlrt/releases/latest/download/vs-mlrt-cuda-linux64.tar.gz"
    local TMP_DIR=$(mktemp -d)
    
    log "  Скачивание vs-mlrt..."
    wget -q -O "$TMP_DIR/vsmlrt.tar.gz" "$MLRT_RELEASE" || {
        log "  ⚠️  Не удалось скачать vs-mlrt"
        return 1
    }
    
    tar -xzf "$TMP_DIR/vsmlrt.tar.gz" -C "$TMP_DIR"
    
    # Копируем плагины
    mkdir -p "$VS_PLUGIN_DIR"
    cp -v "$TMP_DIR"/*.so "$VS_PLUGIN_DIR/" 2>/dev/null || {
        log "  Копирование в $VS_PLUGIN_DIR не удалось, пробуем ~/.local/lib/vapoursynth/"
        mkdir -p ~/.local/lib/vapoursynth
        cp -v "$TMP_DIR"/*.so ~/.local/lib/vapoursynth/
    }
    
    rm -rf "$TMP_DIR"
    log "  ✓ vs-mlrt установлен вручную"
}

# Проверка установки vs-mlrt
python3 -c "from vsmlrt import Backend; print('vsmlrt:', Backend)" 2>/dev/null && \
    log "✓ vsmlrt Python API работает" || \
    log "⚠️  vsmlrt Python API не импортируется (проверь установку)"

# ── TensorRT ─────────────────────────────────────────────────────────────────
log "Проверка TensorRT..."
if python3 -c "import tensorrt; print('TensorRT:', tensorrt.__version__)" 2>/dev/null; then
    log "✓ TensorRT Python API установлен"
else
    log "Установка TensorRT..."
    pip install tensorrt --extra-index-url https://pypi.nvidia.com || \
        log "⚠️  TensorRT: установи вручную через ./setup/install_tensorrt_tarball.sh"
fi

# ── ffmpeg с vid.stab ────────────────────────────────────────────────────────
log "Проверка vid.stab..."
if ! ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
    log "Установка ffmpeg с vid.stab..."
    
    if ! grep -q "savoury1/ffmpeg" /etc/apt/sources.list.d/*.list 2>/dev/null; then
        sudo add-apt-repository -y ppa:savoury1/ffmpeg4 || \
            sudo add-apt-repository -y ppa:savoury1/ffmpeg5
        sudo apt update
    fi
    
    sudo apt install -y ffmpeg
    
    # Проверка после установки
    if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
        log "✓ vid.stab установлен"
    else
        log "⚠️  vid.stab всё ещё отсутствует (собери ffmpeg вручную)"
    fi
else
    log "✓ vid.stab найден в ffmpeg"
fi

# ── Дополнительные утилиты ───────────────────────────────────────────────────
log "Установка вспомогательных инструментов..."
sudo apt install -y libimage-exiftool-perl mediainfo 2>/dev/null || true

# ── Gyroflow (опционально) ───────────────────────────────────────────────────
if ! command -v gyroflow &>/dev/null; then
    log "Gyroflow не найден (опционально)"
    log "  Скачай с https://gyroflow.xyz/download для продвинутой стабилизации"
fi

# ── Финальная проверка ───────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ Установка завершена"
log ""
log "  Проверка компонентов:"
python3 -c "import vapoursynth as vs; print('  ✓ VapourSynth:', vs.__version__)"
python3 -c "import scenedetect; print('  ✓ SceneDetect:', scenedetect.__version__)"
python3 -c "import tensorrt; print('  ✓ TensorRT:', tensorrt.__version__)" 2>/dev/null || echo "  ⚠️  TensorRT Python API не установлен"
python3 -c "from vsmlrt import Backend; print('  ✓ vsmlrt')" 2>/dev/null || echo "  ⚠️  vsmlrt не установлен"
command -v trtexec &>/dev/null && echo "  ✓ trtexec: $(which trtexec)" || echo "  ⚠️  trtexec не найден"
log ""
log "  Следующий шаг: ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
