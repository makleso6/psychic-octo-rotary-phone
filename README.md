cat > setup/diagnose.sh <<'SCRIPT'
#!/usr/bin/env bash
# setup/diagnose.sh - Полная диагностика окружения для concert video enhancement
set -u

# Цвета
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

print_header() {
    echo -e "\n${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}  $1${NC}"
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
}

check_ok() {
    echo -e "${GREEN}✓${NC} $1"
}

check_warn() {
    echo -e "${YELLOW}⚠${NC} $1"
}

check_fail() {
    echo -e "${RED}✗${NC} $1"
}

run_cmd() {
    echo -e "${BLUE}$${NC} $1"
    eval "$1" 2>&1 | sed 's/^/  /'
}

# ══════════════════════════════════════════════════════════════════════════════
# СИСТЕМНАЯ ИНФОРМАЦИЯ
# ══════════════════════════════════════════════════════════════════════════════

print_header "СИСТЕМНАЯ ИНФОРМАЦИЯ"

echo "Дистрибутив:"
if [ -f /etc/os-release ]; then
    source /etc/os-release
    echo "  $PRETTY_NAME"
    echo "  Версия: $VERSION_ID"
else
    echo "  Неизвестно"
fi

echo ""
echo "Ядро:"
uname -r

echo ""
echo "Архитектура:"
uname -m

echo ""
echo "Дата и время:"
date

# ══════════════════════════════════════════════════════════════════════════════
# PYTHON ОКРУЖЕНИЕ
# ══════════════════════════════════════════════════════════════════════════════

print_header "PYTHON ОКРУЖЕНИЕ"

if [ -n "${VIRTUAL_ENV:-}" ]; then
    check_ok "Виртуальное окружение активно: $VIRTUAL_ENV"
else
    check_warn "Виртуальное окружение НЕ активно"
fi

echo ""
echo "Python версия:"
if command -v python3 &>/dev/null; then
    python3 --version
    echo "  Путь: $(which python3)"
else
    check_fail "python3 не найден"
fi

echo ""
echo "pip версия:"
if command -v pip3 &>/dev/null || command -v pip &>/dev/null; then
    pip --version 2>/dev/null || pip3 --version
else
    check_fail "pip не найден"
fi

echo ""
echo "Установленные Python пакеты (основные):"
for pkg in numpy opencv-python tqdm rich scenedetect vapoursynth tensorrt vsrepo; do
    if python3 -c "import $pkg" 2>/dev/null; then
        version=$(python3 -c "import $pkg; print(getattr($pkg, '__version__', 'unknown'))" 2>/dev/null)
        check_ok "$pkg: $version"
    else
        check_fail "$pkg не установлен"
    fi
done

# ══════════════════════════════════════════════════════════════════════════════
# NVIDIA / CUDA
# ══════════════════════════════════════════════════════════════════════════════

print_header "NVIDIA / CUDA"

if command -v nvidia-smi &>/dev/null; then
    check_ok "nvidia-smi найден"
    echo ""
    run_cmd "nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv,noheader"
    echo ""
    echo "CUDA версия из драйвера:"
    nvidia-smi | grep "CUDA Version" | awk '{print "  "$9}'
else
    check_fail "nvidia-smi не найден (NVIDIA драйвер не установлен)"
fi

echo ""
if command -v nvcc &>/dev/null; then
    check_ok "nvcc (CUDA Toolkit) найден"
    nvcc --version | grep "release" | awk '{print "  Версия: "$5}'
else
    check_warn "nvcc не найден (CUDA Toolkit не установлен)"
fi

# ══════════════════════════════════════════════════════════════════════════════
# TENSORRT
# ══════════════════════════════════════════════════════════════════════════════

print_header "TENSORRT"

# trtexec
if command -v trtexec &>/dev/null; then
    check_ok "trtexec найден: $(which trtexec)"
    echo ""
    run_cmd "trtexec --version 2>&1 | head -3"
else
    check_fail "trtexec НЕ НАЙДЕН (критично для сборки engines)"
fi

echo ""
# TensorRT библиотеки
echo "Поиск libnvinfer.so:"
LIBNVINFER=$(find /usr /opt -name "libnvinfer.so*" 2>/dev/null | head -5)
if [ -n "$LIBNVINFER" ]; then
    echo "$LIBNVINFER" | while read line; do
        check_ok "$line"
    done
else
    check_fail "libnvinfer.so не найден"
fi

echo ""
# Python API
if python3 -c "import tensorrt" 2>/dev/null; then
    TRT_VER=$(python3 -c "import tensorrt; print(tensorrt.__version__)" 2>/dev/null)
    check_ok "TensorRT Python API: $TRT_VER"
else
    check_warn "TensorRT Python API не установлен (не критично)"
fi

echo ""
# TensorRT директория
if [ -d "/opt/tensorrt" ]; then
    check_ok "TensorRT директория: /opt/tensorrt"
    echo "  Содержимое:"
    ls -la /opt/tensorrt/ 2>/dev/null | head -10 | sed 's/^/    /'
fi

# ══════════════════════════════════════════════════════════════════════════════
# VAPOURSYNTH
# ══════════════════════════════════════════════════════════════════════════════

print_header "VAPOURSYNTH"

if python3 -c "import vapoursynth" 2>/dev/null; then
    VS_VER=$(python3 -c "import vapoursynth; print(vapoursynth.__version__)" 2>/dev/null)
    check_ok "VapourSynth: $VS_VER"
else
    check_fail "VapourSynth не установлен"
fi

echo ""
echo "Поиск vapoursynth библиотек:"
find /usr -name "libvapoursynth.so*" 2>/dev/null | head -3 | while read line; do
    echo "  $line"
done

echo ""
echo "Директории плагинов VapourSynth:"
VS_PLUGIN_DIRS=(
    "$HOME/.local/lib/vapoursynth"
    "/usr/lib/x86_64-linux-gnu/vapoursynth"
    "/usr/local/lib/vapoursynth"
)

for dir in "${VS_PLUGIN_DIRS[@]}"; do
    if [ -d "$dir" ]; then
        check_ok "$dir"
        echo "  Плагины (.so):"
        ls -1 "$dir"/*.so 2>/dev/null | xargs -n1 basename | sed 's/^/    - /' || echo "    (нет .so файлов)"
    else
        check_warn "$dir не существует"
    fi
done

echo ""
echo "Тест загрузки плагинов в VapourSynth:"
python3 <<'PYEOF'
import vapoursynth as vs
core = vs.core

print(f"  VapourSynth API: {vs.API_VERSION}")
print(f"  Число потоков: {core.num_threads}")
print(f"\n  Загруженные плагины:")

# Попытка вызвать core.plugins() (API v4) или перебор атрибутов
try:
    plugins = core.plugins()
    for p in plugins:
        print(f"    - {p}")
except:
    # Альтернативный способ для старых версий
    attrs = [x for x in dir(core) if not x.startswith('_')]
    for attr in attrs[:20]:  # первые 20
        print(f"    - {attr}")

# Проверка конкретных плагинов
checks = {
    'ffms2': 'core.ffms2',
    'trt': 'core.trt',
    'ort': 'core.ort',
    'bm3d': 'core.bm3d',
    'rife': 'core.rife'
}

print(f"\n  Проверка специфичных плагинов:")
for name, attr_path in checks.items():
    try:
        exec(f"_ = {attr_path}")
        print(f"    ✓ {name} доступен")
    except:
        print(f"    ✗ {name} недоступен")
PYEOF

# ══════════════════════════════════════════════════════════════════════════════
# FFMPEG
# ══════════════════════════════════════════════════════════════════════════════

print_header "FFMPEG"

if command -v ffmpeg &>/dev/null; then
    check_ok "ffmpeg найден: $(which ffmpeg)"
    echo ""
    run_cmd "ffmpeg -version | head -3"
else
    check_fail "ffmpeg не найден"
fi

echo ""
echo "Проверка фильтров ffmpeg:"
FILTERS=("vidstabdetect" "vidstabtransform" "scale_cuda" "scale_npp")
for filter in "${FILTERS[@]}"; do
    if ffmpeg -filters 2>&1 | grep -q "^.*$filter"; then
        check_ok "$filter"
    else
        check_warn "$filter отсутствует"
    fi
done

echo ""
echo "Кодеки NVIDIA (nvenc):"
if ffmpeg -codecs 2>&1 | grep -q "h264_nvenc"; then
    check_ok "h264_nvenc доступен"
else
    check_warn "h264_nvenc недоступен"
fi

# ══════════════════════════════════════════════════════════════════════════════
# ДОПОЛНИТЕЛЬНЫЕ УТИЛИТЫ
# ══════════════════════════════════════════════════════════════════════════════

print_header "ДОПОЛНИТЕЛЬНЫЕ УТИЛИТЫ"

TOOLS=("exiftool" "mediainfo" "7z" "git" "cmake" "ninja" "wget" "curl")
for tool in "${TOOLS[@]}"; do
    if command -v "$tool" &>/dev/null; then
        check_ok "$tool: $(which $tool)"
    else
        check_warn "$tool не найден"
    fi
done

# ══════════════════════════════════════════════════════════════════════════════
# СТРУКТУРА ПРОЕКТА
# ══════════════════════════════════════════════════════════════════════════════

print_header "СТРУКТУРА ПРОЕКТА"

PROJECT_ROOT="$HOME/legendary-potato"
if [ -d "$PROJECT_ROOT" ]; then
    check_ok "Директория проекта: $PROJECT_ROOT"
    echo ""
    echo "Содержимое:"
    ls -la "$PROJECT_ROOT" | sed 's/^/  /'
    
    echo ""
    echo "Директория setup/:"
    if [ -d "$PROJECT_ROOT/setup" ]; then
        ls -la "$PROJECT_ROOT/setup" | sed 's/^/  /'
    else
        check_warn "setup/ не существует"
    fi
    
    echo ""
    echo "Директория models/:"
    if [ -d "$PROJECT_ROOT/models" ]; then
        echo "  ONNX модели:"
        ls -lh "$PROJECT_ROOT/models/onnx"/*.onnx 2>/dev/null | awk '{print "    "$9" ("$5")"}' || echo "    (нет .onnx файлов)"
        echo "  TensorRT engines:"
        ls -lh "$PROJECT_ROOT/models/engines"/*.engine 2>/dev/null | awk '{print "    "$9" ("$5")"}' || echo "    (нет .engine файлов)"
    else
        check_warn "models/ не существует"
    fi
else
    check_fail "Директория проекта не найдена: $PROJECT_ROOT"
fi

# ══════════════════════════════════════════════════════════════════════════════
# ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ
# ══════════════════════════════════════════════════════════════════════════════

print_header "ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ"

ENVS=("PATH" "LD_LIBRARY_PATH" "PYTHONPATH" "CUDA_HOME" "TENSORRT_DIR" "VIRTUAL_ENV")
for env in "${ENVS[@]}"; do
    if [ -n "${!env:-}" ]; then
        echo "$env:"
        echo "${!env}" | tr ':' '\n' | sed 's/^/  /'
    else
        check_warn "$env не установлена"
    fi
    echo ""
done

# ══════════════════════════════════════════════════════════════════════════════
# ИТОГОВАЯ СВОДКА
# ══════════════════════════════════════════════════════════════════════════════

print_header "ИТОГОВАЯ СВОДКА"

echo "Критичные компоненты:"
CRITICAL=0

if command -v python3 &>/dev/null; then
    check_ok "Python 3"
else
    check_fail "Python 3"
    CRITICAL=$((CRITICAL+1))
fi

if [ -n "${VIRTUAL_ENV:-}" ]; then
    check_ok "Virtual environment"
else
    check_warn "Virtual environment (рекомендуется)"
fi

if command -v nvidia-smi &>/dev/null; then
    check_ok "NVIDIA драйвер"
else
    check_fail "NVIDIA драйвер"
    CRITICAL=$((CRITICAL+1))
fi

if command -v trtexec &>/dev/null; then
    check_ok "trtexec (TensorRT)"
else
    check_fail "trtexec (TensorRT) - НЕОБХОДИМ ДЛЯ СБОРКИ ENGINES"
    CRITICAL=$((CRITICAL+1))
fi

if python3 -c "import vapoursynth" 2>/dev/null; then
    check_ok "VapourSynth"
else
    check_fail "VapourSynth"
    CRITICAL=$((CRITICAL+1))
fi

if command -v ffmpeg &>/dev/null; then
    check_ok "ffmpeg"
else
    check_fail "ffmpeg"
    CRITICAL=$((CRITICAL+1))
fi

if python3 -c "import scenedetect" 2>/dev/null; then
    check_ok "PySceneDetect"
else
    check_warn "PySceneDetect (необязательно)"
fi

echo ""
echo "Опциональные компоненты:"

if ffmpeg -filters 2>&1 | grep -q vidstabdetect; then
    check_ok "vid.stab (стабилизация)"
else
    check_warn "vid.stab (улучшит стабилизацию)"
fi

if python3 -c "from vsmlrt import Backend" 2>/dev/null; then
    check_ok "vs-mlrt (inference через VapourSynth)"
else
    check_warn "vs-mlrt (для VapourSynth inference)"
fi

echo ""
if [ $CRITICAL -eq 0 ]; then
    echo -e "${GREEN}════════════════════════════════════════════════════════════${NC}"
    echo -e "${GREEN}  ✓ ВСЕ КРИТИЧНЫЕ КОМПОНЕНТЫ УСТАНОВЛЕНЫ${NC}"
    echo -e "${GREEN}════════════════════════════════════════════════════════════${NC}"
    echo ""
    echo "Готов к запуску: ./setup/02_build_trt_engines.sh"
else
    echo -e "${RED}════════════════════════════════════════════════════════════${NC}"
    echo -e "${RED}  ✗ ОТСУТСТВУЮТ $CRITICAL КРИТИЧНЫХ КОМПОНЕНТА(-ОВ)${NC}"
    echo -e "${RED}════════════════════════════════════════════════════════════${NC}"
    echo ""
    echo "Рекомендации:"
    
    if ! command -v trtexec &>/dev/null; then
        echo "  1. Установи TensorRT: ./setup/install_tensorrt_tarball.sh"
    fi
    
    if ! python3 -c "import vapoursynth" 2>/dev/null; then
        echo "  2. Установи VapourSynth: sudo apt install vapoursynth python3-vapoursynth"
    fi
    
    if ! command -v ffmpeg &>/dev/null; then
        echo "  3. Установи ffmpeg: sudo apt install ffmpeg"
    fi
fi

echo ""
echo -e "${BLUE}════════════════════════════════════════════════════════════${NC}"
echo "Сохрани вывод этого скрипта и отправь для анализа"
echo -e "${BLUE}════════════════════════════════════════════════════════════${NC}"
SCRIPT

chmod +x setup/diagnose.sh
