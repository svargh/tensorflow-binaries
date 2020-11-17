# tensorflow-binaries
Prebuilt Tensorflow binaries for my laptop with this configuration.

I am not using the most recent NVidia Driver because they are very unstable when doing docking/undocking/sleeping/resuming.

# Hardware config

- Dell Precision 15' 7540 128 ECC RAM Xeon E-2286M@2.4GHz
- NVIDIA Quadro RX 3000 Driver Version 26.21.14.4283 (442.83)
  Cannot use Cuda 11 Toolit because Driver is too old. 
  Cuda 10.2 should be usable, but not compilable.
- NVidia Driver is form Dell which is very stable, but not to most recent one

# Software Config:
on Host PC:
- Install same NVidia Software as on Host - see below.
  Important: DO NOT INSTALL the supplied nvidia drivers (not stable for docking/undocking/sleeping/hibernate/monitor changes etc), just install the SDK 
  
On Dev VirtualMachine VMWare:
- Install same NVidia Software as on Host. Also here: Do not install the NVIDIA Drivers 
- MS Visual Studio 2019 Buildtools (Windows Universal C Runtime, MSVC v142 x68/86 build tools..., C++ 2019 Redistributable Update, Win 10 SDK 10.0.18362.0 C++ 
  CMake 
  toosl for Windows, C++ 2019 Redistributable MSMs)
- Python 3.7.9 virtual enviroment for each TF Version
- Bazel 3.7.0
- MSys64 Version https://repo.msys2.org/distrib/x86_64/msys2-x86_64-20200903.exe
- 300GB Disk space for building TF2.3 and T.4

# How to use the libraries on Host with NVIDIA GPU

Examples Steps for using Tensorflow 2.4 whl binary: 
Install these software:
- cuda_10.1.243_426.00_win10.exe - IMPORTANT: DO NOT INSTALL THe SUPPLIED NVIDA DRIVER - NOT STABLE FOR DAILY USE or LONG UPTIME
- cudnn-10.1-windows10-x64-v7.6.5.32.zip (Content of of subfolders must be copied into subfolders of CUDA Tookit folder)
- TensorRT-6.0.1.5.Windows10.x86_64.cuda-10.1.cudnn7.6.zip
- Tested on Host Windows 10 1909
- THen do this:
```
set PDIR=c:\dev\python\Python37
set PATH=%PDIR%;%PDIR%\Scripts;%PATH%
cd c:\dev\tensorflow

--- Do once ---
mkdir tf2.4
cd tf2.4
pip install virtualenv
virtualenv venv
--- ---

venv\Scripts\activate
pip install e:\ShareData2\tensorflow-binaries\tensorflow-2.4.0rc2-cp37-cp37m-win_amd64.whl
python ..\gpu.py
python ..\gpu2.py
```



# How the binaries have been built - Build Process in VM - not relevant for using the library:
- Install NVida software as stated above - except the NVIDIA hardware drivers

## Create local isolated python enviroment
```

set PDIR=c:\dev\python\Python37
set PATH=%PDIR%;%PDIR%\Scripts;%PATH%


cd c:\dev\tensorflow\tf2.4

python -m pip install --upgrade pip
pip install virtualenv
virtualenv venv
venv\Scripts\activate
cd tensorflow

pip install six h5py numpy wheel
pip install keras_applications==1.0.6 --no-deps
pip install keras_preprocessing # After build error for next time	


git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
git checkout r2.4

## Secondtime / Reoccuring actions
set PDIR=c:\dev\python\Python37
set PATH=%PDIR%;%PDIR%\Scripts;%PATH%
cd C:\dev\tensorflow\tf2.4   
cd tensorflow     
venv\Scripts\activate
cd tensorflow
If needed, update or switch branch, but clean has to be done! 
git pull etc

## Configure
python ./configure.py # Choose Cuda Compute Capability 7.5. It will detect CUDA directories
- It will detect nvdia cuda directories
           

## Build        
In case versions have changed: bazel clean --expunge

           
bazel build --local_cpu_resources=HOST_CPUS-2 --config=opt --config=cuda --define=no_tensorflow_py_deps=true //tensorflow/tools/pip_package:build_pip_package


## Build whl package
(venv) c:\dev\tensorflow\tf2.4\tensorflow>
bazel-bin\tensorflow\tools\pip_package\build_pip_package c:\temp\tensorflow_pkg

## Build errors
- .\tensorflow/core/util/cuda_sparse.h(39): error C2061: syntax error: identifier 'cusparseDnMatDescr_t'
Incompatible Cuda Versionfor given tensorflow version
Reason:  https://github.com/tensorflow/tensorflow/issues/40859 -> variables not available when on x64 then it is 10.2
Solution: Use Cuda 10.1 on windows

- Create cuda local repo problem: 
Solution: MSVC Tools not all installed (also WinSdk) See Visual Studio Installer Screenshots above what to select


- patch.exe Exec format error: Download & Reinstall msys2 again


- Original error was: No module named 'numpy.core._multiarray_umath': 
Solution: pip3 install h5py

- .\tensorflow/core/util/cuda_sparse.h(39): error C2061: syntax error: identifier 'cusparseDnMatDescr_t'
Incompatible Cuda Versionfor given tensorflow version
Reason:  https://github.com/tensorflow/tensorflow/issues/40859 -> variables not available when on x64 then it is 10.2
Solution: Use Cuda 10.1 on windows


- TF2.3 Windows: buildcc error: 
NOT WORKING: pip install numpy<1.18.5
NOT WORKING:  pip install numpy==1.18.0
Works: modify  tensorflow\python\lib\core\bfloat16.cc 
‘Around line # 508, is the function CompareUFunc, add "const" to the parameters, like this:’ https://github.com/tensorflow/tensorflow/issues/41086


- Windows: Final build error:
ModuleNotFoundError: No module named 'keras_preprocessing'
pip install keras_preprocessing
pip install numpy==1.18.0

```
