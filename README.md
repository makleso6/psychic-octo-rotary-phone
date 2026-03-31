#!/usr/bin/env bash
# setup/install_tensorrt_tarball.sh
set -euo pipefail

log() { echo -e "\033[1;32m[TensorRT]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Параметры ────────────────────────────────────────────────────────────────
TRT_VERSION="10.7.0.23"
CUDA_VERSION="12.6"  # Проверь свою версию через: nvidia-smi
PYTHON_VERSION="3.12"

TRT_BASE="TensorRT-${TRT_VERSION}"
TRT_ARCHIVE="${TRT_BASE}.Linux.x86_64-gnu.cuda-${CUDA_VERSION}.tar.gz"
TRT_URL="https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.7.0/tars/${TRT_ARCHIVE}"

INSTALL_DIR="/opt/tensorrt"
DOWNLOAD_DIR="$HOME/Downloads"

log "Установка TensorRT ${TRT_VERSION} для CUDA ${CUDA_VERSION}, Python ${PYTHON_VERSION}"

# ── Проверка CUDA ────────────────────────────────────────────────────────────
CUDA_VER=$(nvidia-smi 2>/dev/null | grep -oP "CUDA Version: \K[\d.]+" || echo "unknown")
log "Обнаруженная CUDA: ${CUDA_VER}"

if [[ "$CUDA_VER" != "$CUDA_VERSION"* ]]; then
    log "⚠️  CUDA ${CUDA_VER} != ${CUDA_VERSION}, возможны проблемы"
    log "   Продолжаем с версией для CUDA ${CUDA_VERSION}..."
fi

# ── Скачивание (требуется NVIDIA Developer Account) ─────────────────────────
mkdir -p "$DOWNLOAD_DIR"
cd "$DOWNLOAD_DIR"

if [[ -f "$TRT_ARCHIVE" ]]; then
    log "✓ Архив уже скачан: $TRT_ARCHIVE"
else
    log "Требуется скачать TensorRT вручную:"
    log ""
    log "1. Открой: https://developer.nvidia.com/tensorrt-download"
    log "2. Выбери: TensorRT 10.7 → TensorRT 10.7.0 for Linux x86_64 and CUDA 12.0 to 12.6 TAR Package"
    log "3. Сохрани в: $DOWNLOAD_DIR"
    log ""
    log "Альтернативные версии:"
    log "  - TensorRT 10.6.0 (более стабильная)"
    log "  - TensorRT 8.6.x (для CUDA 11.x)"
    log ""
    read -p "Нажми Enter после скачивания..."
    
    [[ -f "$TRT_ARCHIVE" ]] || err "Файл $TRT_ARCHIVE не найден в $DOWNLOAD_DIR"
fi

# ── Распаковка ───────────────────────────────────────────────────────────────
log "Распаковка TensorRT..."
tar -xzf "$TRT_ARCHIVE" || err "Не удалось распаковать архив"

# Удаляем старую версию если есть
if [[ -d "$INSTALL_DIR" ]]; then
    log "Удаление старой версии..."
    sudo rm -rf "$INSTALL_DIR"
fi

# Перемещаем в /opt
sudo mv "$TRT_BASE" "$INSTALL_DIR"
sudo chmod -R 755 "$INSTALL_DIR"

log "✓ TensorRT установлен в: $INSTALL_DIR"

# ── Настройка переменных окружения ───────────────────────────────────────────
log "Настройка переменных окружения..."

# Удаляем старые записи из .bashrc
sed -i '/# TensorRT/,+3d' ~/.bashrc 2>/dev/null || true

cat >> ~/.bashrc <<EOF

# TensorRT
export TENSORRT_DIR=${INSTALL_DIR}
export PATH=\$TENSORRT_DIR/bin:\$PATH
export LD_LIBRARY_PATH=\$TENSORRT_DIR/lib:\$LD_LIBRARY_PATH
EOF

# Применяем сразу
export TENSORRT_DIR="$INSTALL_DIR"
export PATH="$TENSORRT_DIR/bin:$PATH"
export LD_LIBRARY_PATH="$TENSORRT_DIR/lib:$LD_LIBRARY_PATH"

# ── Проверка trtexec ─────────────────────────────────────────────────────────
log "Проверка trtexec..."
if [[ -f "$INSTALL_DIR/bin/trtexec" ]]; then
    log "✓ trtexec найден: $INSTALL_DIR/bin/trtexec"
    "$INSTALL_DIR/bin/trtexec" --version | head -3
else
    err "trtexec не найден в $INSTALL_DIR/bin/"
fi

# Создаём символическую ссылку для удобства
sudo ln -sf "$INSTALL_DIR/bin/trtexec" /usr/local/bin/trtexec 2>/dev/null || true

# ── Установка Python wheel ───────────────────────────────────────────────────
if [[ -n "${VIRTUAL_ENV:-}" ]]; then
    log "Установка Python биндингов TensorRT..."
    
    # Ищем wheel для текущей версии Python
    PYTHON_VER_SHORT=$(python3 -c "import sys; print(f'{sys.version_info.major}{sys.version_info.minor}')")
    TRT_WHEEL=$(find "$INSTALL_DIR/python" -name "tensorrt-*-cp${PYTHON_VER_SHORT}-*.whl" 2>/dev/null | head -1)
    
    if [[ -n "$TRT_WHEEL" && -f "$TRT_WHEEL" ]]; then
        log "✓ Найден wheel: $(basename $TRT_WHEEL)"
        pip install "$TRT_WHEEL"
        
        python3 -c "import tensorrt; print('TensorRT Python API:', tensorrt.__version__)" || \
            log "⚠️  Python API установлен, но импорт не работает"
    else
        log "⚠️  Wheel для Python ${PYTHON_VER_SHORT} не найден в $INSTALL_DIR/python/"
        log "   Доступные wheels:"
        ls -1 "$INSTALL_DIR/python/" 2>/dev/null || echo "   (директория пуста)"
        log ""
        log "   trtexec будет работать без Python API"
    fi
else
    log "⚠️  Виртуальное окружение не активировано"
    log "   Для установки Python API выполни:"
    log "   source ~/legendary-potato/venv/bin/activate"
    log "   pip install $INSTALL_DIR/python/tensorrt-*-cp${PYTHON_VER_SHORT}-*.whl"
fi

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ TensorRT ${TRT_VERSION} установлен"
log ""
log "  📂 Директория: $INSTALL_DIR"
log "  🔧 trtexec: $(which trtexec 2>/dev/null || echo "$INSTALL_DIR/bin/trtexec")"
log ""
log "  Для применения переменных окружения:"
log "    source ~/.bashrc"
log "  или просто закрой и открой терминал заново"
log ""
log "  Следующий шаг:"
log "    source ~/.bashrc"
log "    ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
