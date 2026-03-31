cat > setup/final_setup.sh <<'SCRIPT'
#!/usr/bin/env bash
# setup/final_setup.sh - Финальная настройка окружения
set -euo pipefail

log() { echo -e "\033[1;32m[setup]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

cd ~/legendary-potato

# Проверка venv
[[ -z "${VIRTUAL_ENV:-}" ]] && source venv/bin/activate

log "Установка недостающих Python-пакетов..."
pip install -q opencv-python

log "Установка системных утилит для сборки..."
sudo apt update
sudo apt install -y cmake ninja-build curl

log "Распаковка vs-mlrt из архива..."
if [[ -f vsmlrt-cuda.v15.16.7z.001 ]] && [[ -f vsmlrt-cuda.v15.16.7z.002 ]]; then
    # Объединяем многотомный архив
    cat vsmlrt-cuda.v15.16.7z.* > vsmlrt-cuda.v15.16.7z
    
    # Распаковываем
    7z x -y vsmlrt-cuda.v15.16.7z -o./vsmlrt-extracted
    
    # Копируем .so плагины в VapourSynth
    VS_PLUGIN_DIR="$HOME/.local/lib/vapoursynth"
    mkdir -p "$VS_PLUGIN_DIR"
    
    find ./vsmlrt-extracted -name "*.so" -exec cp -v {} "$VS_PLUGIN_DIR/" \;
    
    log "✓ vs-mlrt плагины установлены в $VS_PLUGIN_DIR"
    
    # Очистка
    rm -rf vsmlrt-extracted vsmlrt-cuda.v15.16.7z
else
    log "⚠️  Архивы vsmlrt не найдены (пропускаем)"
fi

log "Скачивание ONNX моделей..."
mkdir -p models/onnx

# RIFE v4.6
if [[ ! -f models/onnx/rife_v4.6.onnx ]]; then
    wget -q --show-progress -O models/onnx/rife_v4.6.onnx \
        https://github.com/hzwer/Practical-RIFE/releases/download/4.6/rife46.onnx
fi

# Real-ESRGAN x4plus
if [[ ! -f models/onnx/realesrgan_x4plus.onnx ]]; then
    wget -q --show-progress -O models/onnx/realesrgan_x4plus.onnx \
        https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.5.0/realesr-general-x4v3.onnx
fi

log "Проверка установки..."
python3 -c "import cv2; print('✓ OpenCV:', cv2.__version__)"

python3 <<'PYEOF'
import vapoursynth as vs
core = vs.core
plugins = [x for x in dir(core) if not x.startswith('_')]
print(f"✓ VapourSynth плагины: {len(plugins)}")
if hasattr(core, 'trt'):
    print("✓ vs-mlrt (TRT) доступен")
else:
    print("⚠️  vs-mlrt не загрузился")
PYEOF

log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ ОКРУЖЕНИЕ ГОТОВО"
log ""
log "  Следующий шаг: ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
SCRIPT

chmod +x setup/final_setup.sh
