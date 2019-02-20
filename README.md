# Building DeepSpeech for Windows

Now we can build the native DeepSpeech library and run inference on Windows using the C# client.

**Table of Contents**

- [Prerequisites](#prerequisites)
- [Getting the code](#getting-the-code)
- [Configuring the paths](#configuring-the-paths)
- [Adding environment variables](#adding-environment-variables)
- [Building the code](#building-the-code)
    - [Build for CPU](#cpu)
    - [Build with CUDA support](#gpu-with-cuda)

## Prerequisites

* [Python 3.6](https://www.python.org/)
* [Git Large File Storage](https://git-lfs.github.com/)
* [MSYS2](https://www.msys2.org/)
* [Bazel v0.17.2](https://github.com/bazelbuild/bazel/releases)
* [Windows 10 SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk)
* Windows 10
* [Visual Studio 2017 Community](https://visualstudio.microsoft.com/vs/community/) 

Inside the Visual Studio Installer enable `MS Build Tools` and `VC++ 2015.3 v14.00 (v140) toolset for desktop`

If you want to enable CUDA support you need to install:

* [CUDA 9.0 and cuDNN 7.3.1](https://developer.nvidia.com/cuda-90-download-archive) 


## Getting the code

We need to clone `mozilla/DeepSpeech` and `mozilla/tensorflow`.

```bash
git clone https://github.com/mozilla/DeepSpeech
```

```bash
git clone https://github.com/mozilla/tensorflow
```

## Configuring the paths

We need to create a symbolic link, for this example let's supose that we cloned into `D:\cloned` and now the structure looks like:

    .
    ├── D:\
    │   ├── cloned                 # Contains DeepSpeech and tensorflow side by side
    │   │   ├── DeepSpeech         # Root of the cloned DeepSpeech
    │   │   ├── tensorflow         # Root of the cloned Mozilla's tensorflow 
    └── ...

Change your path accordingly to your path structure, for the structure above we are going to use the following command:

```bash
mklink /d "D:\cloned\tensorflow\native_client" "D:\cloned\DeepSpeech\native_client"
```

## Adding environment variables

After you install the requirements there are few environment variables that we need to add.

#### MSYS2

For MSYS2 we need to add `C:\msys64\usr\bin` to the environment variables then we need to run:

```bash
pacman -Syu
```

```bash
pacman -Su
```

```bash
pacman -S patch unzip
```

#### BAZEL

For BAZEL we need to add the path to the executable, very important to rename the executable to `bazel`.

To check the version installed you can run:

```bash
bazel version
```

#### PYTHON

Add `python.exe` path to the environment variables.


#### CUDA

If you want to enable CUDA support you need to add the following paths to the environment variables.

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0
```

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\extras\CUPTI\libx64
```

### Building the code

There's one last command to run before building, you need to run the [configure.py](https://github.com/mozilla/tensorflow/blob/master/configure.py) inside `tensorflow` cloned directory.

Run:

```bash
python configure.py
```
The script will ask the configuration that you want to use, for AVX/AVX2 we will specify manually in the command so, we will skip this option by hitting `enter`. If you want to support CUDA please for this option select `y` when it asks for CUDA support.

Here's one example of the configuration:

```
Please input the desired Python library path to use.  Default is [D:\py\lib\site-packages]

Do you wish to build TensorFlow with Apache Ignite support? [Y/n]: n
No Apache Ignite support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: n
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: n
No CUDA support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is /arch:AVX]:


Would you like to override eigen strong inline for some C++ compilation to reduce the compilation time? [Y/n]: n
Not overriding eigen strong inline, some compilations could take more than 20 mins. 
```


### Running the build commands

At this point we are ready to start building the library, go to `tensorflow` directory that you cloned, following our examples should be `D:\cloned\tensorflow`.  

#### CPU
We will add AVX/AVX2 support in the command, you can remove the flags if you don't want to support them.

```bash
bazel build -c opt --copt=/arch:AVX --copt=/arch:AVX2 //native_client:libdeepspeech.so
```


#### GPU with CUDA
If you enabled CUDA in [configure.py](https://github.com/mozilla/tensorflow/blob/master/configure.py) configuration command now you can add `--config=cuda` to compile with CUDA support.

```bash
bazel build -c opt --config=cuda --copt=/arch:AVX --copt=/arch:AVX2 //native_client:libdeepspeech.so
```
