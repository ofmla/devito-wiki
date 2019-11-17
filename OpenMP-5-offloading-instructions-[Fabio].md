_Loosely readapted from https://hpc-wiki.info/hpc/Building_LLVM/Clang_with_OpenMP_Offloading_to_NVIDIA_GPUs_

Assuming:

* a pristine machine
* Ubuntu 18.04 as OS
* a Tesla K80 available, which has compute capability `3.7`. You can find out what card you got via `lshw -C display`; you can find out the compute capability of your card [here](https://en.wikipedia.org/wiki/CUDA#GPUs_supported).

1. Install build essentials (gcc, make, ...), cmake, libelf.

```
sudo apt update
sudo apt install build-essential
sudo snap install cmake --classic
sudo apt-get install -y libelf-dev
```

2. Download and install CUDA (recall: the download link is for an Ubuntu 18.04 machine)

```
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run
```

Follow the instructions on screen to install CUDA. Then:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```

3. Get the source code for the required LLVM packages

```
wget https://releases.llvm.org/9.0.0/llvm-9.0.0.src.tar.xz
wget https://releases.llvm.org/9.0.0/cfe-9.0.0.src.tar.xz
wget https://releases.llvm.org/9.0.0/openmp-9.0.0.src.tar.xz
wget https://releases.llvm.org/9.0.0/compiler-rt-9.0.0.src.tar.xz
```

Which we have to untar

```
tar xf llvm-9.0.0.src.tar.xz
tar xf cfe-9.0.0.src.tar.xz
tar xf openmp-9.0.0.src.tar.xz
tar xf compiler-rt-9.0.0.src.tar.xz
```

This leaves you with 4 directories named `llvm-9.0.0.src`, `cfe-9.0.0.src`, `openmp-9.0.0.src`, and `compiler-rt-9.0.0.src`. All these components can be built together if the directories are correctly nested:

```
mv cfe-9.0.0.src llvm-9.0.0.src/tools/clang
mv openmp-9.0.0.src llvm-9.0.0.src/projects/openmp
mv compiler-rt-9.0.0.src llvm-9.0.0.src/projects/compiler-rt
```

4. Build the compiler

```
mkdir build
cd build
```

Next CMake needs to generate Makefiles which will eventually be used for compilation:

```
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$(pwd)/../install \
	-DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_37 \
	-DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=37,60,70 ../llvm-9.0.0.src
```

If all went right, you should see these lines:

```
...
-- Found LIBOMPTARGET_DEP_CUDA_DRIVER: /usr/lib/x86_64-linux-gnu/libcuda.so
-- LIBOMPTARGET: Building offloading runtime library libomptarget.
...
-- LIBOMPTARGET: Building CUDA offloading plugin.
...
```

Now it's finally time to actually compile

```
make -j6
```

Note: here I'm using `-j6` because my system has 6 physical cores.
This is gonna take a while...

