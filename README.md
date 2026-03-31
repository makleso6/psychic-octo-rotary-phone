#!/usr/bin/env bash
# setup/install_tensorrt.sh - ИСПРАВЛЕННАЯ ВЕРСИЯ
set -euo pipefail

log() { echo -e "\033[1;32m[TensorRT]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Проверка CUDA ────────────────────────────────────────────────────────────
command -v nvidia-smi &>/dev/null || err "NVIDIA драйвер не найден"
CUDA_VERSION=$(nvidia-smi | grep -oP "CUDA Version: \K[\d.]+" | cut -d. -f1,2)
log "Обнаружена CUDA версия: ${CUDA_VERSION}"

# Определяем версию Ubuntu
if [[ -f /etc/os-release ]]; then
    source /etc/os-release
    UBUNTU_VERSION=$VERSION_ID
else
    err "Не удалось определить версию Ubuntu"
fi
log "Ubuntu версия: ${UBUNTU_VERSION}"

# Преобразуем 22.04 → 2204 для URL репозитория
UBUNTU_CODE="${UBUNTU_VERSION//./}"

# ── Добавление NVIDIA Machine Learning Repository ───────────────────────────
log "Добавление NVIDIA ML репозитория..."

# Удаляем старый keyring если есть
sudo rm -f /usr/share/keyrings/nvidia-ml.gpg 2>/dev/null || true

# Скачиваем и добавляем GPG ключ
wget -qO - https://developer.download.nvidia.com/compute/cuda/repos/ubuntu${UBUNTU_CODE}/x86_64/3bf863cc.pub | \
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-ml.gpg

# Добавляем репозиторий
echo "deb [signed-by=/usr/share/keyrings/nvidia-ml.gpg] https://developer.download.nvidia.com/compute/cuda/repos/ubuntu${UBUNTU_CODE}/x86_64/ /" | \
    sudo tee /etc/apt/sources.list.d/nvidia-ml.list

sudo apt update

# ── Установка TensorRT ──────────────────────────────────────────────────────
log "Установка TensorRT (полный набор)..."

# Для CUDA 12.x устанавливаем TensorRT 10.x
sudo apt install -y \
    libnvinfer10 \
    libnvinfer-plugin10 \
    libnvinfer-dev \
    libnvinfer-plugin-dev \
    libnvonnxparsers10 \
    libnvonnxparsers-dev \
    libnvinfer-bin \
    libnvinfer-samples \
    python3-libnvinfer \
    python3-libnvinfer-dev

# ── Проверка trtexec ─────────────────────────────────────────────────────────
log "Поиск trtexec..."

# trtexec может быть в разных местах
TRTEXEC_PATHS=(
    "/usr/bin/trtexec"
    "/usr/local/bin/trtexec"
    "/usr/src/tensorrt/bin/trtexec"
)

TRTEXEC_FOUND=""
for path in "${TRTEXEC_PATHS[@]}"; do
    if [[ -f "$path" ]]; then
        TRTEXEC_FOUND="$path"
        break
    fi
done

if [[ -n "$TRTEXEC_FOUND" ]]; then
    log "✅ trtexec найден: ${TRTEXEC_FOUND}"
    $TRTEXEC_FOUND --version || log "Версия trtexec:"
    $TRTEXEC_FOUND --help | head -5
else
    # Пробуем найти в системе
    TRTEXEC_FOUND=$(find /usr -name trtexec 2>/dev/null | head -1)
    
    if [[ -n "$TRTEXEC_FOUND" ]]; then
        log "✅ trtexec найден: ${TRTEXEC_FOUND}"
        sudo ln -sf "$TRTEXEC_FOUND" /usr/local/bin/trtexec
        log "Создана символическая ссылка: /usr/local/bin/trtexec"
    else
        err "trtexec не найден после установки пакетов!\n\
Попробуй вариант 2: ручная установка из tar-архива"
    fi
fi

# ── Python биндинги ──────────────────────────────────────────────────────────
if [[ -n "${VIRTUAL_ENV:-}" ]]; then
    log "Установка Python биндингов TensorRT в venv..."
    
    # Находим установленный wheel
    TRT_WHEEL=$(find /usr/lib -name "tensorrt-*-cp3*-linux_x86_64.whl" 2>/dev/null | head -1)
    
    if [[ -n "$TRT_WHEEL" ]]; then
        pip install "$TRT_WHEEL"
    else
        # Пробуем через pip
        pip install tensorrt --extra-index-url https://pypi.nvidia.com
    fi
    
    python3 -c "import tensorrt; print('TensorRT Python API:', tensorrt.__version__)" || \
        log "⚠️  Python API не установлен (не критично для trtexec)"
fi

log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ TensorRT установлен"
log "  trtexec: $(which trtexec 2>/dev/null || echo $TRTEXEC_FOUND)"
log ""
log "  Следующий шаг: ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
