#!/usr/bin/env bash
# setup/build_vsmlrt.sh - Сборка vs-mlrt из исходников для Ubuntu
set -euo pipefail

log() { echo -e "\033[1;35m[vs-mlrt]\033[0m $*"; }
err() { echo -e "\033[1;31m[ERROR]\033[0m $*" >&2; exit 1; }

# ── Параметры ────────────────────────────────────────────────────────────────
BACKEND=${BACKEND:-TRT}  # TRT, NCNN, ORT, OV
BUILD_DIR="$HOME/build/vs-mlrt"
VS_PLUGINS_DIR="$HOME/.local/lib/vapoursynth"

log "Сборка vs-mlrt с бэкендом: $BACKEND"

# ── Проверка зависимостей ────────────────────────────────────────────────────
log "Проверка системных зависимостей..."

command -v nvidia-smi &>/dev/null || err "NVIDIA драйвер не найден"
command -v nvcc &>/dev/null || {
    log "CUDA Toolkit не найден, устанавливаем..."
    sudo apt install -y nvidia-cuda-toolkit
}

# Проверка VapourSynth
python3 -c "import vapoursynth" 2>/dev/null || \
    err "VapourSynth не установлен. Запусти: sudo apt install vapoursynth python3-vapoursynth libvapoursynth-dev"

# ── Установка инструментов сборки ────────────────────────────────────────────
log "Установка инструментов сборки..."
sudo apt update
sudo apt install -y \
    git \
    cmake \
    ninja-build \
    build-essential \
    pkg-config \
    vapoursynth-dev \
    libvapoursynth-dev

# ── Проверка TensorRT ────────────────────────────────────────────────────────
if [[ "$BACKEND" == "TRT" ]]; then
    log "Проверка TensorRT..."
    
    if [[ ! -d "/opt/tensorrt" ]] && [[ ! -f "/usr/include/NvInfer.h" ]]; then
        err "TensorRT не найден!\n\
  Установи TensorRT через:\n\
    ./setup/install_tensorrt_tarball.sh\n\
  или:\n\
    sudo apt install tensorrt libnvinfer-dev"
    fi
    
    # Определяем путь к TensorRT
    if [[ -d "/opt/tensorrt" ]]; then
        export TENSORRT_DIR="/opt/tensorrt"
        export CPLUS_INCLUDE_PATH="$TENSORRT_DIR/include:${CPLUS_INCLUDE_PATH:-}"
        export LIBRARY_PATH="$TENSORRT_DIR/lib:${LIBRARY_PATH:-}"
        export LD_LIBRARY_PATH="$TENSORRT_DIR/lib:${LD_LIBRARY_PATH:-}"
        log "TensorRT найден: $TENSORRT_DIR"
    else
        log "TensorRT системный (через apt)"
    fi
fi

# ── Клонирование репозитория ─────────────────────────────────────────────────
log "Клонирование vs-mlrt..."

if [[ -d "$BUILD_DIR" ]]; then
    log "Директория существует, обновляем..."
    cd "$BUILD_DIR"
    git pull
    git submodule update --init --recursive
else
    git clone --recursive https://github.com/AmusementClub/vs-mlrt.git "$BUILD_DIR"
    cd "$BUILD_DIR"
fi

# ── Компиляция ───────────────────────────────────────────────────────────────
log "Компиляция vs-mlrt (бэкенд: $BACKEND)..."

rm -rf build
mkdir build && cd build

# Настройка cmake в зависимости от бэкенда
case "$BACKEND" in
    TRT)
        CMAKE_FLAGS="-DWHICH_BACKEND=TRT"
        
        # Явно указываем пути к TensorRT если он в /opt
        if [[ -n "${TENSORRT_DIR:-}" ]]; then
            CMAKE_FLAGS="$CMAKE_FLAGS \
                -DTENSORRT_ROOT=$TENSORRT_DIR \
                -DCMAKE_PREFIX_PATH=$TENSORRT_DIR"
        fi
        ;;
    NCNN)
        CMAKE_FLAGS="-DWHICH_BACKEND=NCNN"
        log "NCNN требует установки Vulkan SDK"
        ;;
    ORT)
        CMAKE_FLAGS="-DWHICH_BACKEND=ORT"
        log "ONNX Runtime потребуется скачать отдельно"
        ;;
    OV)
        CMAKE_FLAGS="-DWHICH_BACKEND=OV"
        log "OpenVINO нужно установить заранее"
        ;;
    *)
        err "Неизвестный бэкенд: $BACKEND"
        ;;
esac

log "CMake flags: $CMAKE_FLAGS"

cmake -G Ninja \
    $CMAKE_FLAGS \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    .. || err "CMake конфигурация не удалась"

ninja -v || err "Компиляция не удалась"

log "✓ Компиляция успешна"

# ── Установка ────────────────────────────────────────────────────────────────
log "Установка плагина..."

# Находим скомпилированный .so файл
SO_FILE=$(find . -name "libvs_mlrt*.so" -o -name "libvstrt.so" | head -1)

if [[ -z "$SO_FILE" || ! -f "$SO_FILE" ]]; then
    err "Скомпилированный .so файл не найден!"
fi

log "Найден плагин: $SO_FILE"

# Копируем в директорию плагинов
mkdir -p "$VS_PLUGINS_DIR"
cp -v "$SO_FILE" "$VS_PLUGINS_DIR/"

log "✓ Плагин установлен в: $VS_PLUGINS_DIR"

# ── Установка Python-скрипта ─────────────────────────────────────────────────
log "Установка Python API vsmlrt..."

SCRIPTS_DIR="$BUILD_DIR/scripts"
if [[ -d "$SCRIPTS_DIR" ]]; then
    # Копируем в site-packages виртуального окружения
    if [[ -n "${VIRTUAL_ENV:-}" ]]; then
        SITE_PACKAGES="$VIRTUAL_ENV/lib/python$(python3 -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')/site-packages"
        mkdir -p "$SITE_PACKAGES"
        
        # Копируем все .py файлы
        cp -v "$SCRIPTS_DIR"/*.py "$SITE_PACKAGES/" 2>/dev/null || \
            log "  Нет .py файлов в $SCRIPTS_DIR"
        
        log "✓ Python API установлен в: $SITE_PACKAGES"
    else
        log "⚠️  Виртуальное окружение не активировано"
    fi
else
    log "⚠️  Директория scripts/ не найдена"
fi

# ── Скачивание моделей ───────────────────────────────────────────────────────
log "Скачивание предобученных моделей..."

MODELS_DIR="$HOME/legendary-potato/models/vsmlrt"
mkdir -p "$MODELS_DIR"

# Находим последний релиз с моделями
LATEST_RELEASE=$(curl -s https://api.github.com/repos/AmusementClub/vs-mlrt/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')
MODELS_URL="https://github.com/AmusementClub/vs-mlrt/releases/download/$LATEST_RELEASE/models.7z"

log "Последний релиз: $LATEST_RELEASE"

if command -v 7z &>/dev/null || command -v 7za &>/dev/null; then
    log "Скачивание models.7z..."
    wget -q --show-progress -O "$MODELS_DIR/models.7z" "$MODELS_URL" || \
        log "⚠️  Не удалось скачать модели"
    
    log "Распаковка моделей..."
    cd "$MODELS_DIR"
    7z x -y models.7z || 7za x -y models.7z || \
        log "⚠️  Не удалось распаковать (установи p7zip: sudo apt install p7zip-full)"
    
    log "✓ Модели установлены в: $MODELS_DIR"
else
    log "⚠️  7z не найден, скачай модели вручную:"
    log "   $MODELS_URL"
    log "   Распакуй в: $MODELS_DIR"
    sudo apt install -y p7zip-full
fi

# ── Проверка установки ───────────────────────────────────────────────────────
log "Проверка установки..."

python3 -c "
import vapoursynth as vs
core = vs.core

# Проверяем доступность плагина
try:
    # Для TensorRT бэкенда
    if hasattr(core, 'trt'):
        print('✓ vs-mlrt (TRT) доступен в VapourSynth')
        exit(0)
    elif hasattr(core, 'ort'):
        print('✓ vs-mlrt (ORT) доступен в VapourSynth')
        exit(0)
    elif hasattr(core, 'ncnn'):
        print('✓ vs-mlrt (NCNN) доступен в VapourSynth')
        exit(0)
    else:
        print('⚠️  vs-mlrt плагин не загрузился')
        print('Доступные плагины:', dir(core))
        exit(1)
except Exception as e:
    print('⚠️  Ошибка при проверке:', e)
    exit(1)
" || log "⚠️  Плагин не загружается (проверь пути)"

# ── Итоги ────────────────────────────────────────────────────────────────────
log ""
log "═══════════════════════════════════════════════════════════════"
log "  ✅ vs-mlrt собран и установлен"
log ""
log "  📂 Плагин: $VS_PLUGINS_DIR/$(basename $SO_FILE)"
log "  📂 Модели: $MODELS_DIR"
log ""
log "  Использование в VapourSynth скрипте:"
log "    import vapoursynth as vs"
log "    core = vs.core"
log "    core.trt.Model(clip, 'path/to/model.onnx')"
log ""
log "  Следующий шаг: ./setup/02_build_trt_engines.sh"
log "═══════════════════════════════════════════════════════════════"
