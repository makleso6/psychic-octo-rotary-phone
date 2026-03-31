cat > setup/02_build_trt_engines.sh <<'SCRIPT'
#!/usr/bin/env bash
set -euo pipefail

log() { echo -e "\033[1;34m[trt-build]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Параметры ────────────────────────────────────────────────────────────────
TILE_SIZE=${TILE_SIZE:-384}
OVERLAP=${OVERLAP:-24}
PRECISION=${PRECISION:-fp16}

MODEL_DIR="${MODEL_DIR:-$(pwd)/models}"
ENGINE_DIR="${MODEL_DIR}/engines"

log "Параметры: tile=${TILE_SIZE}, overlap=${OVERLAP}, precision=${PRECISION}"
log "Директория моделей: ${MODEL_DIR}"

# ── Проверка trtexec (ЕДИНСТВЕННАЯ ОБЯЗАТЕЛЬНАЯ ПРОВЕРКА) ────────────────────
if ! command -v trtexec &>/dev/null; then
    err "trtexec не найден. Установи TensorRT:\n  ./setup/install_tensorrt_tarball.sh"
fi

log "✓ trtexec: $(which trtexec)"
trtexec --version 2>&1 | head -2 | sed 's/^/  /'

# ── Создание директорий ──────────────────────────────────────────────────────
mkdir -p "${MODEL_DIR}"/{onnx,engines}

# ── Скачивание ONNX моделей ──────────────────────────────────────────────────
log "Проверка ONNX моделей..."

RIFE_ONNX="${MODEL_DIR}/onnx/rife_v4.6.onnx"
ESRGAN_ONNX="${MODEL_DIR}/onnx/realesrgan_x4plus.onnx"

if [[ ! -f "$RIFE_ONNX" ]]; then
    log "Скачивание RIFE v4.6..."
    wget -q --show-progress -O "$RIFE_ONNX" \
        "https://github.com/hzwer/Practical-RIFE/releases/download/4.6/rife46.onnx" || {
        log "⚠️  RIFE не скачался. Скачай вручную:"
        log "   https://github.com/hzwer/Practical-RIFE/releases/download/4.6/rife46.onnx"
        log "   Сохрани как: $RIFE_ONNX"
    }
fi

if [[ ! -f "$ESRGAN_ONNX" ]]; then
    log "Скачивание Real-ESRGAN..."
    wget -q --show-progress -O "$ESRGAN_ONNX" \
        "https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.5.0/realesr-general-x4v3.onnx" || {
        log "⚠️  Real-ESRGAN не скачался"
    }
fi

# ── Функция сборки engine ────────────────────────────────────────────────────
build_engine() {
    local name=$1
    local onnx=$2
    local engine=$3
    shift 3
    
    [[ ! -f "$onnx" ]] && { log "⏭️  ONNX не найден: $onnx"; return 1; }
    [[ -f "$engine" ]] && { log "⏭️  Engine существует: $(basename "$engine")"; return 0; }
    
    log "🔨 Сборка: ${name}"
    
    trtexec \
        --onnx="$onnx" \
        --saveEngine="$engine" \
        "$@" \
        $([ "$PRECISION" = "fp16" ] && echo "--fp16") \
        --verbose \
        --workspace=4096 \
        2>&1 | tee "${engine}.build.log"
    
    [[ -f "$engine" ]] && log "✅ Готово: $(basename "$engine") ($(du -h "$engine" | cut -f1))"
}

# ── RIFE ─────────────────────────────────────────────────────────────────────
build_engine "RIFE v4.6" "$RIFE_ONNX" \
    "${ENGINE_DIR}/rife_v4.6_${TILE_SIZE}_${PRECISION}.engine" \
    --minShapes=img0:1x3x256x256,img1:1x3x256x256,timestep:1x1x1x1 \
    --optShapes=img0:1x3x${TILE_SIZE}x${TILE_SIZE},img1:1x3x${TILE_SIZE}x${TILE_SIZE},timestep:1x1x1x1 \
    --maxShapes=img0:1x3x1080x1920,img1:1x3x1080x1920,timestep:1x1x1x1

# ── Real-ESRGAN ──────────────────────────────────────────────────────────────
build_engine "Real-ESRGAN x4" "$ESRGAN_ONNX" \
    "${ENGINE_DIR}/realesrgan_x4_${TILE_SIZE}_${PRECISION}.engine" \
    --minShapes=input:1x3x128x128 \
    --optShapes=input:1x3x${TILE_SIZE}x${TILE_SIZE} \
    --maxShapes=input:1x3x1080x1920

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ Сборка завершена"
log ""
log "  📁 Engines:"
ls -lh "${ENGINE_DIR}"/*.engine 2>/dev/null | awk '{print "     "$9" ("$5")"}' || echo "     (нет файлов)"
log ""
log "  Следующий шаг: ./enhance_concert.sh input.mp4"
log "═══════════════════════════════════════════════════════════════"
SCRIPT

chmod +x setup/02_build_trt_engines.sh
