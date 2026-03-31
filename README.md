(venv) makleso6@makleso6:~/legendary-potato$ ./setup/01_setup_env.sh 
[setup] CUDA: 13.0
[setup] Установка Python-пакетов...
Requirement already satisfied: pip in ./venv/lib/python3.12/site-packages (26.0.1)
Requirement already satisfied: numpy in ./venv/lib/python3.12/site-packages (2.4.4)
Requirement already satisfied: tqdm in ./venv/lib/python3.12/site-packages (4.67.3)
Requirement already satisfied: rich in ./venv/lib/python3.12/site-packages (14.3.3)
Requirement already satisfied: scenedetect[opencv] in ./venv/lib/python3.12/site-packages (0.6.7.1)
Requirement already satisfied: click<8.3.0,~=8.0 in ./venv/lib/python3.12/site-packages (from scenedetect[opencv]) (8.2.1)
Requirement already satisfied: platformdirs in ./venv/lib/python3.12/site-packages (from scenedetect[opencv]) (4.9.4)
Requirement already satisfied: opencv-python in ./venv/lib/python3.12/site-packages (from scenedetect[opencv]) (4.13.0.92)
Requirement already satisfied: markdown-it-py>=2.2.0 in ./venv/lib/python3.12/site-packages (from rich) (4.0.0)
Requirement already satisfied: pygments<3.0.0,>=2.13.0 in ./venv/lib/python3.12/site-packages (from rich) (2.20.0)
Requirement already satisfied: mdurl~=0.1 in ./venv/lib/python3.12/site-packages (from markdown-it-py>=2.2.0->rich) (0.1.2)
[setup] Проверка VapourSynth...
VS OK: R74
[setup] VS plugin dir (диагностика): /home/makleso6/legendary-potato/venv/lib/python3.12/site-packages/vapoursynth
[setup] Установка ffms2...
Failed to open vspackages3.json. Run update command.
[setup]   (ffms2 уже установлен или требует ручной установки)
[setup] Установка vs-bm3d (CUDA)...
Failed to open vspackages3.json. Run update command.
[setup]   vsrepo не нашёл bm3dcuda, пробуем pip...
[setup]   Установи vs-bm3dcuda вручную с https://github.com/WolframRhodium/VapourSynth-BM3DCUDA
[setup] Установка vs-mlrt...
[setup]   vsmlrt Python-обёртка не установилась (попробуй вручную)
Failed to open vspackages3.json. Run update command.
[setup]   Установи vs-mlrt вручную: https://github.com/AmusementClub/vs-mlrt
[setup] Установка VapourSynth-RIFE...
Failed to open vspackages3.json. Run update command.
[setup]   Установи RIFE plugin вручную: https://github.com/HomeOfVapourSynthEvolution/VapourSynth-RIFE
[setup] Проверка TensorRT...
TRT OK: 10.16.0.72
[setup] Проверка vid.stab...
[setup]   vid.stab не найден в ffmpeg. Устанавливаем ffmpeg с vid.stab...
[setup]   vid.stab не найден. Варианты:
[setup]   1) PPA (рекомендуется): sudo add-apt-repository ppa:savoury1/ffmpeg4 && sudo apt-get update && sudo apt-get install ffmpeg
[setup]   2) Собери ffmpeg с --enable-libvidstab вручную: https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
[setup]   Стабилизация через VidStab будет недоступна до установки!
[setup] Проверка Gyroflow CLI...
[setup]   Gyroflow не найден (опционально). Скачай с https://gyroflow.xyz/download
[setup]   Помести gyroflow в PATH или укажи GYROFLOW_BIN в enhance_concert.sh
[setup] Проверка PySceneDetect...
SceneDetect OK: 0.6.7.1
[setup] 
[setup] ═══════════════════════════════════════════════════════════════
[setup]   Окружение настроено. Следующий шаг: setup/02_build_trt_engines.sh
[setup] ═══════════════════════════════════════════════════════════════
(venv) makleso6@makleso6:~/legendary-potato$ ./setup/02_build_trt_engines.sh
[trt-build] Параметры: tile=384, overlap=24, precision=fp16
[trt-build] Директория моделей: /home/makleso6/legendary-potato/models
[ERROR] vs-mlrt не установлен. Запусти 01_setup_env.sh.
