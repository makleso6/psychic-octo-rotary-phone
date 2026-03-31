#!/usr/bin/env bash
set -euo pipefail

log() { echo -e "\033[1;32m[setup]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Проверка виртуального окружения ──────────────────────────────────────────
if [[ -z "${VIRTUAL_ENV:-}" ]]; then
    err "Запусти из venv:\n  source venv/bin/activate"
fi

log "Virtual env: $VIRTUAL_ENV ✓"

# ── Базовые проверки ─────────────────────────────────────────────────────────
command -v nvidia-smi &>/dev/null || err "NVIDIA driver не найден"
command -v python3    &>/dev/null || err "Python 3 не найден"
command -v ffmpeg     &>/dev/null || err "ffmpeg не найден"

CUDA_VER=$(nvidia-smi 2>/dev/null | grep -oP "CUDA Version: \K[\d.]+" || true)
log "CUDA: ${CUDA_VER:-неизвестна}"

# ── Python-пакеты ────────────────────────────────────────────────────────────
log "Установка Python-пакетов..."
pip install -q --upgrade pip wheel setuptools
pip install -q numpy tqdm rich opencv-python Cython 'scenedetect[opencv]'

# ── VapourSynth ──────────────────────────────────────────────────────────────
log "Проверка VapourSynth..."
if python3 -c "import vapoursynth as vs; print('VapourSynth:', vs.__version__)" 2>/dev/null; then
    log "✓ VapourSynth: $(python3 -c 'import vapoursynth; print(vapoursynth.__version__)')"
else
    err "VapourSynth не установлен. Запусти:\n  sudo apt install vapoursynth python3-vapoursynth libvapoursynth-dev"
fi

# Определяем директорию плагинов VS
VS_PLUGINS_DIR=$(python3 -c "
import vapoursynth as vs
import os
# Пробуем разные пути
paths = [
    os.path.expanduser('~/.local/lib/vapoursynth'),
    '/usr/lib/x86_64-linux-gnu/vapoursynth',
    '/usr/local/lib/vapoursynth'
]
for p in paths:
    if os.path.exists(p) or p == paths[0]:
        print(p)
        break
" 2>/dev/null || echo "$HOME/.local/lib/vapoursynth")

mkdir -p "$VS_PLUGINS_DIR"
log "Директория плагинов VS: $VS_PLUGINS_DIR"

# ── vsrepo ───────────────────────────────────────────────────────────────────
log "Установка vsrepo..."
pip install -q vsrepo

log "Обновление базы vsrepo..."
if python3 -m vsrepo update 2>&1 | grep -q "successfully"; then
    log "✓ База vsrepo обновлена"
    VSREPO_OK=1
else
    log "⚠️  vsrepo update не удался (будем устанавливать плагины вручную)"
    VSREPO_OK=0
fi

# ── vs-mlrt (TensorRT inference плагин) ──────────────────────────────────────
log "Установка vs-mlrt..."

# Пробуем через vsrepo
if [[ $VSREPO_OK -eq 1 ]] && python3 -m vsrepo install mlrt 2>&1 | grep -qE "(Successfully|already installed)"; then
    log "✓ vs-mlrt установлен через vsrepo"
else
    log "  Установка vs-mlrt вручную..."
    
    # Определяем архитектуру и версию CUDA
    ARCH="linux64"
    CUDA_MAJOR=$(echo "$CUDA_VER" | cut -d. -f1)
    
    # URL для скачивания (CUDA 12.x)
    if [[ "$CUDA_MAJOR" -ge 12 ]]; then
        MLRT_URL="https://github.com/AmusementClub/vs-mlrt/releases/latest/download/vs-mlrt-cuda-${ARCH}.tar.gz"
    else
        MLRT_URL="https://github.com/AmusementClub/vs-mlrt/releases/download/v13/vs-mlrt-cuda11-${ARCH}.tar.gz"
    fi
    
    TMP=$(mktemp -d)
    
    if wget -q --show-progress -O "$TMP/vsmlrt.tar.gz" "$MLRT_URL"; then
        tar -xzf "$TMP/vsmlrt.tar.gz" -C "$TMP" 2>/dev/null || {
            log "  Пробуем альтернативную распаковку..."
            cd "$TMP" && tar -xzf vsmlrt.tar.gz
        }
        
        # Копируем .so файлы
        find "$TMP" -name "*.so" -exec cp -v {} "$VS_PLUGINS_DIR/" \; 2>/dev/null || {
            log "  Нет .so файлов, копируем всё содержимое..."
            cp -r "$TMP"/* "$VS_PLUGINS_DIR/" 2>/dev/null || true
        }
        
        log "✓ vs-mlrt установлен в $VS_PLUGINS_DIR"
    else
        log "  ⚠️  Не удалось скачать vs-mlrt"
        log "     Скачай вручную: $MLRT_URL"
        log "     Распакуй .so файлы в: $VS_PLUGINS_DIR"
    fi
    
    rm -rf "$TMP"
fi

# Проверка vs-mlrt
if python3 -c "
import vapoursynth as vs
core = vs.core
try:
    # Пробуем загрузить функции TensorRT
    if hasattr(core, 'trt') or hasattr(core, 'ort'):
        print('vs-mlrt доступен')
        exit(0)
except:
    pass
exit(1)
" 2>/dev/null; then
    log "✓ vs-mlrt плагин загружается в VapourSynth"
else
    log "⚠️  vs-mlrt не загружается (не критично для сборки TensorRT engines)"
fi

# ── FFMS2 (источник видео) ───────────────────────────────────────────────────
log "Установка ffms2..."
if [[ $VSREPO_OK -eq 1 ]]; then
    python3 -m vsrepo install ffms2 2>&1 | grep -qE "(Successfully|already)" && \
        log "✓ ffms2 установлен" || log "  ffms2: установка через vsrepo не удалась"
else
    log "  ffms2: установи вручную или через apt (libffms2-dev)"
fi

# ── TensorRT ─────────────────────────────────────────────────────────────────
log "Проверка TensorRT..."

# Python API
if python3 -c "import tensorrt; print('TensorRT Python:', tensorrt.__version__)" 2>/dev/null; then
    log "✓ TensorRT Python API установлен"
else
    log "⚠️  TensorRT Python API не найден (не критично)"
fi

# trtexec (ОБЯЗАТЕЛЬНО для сборки engines)
if command -v trtexec &>/dev/null; then
    log "✓ trtexec: $(which trtexec)"
    trtexec --version 2>&1 | head -1 | sed 's/^/  /'
else
    log "❌ trtexec не найден!"
    log "   Запусти для установки: ./setup/install_tensorrt_tarball.sh"
fi

# ── ffmpeg с vid.stab ────────────────────────────────────────────────────────
log "Проверка vid.stab..."
if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
    log "✓ vid.stab доступен в ffmpeg"
else
    log "⚠️  vid.stab не найден, устанавливаем..."
    
    if ! grep -qr "savoury1/ffmpeg" /etc/apt/sources.list.d/ 2>/dev/null; then
        sudo add-apt-repository -y ppa:savoury1/ffmpeg5 || \
            sudo add-apt-repository -y ppa:savoury1/ffmpeg4
        sudo apt update
    fi
    
    sudo apt install -y ffmpeg
    
    if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
        log "✓ vid.stab установлен"
    else
        log "⚠️  vid.stab всё ещё недоступен (собери ffmpeg вручную)"
    fi
fi

# ── Дополнительные утилиты ───────────────────────────────────────────────────
log "Установка вспомогательных инструментов..."
sudo apt install -y libimage-exiftool-perl mediainfo 2>/dev/null || true

# ── Итоговая диагностика ─────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ Установка завершена"
log ""
log "  Установленные компоненты:"
python3 -c "import vapoursynth; print('  ✓ VapourSynth:', vapoursynth.__version__)"
python3 -c "import scenedetect; print('  ✓ PySceneDetect:', scenedetect.__version__)"
python3 -c "import cv2; print('  ✓ OpenCV:', cv2.__version__)"
python3 -c "import tensorrt; print('  ✓ TensorRT Python:', tensorrt.__version__)" 2>/dev/null || echo "  ⚠️  TensorRT Python API"

if command -v trtexec &>/dev/null; then
    echo "  ✓ trtexec: $(which trtexec)"
else
    echo "  ❌ trtexec НЕ УСТАНОВЛЕН (запусти install_tensorrt_tarball.sh)"
fi

if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
    echo "  ✓ ffmpeg vid.stab"
else
    echo "  ⚠️  ffmpeg vid.stab"
fi

log ""
log "  Плагины VapourSynth в: $VS_PLUGINS_DIR"
ls -1 "$VS_PLUGINS_DIR"/*.so 2>/dev/null | sed 's|.*/|  - |' || echo "  (нет .so файлов)"
log ""

if command -v trtexec &>/dev/null; then
    log "  🎯 Готов к сборке engines: ./setup/02_build_trt_engines.sh"
else
    log "  ⚠️  Сначала установи TensorRT: ./setup/install_tensorrt_tarball.sh"
fi

log "═══════════════════════════════════════════════════════════════"
