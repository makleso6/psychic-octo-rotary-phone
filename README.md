#!/usr/bin/env bash
# setup/install_tensorrt_tarball.sh
set -euo pipefail

log() { echo -e "\033[1;32m[TensorRT]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Параметры ────────────────────────────────────────────────────────────────
TRT_VERSION="10.7.0.23"
CUDA_VERSION="12.6"
PYTHON_VERSION="3.12"

TRT_BASE="TensorRT-${TRT_VERSION}"
TRT_ARCHIVE="${TRT_BASE}.Linux.x86_64-gnu.cuda-${CUDA_VERSION}.tar.gz"

INSTALL_DIR="/opt/tensorrt"
DOWNLOAD_DIR="$HOME/Downloads"

log "Установка TensorRT ${TRT_VERSION} для CUDA ${CUDA_VERSION}, Python ${PYTHON_VERSION}"

# ── Проверка CUDA ────────────────────────────────────────────────────────────
CUDA_VER=$(nvidia-smi 2>/dev/null | grep -oP "CUDA Version: \K[\d.]+" || echo "unknown")
log "Обнаруженная CUDA: ${CUDA_VER}"

# ── Скачивание ───────────────────────────────────────────────────────────────
mkdir -p "$DOWNLOAD_DIR"
cd "$DOWNLOAD_DIR"

if [[ -f "$TRT_ARCHIVE" ]]; then
    log "✓ Архив уже скачан: $TRT_ARCHIVE"
else
    log "Требуется скачать TensorRT вручную:"
    log ""
    log "1. Открой: https://developer.nvidia.com/tensorrt-download"
    log "2. Выбери: TensorRT 10.7 → TAR Package for CUDA 12.0-12.6"
    log "3. Сохрани в: $DOWNLOAD_DIR"
    log ""
    read -p "Нажми Enter после скачивания..."
    
    [[ -f "$TRT_ARCHIVE" ]] || err "Файл $TRT_ARCHIVE не найден в $DOWNLOAD_DIR"
fi

# ── Распаковка ───────────────────────────────────────────────────────────────
log "Распаковка TensorRT..."
tar -xzf "$TRT_ARCHIVE" || err "Не удалось распаковать архив"

if [[ -d "$INSTALL_DIR" ]]; then
    log "Удаление старой версии..."
    sudo rm -rf "$INSTALL_DIR"
fi

sudo mv "$TRT_BASE" "$INSTALL_DIR"
sudo chmod -R 755 "$INSTALL_DIR"

log "✓ TensorRT установлен в: $INSTALL_DIR"

# ── Настройка переменных окружения ───────────────────────────────────────────
log "Настройка переменных окружения..."

# Удаляем старые записи TensorRT из .bashrc
sed -i '/# TensorRT/,/^$/d' ~/.bashrc 2>/dev/null || true

# ✅ ПРАВИЛЬНО: используем 'EOF' в кавычках для отключения раскрытия переменных
cat >> ~/.bashrc <<'HEREDOC'

# TensorRT
export TENSORRT_DIR=/opt/tensorrt
export PATH=$TENSORRT_DIR/bin:$PATH
export LD_LIBRARY_PATH=$TENSORRT_DIR/lib:${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}
HEREDOC

# Альтернативный синтаксис (если нужно раскрыть INSTALL_DIR, но не LD_LIBRARY_PATH):
# cat >> ~/.bashrc <<EOF
# 
# # TensorRT
# export TENSORRT_DIR=${INSTALL_DIR}
# export PATH=\$TENSORRT_DIR/bin:\$PATH
# export LD_LIBRARY_PATH=\$TENSORRT_DIR/lib:\${LD_LIBRARY_PATH:+:\$LD_LIBRARY_PATH}
# EOF

# Применяем переменные для текущей сессии
export TENSORRT_DIR="$INSTALL_DIR"
export PATH="$TENSORRT_DIR/bin:$PATH"
export LD_LIBRARY_PATH="$TENSORRT_DIR/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"

log "✓ Переменные окружения добавлены в ~/.bashrc"

# ── Проверка trtexec ─────────────────────────────────────────────────────────
log "Проверка trtexec..."
if [[ -f "$INSTALL_DIR/bin/trtexec" ]]; then
    log "✓ trtexec найден: $INSTALL_DIR/bin/trtexec"
    
    # Запускаем с явным указанием LD_LIBRARY_PATH
    LD_LIBRARY_PATH="$INSTALL_DIR/lib:${LD_LIBRARY_PATH:-}" \
        "$INSTALL_DIR/bin/trtexec" --version 2>&1 | head -3
else
    err "trtexec не найден в $INSTALL_DIR/bin/"
fi

# Создаём символическую ссылку
if sudo ln -sf "$INSTALL_DIR/bin/trtexec" /usr/local/bin/trtexec 2>/dev/null; then
    log "✓ Символическая ссылка создана: /usr/local/bin/trtexec"
fi

# ── Установка Python wheel ───────────────────────────────────────────────────
if [[ -n "${VIRTUAL_ENV:-}" ]]; then
    log "Установка Python биндингов TensorRT..."
    
    PYTHON_VER_SHORT=$(python3 -c "import sys; print(f'{sys.version_info.major}{sys.version_info.minor}')")
    TRT_WHEEL=$(find "$INSTALL_DIR/python" -name "tensorrt-*-cp${PYTHON_VER_SHORT}-*.whl" 2>/dev/null | head -1)
    
    if [[ -n "$TRT_WHEEL" && -f "$TRT_WHEEL" ]]; then
        log "✓ Найден wheel: $(basename "$TRT_WHEEL")"
        pip install "$TRT_WHEEL"
        
        # Проверка импорта
        if python3 -c "import tensorrt; print('TensorRT Python API:', tensorrt.__version__)" 2>/dev/null; then
            log "✓ Python API установлен и работает"
        else
            log "⚠️  Python API установлен, но импорт не работает (не критично)"
        fi
    else
        log "⚠️  Wheel для Python ${PYTHON_VER_SHORT} не найден"
        log "   Доступные wheels:"
        ls -1 "$INSTALL_DIR/python/"*.whl 2>/dev/null | xargs -n1 basename || echo "   (нет файлов)"
        log ""
        log "   trtexec будет работать без Python API"
    fi
else
    log "⚠️  Виртуальное окружение не активировано"
fi

# ── Создание wrapper-скрипта ─────────────────────────────────────────────────
log "Создание wrapper-скрипта для trtexec..."

cat > /tmp/trtexec-wrapper <<'WRAPPER'
#!/usr/bin/env bash
# Wrapper для trtexec с правильным LD_LIBRARY_PATH
export TENSORRT_DIR=/opt/tensorrt
export LD_LIBRARY_PATH=$TENSORRT_DIR/lib:${LD_LIBRARY_PATH:-}
exec $TENSORRT_DIR/bin/trtexec "$@"
WRAPPER

sudo mv /tmp/trtexec-wrapper /usr/local/bin/trtexec
sudo chmod +x /usr/local/bin/trtexec

log "✓ Wrapper создан: /usr/local/bin/trtexec"

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ TensorRT ${TRT_VERSION} установлен"
log ""
log "  📂 Директория: $INSTALL_DIR"
log "  🔧 trtexec: /usr/local/bin/trtexec (wrapper)"
log ""
log "  Проверка установки:"
echo "  "
trtexec --version 2>&1 | head -2 | sed 's/^/    /'
log ""
log "  Переменные окружения добавлены в ~/.bashrc"
log "  Для применения выполни: source ~/.bashrc"
log ""
log "  Следующий шаг:"
log "    ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
