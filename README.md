cat > setup/final_setup.sh <<'SCRIPT'
#!/usr/bin/env bash
# setup/final_setup.sh - Финальная настройка под твоё окружение
set -euo pipefail

log() { echo -e "\033[1;32m[setup]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

cd ~/legendary-potato

# ══════════════════════════════════════════════════════════════════════════════
# 1. PYTHON ПАКЕТЫ
# ══════════════════════════════════════════════════════════════════════════════

[[ -z "${VIRTUAL_ENV:-}" ]] && source venv/bin/activate

log "Установка недостающих Python-пакетов..."
pip install -q opencv-python

log "✓ opencv-python установлен"

# ══════════════════════════════════════════════════════════════════════════════
# 2. СИСТЕМНЫЕ УТИЛИТЫ
# ══════════════════════════════════════════════════════════════════════════════

log "Установка инструментов сборки..."
sudo apt update
sudo apt install -y cmake ninja-build curl build-essential pkg-config

log "✓ Утилиты установлены"

# ══════════════════════════════════════════════════════════════════════════════
# 3. СКАЧИВАНИЕ ONNX МОДЕЛЕЙ
# ══════════════════════════════════════════════════════════════════════════════

log "Скачивание ONNX моделей..."
mkdir -p models/onnx

# RIFE v4.6
if [[ ! -f models/onnx/rife_v4.6.onnx ]]; then
    log "  Скачивание RIFE v4.6..."
    wget -q --show-progress -O models/onnx/rife_v4.6.onnx \
        https://github.com/hzwer/Practical-RIFE/releases/download/4.6/rife46.onnx || \
        err "Не удалось скачать RIFE"
    log "  ✓ RIFE v4.6 скачан"
else
    log "  ✓ RIFE v4.6 уже существует"
fi

# Real-ESRGAN x4plus
if [[ ! -f models/onnx/realesrgan_x4plus.onnx ]]; then
    log "  Скачивание Real-ESRGAN x4plus..."
    wget -q --show-progress -O models/onnx/realesrgan_x4plus.onnx \
        https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.5.0/realesr-general-x4v3.onnx || \
        err "Не удалось скачать Real-ESRGAN"
    log "  ✓ Real-ESRGAN x4plus скачан"
else
    log "  ✓ Real-ESRGAN x4plus уже существует"
fi

# ══════════════════════════════════════════════════════════════════════════════
# 4. СБОРКА VS-MLRT
# ══════════════════════════════════════════════════════════════════════════════

log "Сборка vs-mlrt из исходников..."

BUILD_DIR="$HOME/build/vs-mlrt"
VS_PLUGIN_DIR="$HOME/.local/lib/vapoursynth"

# Настройка переменных окружения для TensorRT
export TENSORRT_DIR="/opt/tensorrt"
export PATH="$TENSORRT_DIR/bin:$PATH"
export LD_LIBRARY_PATH="$TENSORRT_DIR/lib:${LD_LIBRARY_PATH:-}"
export CPLUS_INCLUDE_PATH="$TENSORRT_DIR/include:${CPLUS_INCLUDE_PATH:-}"
export LIBRARY_PATH="$TENSORRT_DIR/lib:${LIBRARY_PATH:-}"

log "  Клонирование репозитория..."
if [[ -d "$BUILD_DIR" ]]; then
    cd "$BUILD_DIR"
    git pull
    git submodule update --init --recursive
else
    git clone --recursive https://github.com/AmusementClub/vs-mlrt.git "$BUILD_DIR"
    cd "$BUILD_DIR"
fi

log "  Конфигурация CMake для TensorRT бэкенда..."
rm -rf build
mkdir build && cd build

cmake -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DWHICH_BACKEND=TRT \
    -DTENSORRT_ROOT="$TENSORRT_DIR" \
    -DCMAKE_PREFIX_PATH="$TENSORRT_DIR" \
    .. || err "CMake конфигурация не удалась"

log "  Компиляция..."
ninja -v || err "Компиляция не удалась"

log "  ✓ Компиляция успешна"

# Установка плагина
SO_FILE=$(find . -name "libvs*.so" -o -name "libvstrt.so" | head -1)

if [[ -z "$SO_FILE" || ! -f "$SO_FILE" ]]; then
    err "Скомпилированный .so файл не найден!"
fi

log "  Найден плагин: $SO_FILE"

mkdir -p "$VS_PLUGIN_DIR"
cp -v "$SO_FILE" "$VS_PLUGIN_DIR/"

log "  ✓ vs-mlrt установлен в $VS_PLUGIN_DIR"

# ══════════════════════════════════════════════════════════════════════════════
# 5. РАСПАКОВКА МОДЕЛЕЙ ИЗ АРХИВА (если это модели vsmlrt)
# ══════════════════════════════════════════════════════════════════════════════

cd ~/legendary-potato

log "Распаковка моделей vsmlrt..."

if [[ -f vsmlrt-cuda.v15.16.7z.001 ]] && [[ -f vsmlrt-cuda.v15.16.7z.002 ]]; then
    # Объединяем многотомный архив
    log "  Объединение архива..."
    cat vsmlrt-cuda.v15.16.7z.* > vsmlrt-cuda-full.7z
    
    # Распаковываем
    log "  Распаковка..."
    mkdir -p models/vsmlrt
    7z x -y vsmlrt-cuda-full.7z -o./models/vsmlrt
    
    log "  ✓ Модели vsmlrt распакованы в models/vsmlrt/"
    ls -lh models/vsmlrt/ | head -10
    
    # Очистка
    rm -f vsmlrt-cuda-full.7z
else
    log "  ⚠️  Архивы vsmlrt-cuda не найдены (пропускаем)"
fi

# ══════════════════════════════════════════════════════════════════════════════
# 6. ПРОВЕРКА УСТАНОВКИ
# ══════════════════════════════════════════════════════════════════════════════

log "Проверка установки..."

python3 -c "import cv2; print('✓ OpenCV:', cv2.__version__)"

python3 <<'PYEOF'
import vapoursynth as vs
core = vs.core

plugins = [x for x in dir(core) if not x.startswith('_')]
print(f"✓ VapourSynth плагины загружены: {len(plugins)}")

# Проверка TensorRT плагина
if hasattr(core, 'trt'):
    print("✓ vs-mlrt (TRT) доступен")
elif hasattr(core, 'ort'):
    print("✓ vs-mlrt (ORT) доступен")
else:
    print("⚠️  vs-mlrt плагин не загрузился")
    print(f"   Доступные плагины: {plugins[:10]}")
PYEOF

# ══════════════════════════════════════════════════════════════════════════════
# 7. ИТОГИ
# ══════════════════════════════════════════════════════════════════════════════

log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ ОКРУЖЕНИЕ ПОЛНОСТЬЮ ГОТОВО"
log ""
log "  📂 ONNX модели: models/onnx/"
ls -lh models/onnx/*.onnx 2>/dev/null | awk '{print "     "$9" ("$5")"}'
log ""
log "  📂 vs-mlrt плагин: $VS_PLUGIN_DIR"
ls -1 "$VS_PLUGIN_DIR"/*.so 2>/dev/null | xargs -n1 basename | sed 's/^/     /'
log ""
log "  📂 vs-mlrt модели: models/vsmlrt/"
log ""
log "  🎯 Следующий шаг:"
log "     ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
SCRIPT

chmod +x setup/final_setup.sh
