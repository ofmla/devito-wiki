_Loosely readapted from https://hpc-wiki.info/hpc/Building_LLVM/Clang_with_OpenMP_Offloading_to_NVIDIA_GPUs_

Test platform:
* Azure VM | Standard NC6_Promo (6 vcpus, 56 GiB memory)
 * Tesla K80, which has compute capability `3.7`. You can find out what card you got via `lshw -C display`; you can find out the compute capability of your card [here](https://en.wikipedia.org/wiki/CUDA#GPUs_supported).
* Linux (ubuntu 18.04)

## 1. Install build essentials (gcc, make, ...), cmake, libelf.

```
sudo apt update
sudo apt install build-essential
sudo snap install cmake --classic
sudo apt-get install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
```

## 2. Download and install CUDA (recall: this is for an Ubuntu 18.04 machine)

**Note: we specifically need CUDA 10.1 -- the newest version, at the moment of writing 10.2, won't work with clang-10 !**

```
sudo dpkg -i cuda-repo-ubuntu1804_10.1.105-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
```

Then:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```

## 3. Get the source code for the required LLVM packages

```
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.0/llvm-10.0.0.src.tar.xz
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.0/clang-10.0.0.src.tar.xz
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.0/openmp-10.0.0.src.tar.xz
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.0/compiler-rt-10.0.0.src.tar.xz
```

Which we have to untar

```
tar xf llvm-10.0.0.src.tar.xz
tar xf clang-10.0.0.src.tar.xz
tar xf openmp-10.0.0.src.tar.xz
tar xf compiler-rt-10.0.0.src.tar.xz
```

This leaves you with 4 directories named `llvm-10.0.0.src`, `clang-10.0.0.src`, `openmp-10.0.0.src`, and `compiler-rt-10.0.0.src`. All these components can be built together if the directories are correctly nested:

```
mv clang-10.0.0.src llvm-10.0.0.src/tools/clang
mv openmp-10.0.0.src llvm-10.0.0.src/projects/openmp
mv compiler-rt-10.0.0.src llvm-10.0.0.src/projects/compiler-rt
```

## 4. Build the compiler

```
mkdir build
cd build
```

Next CMake needs to generate Makefiles which will eventually be used for compilation:

```
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$(pwd)/../install \
	-DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_37 \
	-DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=37,60,70 ../llvm-10.0.0.src
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

Once finished, we have to install it

```
make -j6 install
```

## 5. Rebuild the OpenMP Runtime Libraries with Clang

If you tried to compile an application with OpenMP offloading right now, Clang would print the following message:

```
clang-10: warning: No library 'libomptarget-nvptx-sm_37.bc' found in the default clang lib directory or in LIBRARY_PATH. Expect degraded performance due to no inlining of runtime functions on target devices. [-Wopenmp-target]
```

To make it go away -- which we do want, for maximum performance -- we need to recompile the OpenMP runtime libraries with Clang. To do so, we first create a new build directory:

```
cd ..
mkdir build-openmp
cd build-openmp
```

And then:

```
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$(pwd)/../install \
	-DCMAKE_C_COMPILER=$(pwd)/../install/bin/clang \
	-DCMAKE_CXX_COMPILER=$(pwd)/../install/bin/clang++ \
	-DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=37,60,70 \
	../llvm-10.0.0.src/projects/openmp
```

And finally, we actually rebuild and reinstall the OpenMP runtime libraries:

```
make -j6
make -j6 install
```

Now we should be good to go.

## 6. Try it!

First, let's put Clang in PATH

```
export PATH=~/install/bin:$PATH
export LD_LIBRARY_PATH=~/install/lib:$LD_LIBRARY_PATH
```

If everything went smooth, you should see something like

```
devito@devito-fl:~$ clang --version
clang version 10.0.0 (tags/RELEASE_900/final)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /home/devito/install/bin
```

Now let's test offloading. We are gonna use the following toy app:

```
#include <malloc.h>
#include <stdio.h>
#include <stdlib.h>
 
int main(int argc, char* argv[])
{
     
    int n = atoi(argv[1]);
     
    double* x = (double*)malloc(sizeof(double) * n);
    double* y = (double*)malloc(sizeof(double) * n);
 
    double idrandmax = 1.0 / RAND_MAX;
    double a = idrandmax * rand();
    for (int i = 0; i < n; i++)
    {
        x[i] = idrandmax * rand();
        y[i] = idrandmax * rand();
    }
 
    #pragma omp target data map(tofrom: x[0:n],y[0:n])
    {
        #pragma omp target
        #pragma omp for
        for (int i = 0; i < n; i++)
            y[i] += a * x[i];
    }
     
    double avg = 0.0, min = y[0], max = y[0];
    for (int i = 0; i < n; i++)
    {
        avg += y[i];
        if (y[i] > max) max = y[i];
        if (y[i] < min) min = y[i];
    }
     
    printf("min = %f, max = %f, avg = %f\n", min, max, avg / n);
     
    free(x);
    free(y);
 
    return 0;
}
```

Let's save it in `~/examples/omp-offloading.c`.

Before proceeding, checkout your PATH and LD_LIBRARY_PATH. Mine are as follows:

```
devito@devito-fl:~$ echo $LD_LIBRARY_PATH
/home/devito/install/lib:/usr/local/cuda-10.1/lib64:
devito@devito-fl:~$ echo $PATH
/home/devito/install/bin:/usr/local/cuda-10.1/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

In particular, `clang` as well as the CUDA libraries must be available.

Now we compile `omp-offloading.c`. Fingers crossed.

```
cd examples
clang -fopenmp -fopenmp-targets=nvptx64-nvidia-cuda -Xopenmp-target -march=sm_37 -Wall -O3 omp-offloading.c -o omp-offloading
``` 

No errors? Good! Some warnings about an old compute capability? That's OK too. The important thing is to see no errors at this point.

And now we run it...

```
./omp-offloading 10000000
```

## 7. Did it work?

You may use either `nvprof` or (for quick visual inspection), `nvtop`. 

To install `nvtop`, first install the prerequisites

```
sudo apt-get install libncurses5-dev
```

Then follow the instructions [here](https://github.com/Syllo/nvtop#nvtop-build).

Now rerun the example while keeping `nvtop` on in another terminal. You should see the GPU utilization spiking at 100% !

## 8. OpenMP 5 GPU offloading in Devito
To use OpenMP 5 GPU offloading in Devito the following environment variables should be set:
```
export DEVITO_ARCH=clang
export DEVITO_PLATFORM=nvidiaX
export DEVITO_LANGUAGE=openmp
```
Alternatively, one can instruct the Devito compiler to generate code for GPUs by passing the corresponding arguments (compiler, platform, language) to an Operator as seen in [this GPU example notebook](https://github.com/devitocodes/devito/blob/master/examples/gpu/01_diffusion_with_openmp_offloading.ipynb).