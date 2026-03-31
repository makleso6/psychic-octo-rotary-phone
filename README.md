cat > setup/01_setup_env.sh <<'SCRIPT'
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
if python3 -c "import vapoursynth; print('VapourSynth:', vapoursynth.__version__)" 2>/dev/null; then
    log "✓ VapourSynth установлен"
else
    err "VapourSynth не установлен. Установи через apt:\n  sudo apt install vapoursynth python3-vapoursynth"
fi

# ── vsrepo ───────────────────────────────────────────────────────────────────
log "Установка vsrepo..."
pip install -q vsrepo

log "Обновление базы плагинов vsrepo..."
python3 -m vsrepo update 2>&1 | grep -v "^$" || log "  (база обновлена)"

# ── vs-mlrt ──────────────────────────────────────────────────────────────────
log "Установка vs-mlrt..."

# Python API
pip install -q vsmlrt || pip install -q git+https://github.com/AmusementClub/vs-mlrt.git || \
    log "  ⚠️  vsmlrt Python API не установлен"

# Бинарный плагин
if python3 -m vsrepo install mlrt 2>&1 | grep -q "Successfully"; then
    log "✓ vs-mlrt плагин установлен через vsrepo"
else
    log "  Установка vs-mlrt вручную..."
    
    MLRT_URL="https://github.com/AmusementClub/vs-mlrt/releases/latest/download/vs-mlrt-cuda-linux64.tar.gz"
    TMP=$(mktemp -d)
    
    if wget -q -O "$TMP/vsmlrt.tar.gz" "$MLRT_URL" && tar -xzf "$TMP/vsmlrt.tar.gz" -C "$TMP"; then
        mkdir -p ~/.local/lib/vapoursynth
        cp "$TMP"/*.so ~/.local/lib/vapoursynth/ 2>/dev/null || true
        log "✓ vs-mlrt установлен в ~/.local/lib/vapoursynth/"
    else
        log "  ⚠️  Не удалось установить vs-mlrt автоматически"
        log "     Скачай вручную: https://github.com/AmusementClub/vs-mlrt/releases"
    fi
    
    rm -rf "$TMP"
fi

# Проверка
if python3 -c "from vsmlrt import Backend; print('vsmlrt OK')" 2>/dev/null; then
    log "✓ vsmlrt Python API работает"
else
    log "⚠️  vsmlrt не импортируется (не критично для сборки engines)"
fi

# ── TensorRT ─────────────────────────────────────────────────────────────────
log "Проверка TensorRT..."
if python3 -c "import tensorrt; print('TensorRT:', tensorrt.__version__)" 2>/dev/null; then
    log "✓ TensorRT установлен"
else
    log "⚠️  TensorRT Python API не найден"
fi

if ! command -v trtexec &>/dev/null; then
    log "⚠️  trtexec не найден. Запусти: ./setup/install_tensorrt_tarball.sh"
else
    log "✓ trtexec: $(which trtexec)"
fi

# ── ffmpeg с vid.stab ────────────────────────────────────────────────────────
log "Проверка vid.stab..."
if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
    log "✓ vid.stab найден"
else
    log "⚠️  vid.stab не найден. Установка..."
    if ! grep -q "savoury1/ffmpeg" /etc/apt/sources.list.d/*.list 2>/dev/null; then
        sudo add-apt-repository -y ppa:savoury1/ffmpeg5
        sudo apt update
    fi
    sudo apt install -y ffmpeg
    
    if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
        log "✓ vid.stab установлен"
    else
        log "⚠️  vid.stab всё ещё отсутствует"
    fi
fi

# ── Утилиты ──────────────────────────────────────────────────────────────────
sudo apt install -y libimage-exiftool-perl mediainfo 2>/dev/null || true

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ Установка завершена"
log ""
python3 -c "import vapoursynth; print('  ✓ VapourSynth:', vapoursynth.__version__)"
python3 -c "import scenedetect; print('  ✓ SceneDetect:', scenedetect.__version__)"
python3 -c "import tensorrt; print('  ✓ TensorRT:', tensorrt.__version__)" 2>/dev/null || echo "  ⚠️  TensorRT Python API"
python3 -c "from vsmlrt import Backend; print('  ✓ vsmlrt')" 2>/dev/null || echo "  ⚠️  vsmlrt"
command -v trtexec &>/dev/null && echo "  ✓ trtexec" || echo "  ⚠️  trtexec (запусти install_tensorrt_tarball.sh)"
log ""
log "  Следующий шаг: ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
SCRIPT

chmod +x setup/01_setup_env.sh
