(venv) makleso6@makleso6:~/legendary-potato$ ./setup/01_setup_env.sh 
[setup] Virtual env: /home/makleso6/legendary-potato/venv ✓
[setup] CUDA: 13.0
[setup] Установка Python-пакетов...
Requirement already satisfied: pip in ./venv/lib/python3.12/site-packages (26.0.1)
Collecting wheel
  Using cached wheel-0.46.3-py3-none-any.whl.metadata (2.4 kB)
Collecting setuptools
  Using cached setuptools-82.0.1-py3-none-any.whl.metadata (6.5 kB)
Collecting packaging>=24.0 (from wheel)
  Using cached packaging-26.0-py3-none-any.whl.metadata (3.3 kB)
Using cached wheel-0.46.3-py3-none-any.whl (30 kB)
Using cached setuptools-82.0.1-py3-none-any.whl (1.0 MB)
Using cached packaging-26.0-py3-none-any.whl (74 kB)
Installing collected packages: setuptools, packaging, wheel
Successfully installed packaging-26.0 setuptools-82.0.1 wheel-0.46.3
Requirement already satisfied: numpy in ./venv/lib/python3.12/site-packages (2.4.4)
Requirement already satisfied: tqdm in ./venv/lib/python3.12/site-packages (4.67.3)
Requirement already satisfied: rich in ./venv/lib/python3.12/site-packages (14.3.3)
Requirement already satisfied: opencv-python in ./venv/lib/python3.12/site-packages (4.13.0.92)
Collecting Cython
  Using cached cython-3.2.4-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (7.5 kB)
Requirement already satisfied: scenedetect[opencv] in ./venv/lib/python3.12/site-packages (0.6.7.1)
Requirement already satisfied: markdown-it-py>=2.2.0 in ./venv/lib/python3.12/site-packages (from rich) (4.0.0)
Requirement already satisfied: pygments<3.0.0,>=2.13.0 in ./venv/lib/python3.12/site-packages (from rich) (2.20.0)
Requirement already satisfied: click<8.3.0,~=8.0 in ./venv/lib/python3.12/site-packages (from scenedetect[opencv]) (8.2.1)
Requirement already satisfied: platformdirs in ./venv/lib/python3.12/site-packages (from scenedetect[opencv]) (4.9.4)
Requirement already satisfied: mdurl~=0.1 in ./venv/lib/python3.12/site-packages (from markdown-it-py>=2.2.0->rich) (0.1.2)
Using cached cython-3.2.4-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (3.4 MB)
Installing collected packages: Cython
Successfully installed Cython-3.2.4
[setup] Проверка VapourSynth...
VapourSynth: R74
[setup] VS plugin dir: /home/makleso6/.local/lib/vapoursynth
[setup] Установка vsrepo...
Requirement already satisfied: vsrepo in ./venv/lib/python3.12/site-packages (1.0.0)
Requirement already satisfied: vapoursynth>=74rc1 in ./venv/lib/python3.12/site-packages (from vsrepo) (74rc2)
Requirement already satisfied: vsstubs>=1.1.2 in ./venv/lib/python3.12/site-packages (from vsrepo) (1.2.0)
Requirement already satisfied: tqdm in ./venv/lib/python3.12/site-packages (from vsrepo) (4.67.3)
Requirement already satisfied: rich>=14.3.0 in ./venv/lib/python3.12/site-packages (from vsstubs>=1.1.2->vsrepo) (14.3.3)
Requirement already satisfied: typer>=0.24.0 in ./venv/lib/python3.12/site-packages (from vsstubs>=1.1.2->vsrepo) (0.24.1)
Requirement already satisfied: typing-extensions>=4.15.0 in ./venv/lib/python3.12/site-packages (from vsstubs>=1.1.2->vsrepo) (4.15.0)
Requirement already satisfied: markdown-it-py>=2.2.0 in ./venv/lib/python3.12/site-packages (from rich>=14.3.0->vsstubs>=1.1.2->vsrepo) (4.0.0)
Requirement already satisfied: pygments<3.0.0,>=2.13.0 in ./venv/lib/python3.12/site-packages (from rich>=14.3.0->vsstubs>=1.1.2->vsrepo) (2.20.0)
Requirement already satisfied: mdurl~=0.1 in ./venv/lib/python3.12/site-packages (from markdown-it-py>=2.2.0->rich>=14.3.0->vsstubs>=1.1.2->vsrepo) (0.1.2)
Requirement already satisfied: click>=8.2.1 in ./venv/lib/python3.12/site-packages (from typer>=0.24.0->vsstubs>=1.1.2->vsrepo) (8.2.1)
Requirement already satisfied: shellingham>=1.3.0 in ./venv/lib/python3.12/site-packages (from typer>=0.24.0->vsstubs>=1.1.2->vsrepo) (1.5.4)
Requirement already satisfied: annotated-doc>=0.0.2 in ./venv/lib/python3.12/site-packages (from typer>=0.24.0->vsstubs>=1.1.2->vsrepo) (0.0.4)
[setup] Обновление базы плагинов vsrepo...
/home/makleso6/legendary-potato/venv/bin/python3: No module named vsrepo.__main__; 'vsrepo' is a package and cannot be directly executed
[setup]   vsrepo update не удался, пробуем инициализацию вручную...
[setup]   (пропускаем)
[setup] ⚠️  vsrepo не работает корректно, будем устанавливать плагины вручную
[setup] Установка плагинов вручную (vsrepo недоступен)...
[setup] Установка vs-mlrt...
ERROR: Could not find a version that satisfies the requirement vsmlrt (from versions: none)
ERROR: No matching distribution found for vsmlrt
[setup]   Установка vsmlrt через pip не удалась, пробуем из исходников...
Collecting git+https://github.com/AmusementClub/vs-mlrt.git
  Cloning https://github.com/AmusementClub/vs-mlrt.git to /tmp/pip-req-build-mam4puk_
  Running command git clone --filter=blob:none --quiet https://github.com/AmusementClub/vs-mlrt.git /tmp/pip-req-build-mam4puk_
  Resolved https://github.com/AmusementClub/vs-mlrt.git to commit 885e8bb827fc431fce8e3109e7d60b0c38aa2035
ERROR: git+https://github.com/AmusementClub/vs-mlrt.git does not appear to be a Python project: neither 'setup.py' nor 'pyproject.toml' found.
[setup]   ⚠️  vsmlrt Python package не установлен
./setup/01_setup_env.sh: строка 98: install_vsmlrt_manual: команда не найдена
