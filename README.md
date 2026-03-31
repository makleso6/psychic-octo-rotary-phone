#!/usr/bin/env bash
# setup/02_build_trt_engines.sh
# Сборка TensorRT-движков для RIFE и Real-ESRGAN
set -euo pipefail

log() { echo -e "\033[1;34m[trt-build]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Параметры ────────────────────────────────────────────────────────────────
TILE_SIZE=${TILE_SIZE:-384}
OVERLAP=${OVERLAP:-24}
PRECISION=${PRECISION:-fp16}  # fp16 или fp32

MODEL_DIR="${MODEL_DIR:-$(pwd)/models}"
ENGINE_DIR="${MODEL_DIR}/engines"

log "Параметры: tile=${TILE_SIZE}, overlap=${OVERLAP}, precision=${PRECISION}"
log "Директория моделей: ${MODEL_DIR}"

# ── Проверка trtexec ─────────────────────────────────────────────────────────
if ! command -v trtexec &>/dev/null; then
    err "trtexec не найден. Установи TensorRT:\n\
  Ubuntu: sudo apt install tensorrt libnvinfer-bin\n\
  Или скачай с https://developer.nvidia.com/tensorrt\n\
  \n\
  Для установки запусти:\n\
    ./setup/install_tensorrt.sh"
fi

TRTEXEC=$(command -v trtexec)
log "✓ trtexec: ${TRTEXEC}"

TRT_VERSION=$(trtexec --version 2>&1 | grep -oP "TensorRT \K[\d.]+" || echo "unknown")
log "✓ TensorRT версия: ${TRT_VERSION}"

# ── Проверка CUDA ────────────────────────────────────────────────────────────
command -v nvidia-smi &>/dev/null || err "NVIDIA драйвер не найден"
CUDA_VER=$(nvidia-smi 2>/dev/null | grep -oP "CUDA Version: \K[\d.]+" || echo "unknown")
log "✓ CUDA версия: ${CUDA_VER}"

# ── Создание директорий ──────────────────────────────────────────────────────
mkdir -p "${MODEL_DIR}"/{onnx,engines}

# ── Скачивание ONNX-моделей ──────────────────────────────────────────────────
log "Проверка ONNX-моделей..."

RIFE_ONNX="${MODEL_DIR}/onnx/rife_v4.6.onnx"
ESRGAN_ONNX="${MODEL_DIR}/onnx/realesrgan_x4plus.onnx"

if [[ ! -f "$RIFE_ONNX" ]]; then
    log "Скачивание RIFE v4.6..."
    wget -O "$RIFE_ONNX" \
        "https://github.com/hzwer/Practical-RIFE/releases/download/4.6/rife46.onnx" || \
        err "Не удалось скачать RIFE. Скачай вручную в ${MODEL_DIR}/onnx/"
fi

if [[ ! -f "$ESRGAN_ONNX" ]]; then
    log "Скачивание Real-ESRGAN x4plus..."
    wget -O "$ESRGAN_ONNX" \
        "https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.5.0/realesr-general-x4v3.onnx" || \
        log "⚠️  Real-ESRGAN: скачай вручную в ${MODEL_DIR}/onnx/"
fi

# ── Конвертация ONNX → TensorRT ──────────────────────────────────────────────
build_engine() {
    local model_name=$1
    local onnx_path=$2
    local engine_path=$3
    local min_shapes=$4
    local opt_shapes=$5
    local max_shapes=$6
    
    if [[ -f "$engine_path" ]]; then
        log "⏭️  Engine уже существует: $(basename "$engine_path")"
        return 0
    fi
    
    log "🔨 Сборка TensorRT engine: ${model_name}..."
    log "   ONNX: ${onnx_path}"
    log "   Engine: ${engine_path}"
    
    local fp_flag=""
    if [[ "$PRECISION" == "fp16" ]]; then
        fp_flag="--fp16"
    fi
    
    trtexec \
        --onnx="$onnx_path" \
        --saveEngine="$engine_path" \
        --minShapes="$min_shapes" \
        --optShapes="$opt_shapes" \
        --maxShapes="$max_shapes" \
        $fp_flag \
        --verbose \
        --workspace=4096 \
        2>&1 | tee "${engine_path}.build.log"
    
    if [[ -f "$engine_path" ]]; then
        log "✅ Engine готов: $(basename "$engine_path")"
        log "   Размер: $(du -h "$engine_path" | cut -f1)"
    else
        err "Не удалось создать engine для ${model_name}"
    fi
}

# ── RIFE (интерполяция кадров) ───────────────────────────────────────────────
if [[ -f "$RIFE_ONNX" ]]; then
    RIFE_ENGINE="${ENGINE_DIR}/rife_v4.6_${TILE_SIZE}_${PRECISION}.engine"
    
    # Формат входов RIFE: img0, img1, timestep
    # Динамические размеры: [batch, channels, height, width]
    build_engine \
        "RIFE v4.6" \
        "$RIFE_ONNX" \
        "$RIFE_ENGINE" \
        "img0:1x3x256x256,img1:1x3x256x256,timestep:1x1x1x1" \
        "img0:1x3x${TILE_SIZE}x${TILE_SIZE},img1:1x3x${TILE_SIZE}x${TILE_SIZE},timestep:1x1x1x1" \
        "img0:1x3x1080x1920,img1:1x3x1080x1920,timestep:1x1x1x1"
else
    log "⚠️  RIFE ONNX не найден, пропускаем"
fi

# ── Real-ESRGAN (апскейлинг) ─────────────────────────────────────────────────
if [[ -f "$ESRGAN_ONNX" ]]; then
    ESRGAN_ENGINE="${ENGINE_DIR}/realesrgan_x4_${TILE_SIZE}_${PRECISION}.engine"
    
    # Формат: [batch, channels, height, width]
    build_engine \
        "Real-ESRGAN x4" \
        "$ESRGAN_ONNX" \
        "$ESRGAN_ENGINE" \
        "input:1x3x128x128" \
        "input:1x3x${TILE_SIZE}x${TILE_SIZE}" \
        "input:1x3x1080x1920"
else
    log "⚠️  Real-ESRGAN ONNX не найден, пропускаем"
fi

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ Сборка TensorRT engines завершена"
log ""
log "  📁 Engines сохранены в: ${ENGINE_DIR}"
ls -lh "${ENGINE_DIR}"/*.engine 2>/dev/null | awk '{print "     "$9" ("$5")"}'
log ""
log "  Следующий шаг: ./enhance_concert.sh input.mp4"
log "═══════════════════════════════════════════════════════════════"
