[setup] VapourSynth не найден. Устанавливаем через pip...
Collecting vapoursynth
  Downloading vapoursynth-73.tar.gz (67 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Building wheels for collected packages: vapoursynth
  Building wheel for vapoursynth (pyproject.toml) ... error
  error: subprocess-exited-with-error
  
  × Building wheel for vapoursynth (pyproject.toml) did not run successfully.
  │ exit code: 1
  ╰─> [35 lines of output]
      running bdist_wheel
      running build
      running build_py
      running egg_info
      writing src/VapourSynth.egg-info/PKG-INFO
      writing dependency_links to src/VapourSynth.egg-info/dependency_links.txt
      writing top-level names to src/VapourSynth.egg-info/top_level.txt
      reading manifest file 'src/VapourSynth.egg-info/SOURCES.txt'
      reading manifest template 'MANIFEST.in'
      warning: no files found matching '*.asm' under directory 'include'
      warning: no previously-included files matching '*' found under directory 'include/cython'
      warning: no previously-included files matching '*.dll' found anywhere in distribution
      adding license file 'COPYING.LESSER'
      writing manifest file 'src/VapourSynth.egg-info/SOURCES.txt'
      creating build/lib.linux-x86_64-cpython-312/cython
      copying src/cython/vapoursynth.pxd -> build/lib.linux-x86_64-cpython-312/cython
      copying src/cython/vapoursynth.pyx -> build/lib.linux-x86_64-cpython-312/cython
      copying src/cython/vsconstants.pxd -> build/lib.linux-x86_64-cpython-312/cython
      copying src/cython/vsscript.pxd -> build/lib.linux-x86_64-cpython-312/cython
      copying src/cython/vsscript_internal.pxd -> build/lib.linux-x86_64-cpython-312/cython
      creating build/lib.linux-x86_64-cpython-312/vsscript
      copying src/vsscript/vsscript_internal.h -> build/lib.linux-x86_64-cpython-312/vsscript
      running build_ext
      Compiling src/cython/vapoursynth.pyx because it changed.
      [1/1] Cythonizing src/cython/vapoursynth.pyx
      building 'vapoursynth' extension
      creating build/temp.linux-x86_64-cpython-312/src/cython
      x86_64-linux-gnu-gcc -fno-strict-overflow -Wsign-compare -DNDEBUG -g -O2 -Wall -fPIC -DPy_LIMITED_API=51118080 -DVS_USE_LATEST_API -DVS_GRAPH_API -DVS_CURRENT_RELEASE=73 -I. -Isrc/cython -Isrc/vsscript -I/home/makleso6/legendary-potato/venv/include -I/usr/include/python3.12 -c src/cython/vapoursynth.c -o build/temp.linux-x86_64-cpython-312/src/cython/vapoursynth.o
      src/cython/vapoursynth.c:91551:18: warning: ‘__pyx_f_11vapoursynth__vsscript_use_or_create_environment’ defined but not used [-Wunused-function]
      91551 | static PyObject *__pyx_f_11vapoursynth__vsscript_use_or_create_environment(int __pyx_v_id) {
            |                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      x86_64-linux-gnu-gcc -shared -Wl,-O1 -Wl,-Bsymbolic-functions -Wl,-Bsymbolic-functions -Wl,-z,relro -g -fwrapv -O2 build/temp.linux-x86_64-cpython-312/src/cython/vapoursynth.o -L. -Lbuild -L/usr/lib/x86_64-linux-gnu -lvapoursynth -o build/lib.linux-x86_64-cpython-312/vapoursynth.abi3.so
      /usr/bin/ld: невозможно найти -lvapoursynth: Нет такого файла или каталога
      collect2: error: ld returned 1 exit status
      error: command '/usr/bin/x86_64-linux-gnu-gcc' failed with exit code 1
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for vapoursynth
Failed to build vapoursynth
error: failed-wheel-build-for-install

× Failed to build installable wheels for some pyproject.toml based projects
╰─> vapoursynth
