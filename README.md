(venv) makleso6@makleso6:~/legendary-potato$ ./setup/env.sh 

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  СИСТЕМНАЯ ИНФОРМАЦИЯ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Дистрибутив:
  Ubuntu 24.04.4 LTS
  Версия: 24.04

Ядро:
6.17.0-19-generic

Архитектура:
x86_64

Дата и время:
Вт 31 мар 2026 22:52:47 MSK

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PYTHON ОКРУЖЕНИЕ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Виртуальное окружение активно: /home/makleso6/legendary-potato/venv

Python версия:
Python 3.12.3
  Путь: /home/makleso6/legendary-potato/venv/bin/python3

pip версия:
pip 26.0.1 from /home/makleso6/legendary-potato/venv/lib/python3.12/site-packages/pip (python 3.12)

Установленные Python пакеты (основные):
✓ numpy: 2.4.4
✗ opencv-python не установлен
✓ tqdm: 4.67.3
✓ rich: unknown
✓ scenedetect: 0.6.7.1
✓ vapoursynth: R74
✓ tensorrt: 10.16.0.72
✓ vsrepo: 1.0.0

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  NVIDIA / CUDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ nvidia-smi найден

24689{NC} nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv,noheader
  NVIDIA GeForce RTX 5080, 580.126.09, 16303 MiB

CUDA версия из драйвера:
  13.0

✓ nvcc (CUDA Toolkit) найден
  Версия: 12.0,

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TENSORRT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ trtexec найден: /opt/tensorrt/bin/trtexec

24689{NC} trtexec --version 2>&1 | head -3
  &&&& RUNNING TensorRT.trtexec [TensorRT v101600] [b72] # trtexec --version
  [03/31/2026-22:52:48] [I] TF32 is enabled by default. Add --noTF32 flag to further improve accuracy with some performance cost.
  === Model Options ===

Поиск libnvinfer.so:
✓ /opt/tensorrt/lib/stubs/libnvinfer.so
✓ /opt/tensorrt/lib/libnvinfer.so.10
✓ /opt/tensorrt/lib/libnvinfer.so
✓ /opt/tensorrt/lib/libnvinfer.so.10.16.0

✓ TensorRT Python API: 10.16.0.72

✓ TensorRT директория: /opt/tensorrt
  Содержимое:
    итого 32
    drwxr-xr-x 8 makleso6 makleso6 4096 мар 31 22:14 .
    drwxr-xr-x 4 root     root     4096 мар 31 22:14 ..
    drwxr-xr-x 2 makleso6 makleso6 4096 мар 10 07:24 bin
    drwxr-xr-x 2 makleso6 makleso6 4096 мар 10 07:24 doc
    drwxr-xr-x 3 makleso6 makleso6 4096 мар 10 07:24 include
    drwxr-xr-x 3 makleso6 makleso6 4096 мар 10 07:24 lib
    drwxr-xr-x 2 makleso6 makleso6 4096 мар 10 07:24 python
    drwxr-xr-x 3 makleso6 makleso6 4096 мар 10 07:24 targets

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  VAPOURSYNTH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ VapourSynth: R74

Поиск vapoursynth библиотек:
  /usr/lib/x86_64-linux-gnu/libvapoursynth.so

Директории плагинов VapourSynth:
✓ /home/makleso6/.local/lib/vapoursynth
  Плагины (.so):
basename: пропущен операнд
По команде «basename --help» можно получить дополнительную информацию.
✓ /usr/lib/x86_64-linux-gnu/vapoursynth
  Плагины (.so):
    - libimwri.so
    - libocr.so
    - libremovegrain.so
    - libsubtext.so
    - libvivtc.so
⚠ /usr/local/lib/vapoursynth не существует

Тест загрузки плагинов в VapourSynth:
Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
AttributeError: module 'vapoursynth' has no attribute 'API_VERSION'

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  FFMPEG
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ ffmpeg найден: /usr/bin/ffmpeg

24689{NC} ffmpeg -version | head -3
  ffmpeg version 6.1.1-3ubuntu5 Copyright (c) 2000-2023 the FFmpeg developers
  built with gcc 13 (Ubuntu 13.2.0-23ubuntu3)
  configuration: --prefix=/usr --extra-version=3ubuntu5 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --disable-omx --enable-gnutls --enable-libaom --enable-libass --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libdav1d --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libglslang --enable-libgme --enable-libgsm --enable-libharfbuzz --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-openal --enable-opencl --enable-opengl --disable-sndio --enable-libvpl --disable-libmfx --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-ladspa --enable-libbluray --enable-libjack --enable-libpulse --enable-librabbitmq --enable-librist --enable-libsrt --enable-libssh --enable-libsvtav1 --enable-libx264 --enable-libzmq --enable-libzvbi --enable-lv2 --enable-sdl2 --enable-libplacebo --enable-librav1e --enable-pocketsphinx --enable-librsvg --enable-libjxl --enable-shared

Проверка фильтров ffmpeg:
✓ vidstabdetect
✓ vidstabtransform
✓ scale_cuda
⚠ scale_npp отсутствует

Кодеки NVIDIA (nvenc):
✓ h264_nvenc доступен

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ДОПОЛНИТЕЛЬНЫЕ УТИЛИТЫ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ exiftool: /usr/bin/exiftool
✓ mediainfo: /usr/bin/mediainfo
✓ 7z: /usr/bin/7z
✓ git: /usr/bin/git
⚠ cmake не найден
⚠ ninja не найден
✓ wget: /usr/bin/wget
⚠ curl не найден

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  СТРУКТУРА ПРОЕКТА
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Директория проекта: /home/makleso6/legendary-potato

Содержимое:
  итого 9941508
  drwxrwxr-x  9 makleso6 makleso6       4096 мар 31 22:37 .
  drwxr-x--- 19 makleso6 makleso6       4096 мар 31 22:25 ..
  -rw-rw-r--  1 makleso6 makleso6      19755 мар 31 21:09 Brainstorm.md
  drwxrwxr-x  2 makleso6 makleso6       4096 мар 31 21:09 .claude
  -rw-rw-r--  1 makleso6 makleso6       4332 апр 20  2023 cuda-keyring_1.1-1_all.deb
  -rwxrwxr-x  1 makleso6 makleso6      13534 мар 31 21:09 enhance_concert.sh
  drwxrwxr-x  8 makleso6 makleso6       4096 мар 31 21:09 .git
  -rw-rw-r--  1 makleso6 makleso6       1239 мар 31 21:09 .gitignore
  drwxrwxr-x  2 makleso6 makleso6       4096 мар 31 21:36 models
  -rw-rw-r--  1 makleso6 makleso6       8272 мар 31 21:09 pipeline.py
  drwxrwxr-x  2 makleso6 makleso6       4096 мар 31 22:52 setup
  -rw-rw-r--  1 makleso6 makleso6 7568039306 мар 31 22:03 TensorRT-10.16.0.72.Linux.x86_64-gnu.cuda-13.2.tar.gz
  drwxrwxr-x  2 makleso6 makleso6       4096 мар 31 21:09 utils
  drwxrwxr-x  5 makleso6 makleso6       4096 мар 31 21:18 venv
  drwxrwxr-x  2 makleso6 makleso6       4096 мар 26 04:51 vsmlrt-cuda
  -rw-rw-r--  1 makleso6 makleso6 2147483647 мар 31 22:33 vsmlrt-cuda.v15.16.7z.001
  -rw-rw-r--  1 makleso6 makleso6  464467988 мар 31 22:33 vsmlrt-cuda.v15.16.7z.002

Директория setup/:
  итого 68
  drwxrwxr-x 2 makleso6 makleso6  4096 мар 31 22:52 .
  drwxrwxr-x 9 makleso6 makleso6  4096 мар 31 22:37 ..
  -rwxrwxr-x 1 makleso6 makleso6  9726 мар 31 22:30 01_setup_env.sh
  -rwxrwxr-x 1 makleso6 makleso6  5207 мар 31 21:09 02_build_trt_engines.sh
  -rwxrwxr-x 1 makleso6 makleso6  8016 мар 31 22:13 03.sh
  -rwxrwxr-x 1 makleso6 makleso6 10407 мар 31 22:45 build_vsmlrt.sh
  -rwxrwxr-x 1 makleso6 makleso6 17670 мар 31 22:52 env.sh

Директория models/:
  ONNX модели:
  TensorRT engines:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PATH:
  /home/makleso6/legendary-potato/venv/bin
  /opt/tensorrt/bin
  /opt/tensorrt/bin
  /opt/tensorrt/bin
  /usr/local/sbin
  /usr/local/bin
  /usr/sbin
  /usr/bin
  /sbin
  /bin
  /usr/games
  /usr/local/games
  /snap/bin
  /snap/bin

LD_LIBRARY_PATH:
  /opt/tensorrt/lib
  
  /opt/tensorrt/lib
  /opt/tensorrt/lib
  

⚠ PYTHONPATH не установлена

⚠ CUDA_HOME не установлена

TENSORRT_DIR:
  /opt/tensorrt

VIRTUAL_ENV:
  /home/makleso6/legendary-potato/venv


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ИТОГОВАЯ СВОДКА
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Критичные компоненты:
✓ Python 3
✓ Virtual environment
✓ NVIDIA драйвер
✓ trtexec (TensorRT)
✓ VapourSynth
✓ ffmpeg
✓ PySceneDetect

Опциональные компоненты:
✓ vid.stab (стабилизация)
⚠ vs-mlrt (для VapourSynth inference)

════════════════════════════════════════════════════════════
  ✓ ВСЕ КРИТИЧНЫЕ КОМПОНЕНТЫ УСТАНОВЛЕНЫ
════════════════════════════════════════════════════════════

Готов к запуску: ./setup/02_build_trt_engines.sh

════════════════════════════════════════════════════════════
Сохрани вывод этого скрипта и отправь для анализа
════════════════════════════════════════════════════════════
