_Loosely readapted from:_
* _https://hpc-wiki.info/hpc/Building_LLVM/Clang_with_OpenMP_Offloading_to_NVIDIA_GPUs_
* _https://github.com/devitocodes/devito/wiki/OpenMP-5-GPU-offloading-instructions--%5BFabio%5D_

Test platform:
* Azure VM | Standard NC6_Promo (24 vcpus, 220 GiB memory)
* 4 x Tesla K80, which has compute capability `3.7`. You can find out what card you got via `lshw -C display`; you can find out the compute capability of your card [here](https://en.wikipedia.org/wiki/CUDA#GPUs_supported).
* Linux (ubuntu 18.04)

## 1. Install build essentials (gcc, make, ...), cmake, libelf.

```
sudo apt update
sudo apt install build-essential
sudo snap install cmake --classic
sudo apt-get install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
```

## 2. Download and install CUDA (recall: the download link is for an Ubuntu 18.04 machine)

```
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run
```

Follow the instructions on the screen to install CUDA. Then add the following lines to the end of the file `~/.bashrc`:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```

After that, update the environment variables through the command:

```
source ~/.bashrc
```

## 3. Clone LLVM git source:

```
git clone https://github.com/llvm/llvm-project.git
```

## 4. Build the compiler

```
mkdir build
cd build
```

Next CMake needs to generate Makefiles which will eventually be used for compilation:

```
cmake                                                                          \
  -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;lld;openmp" \
  -DCMAKE_BUILD_TYPE=Release                                                   \
  -DLLVM_TARGETS_TO_BUILD="X86;NVPTX"                                          \
  -DCMAKE_INSTALL_PREFIX=$(pwd)/../llvm                                        \
  -DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_37                                      \
  -DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=35,37,50,52,60,61,70,75            \
  -DCMAKE_C_COMPILER=gcc                                                       \
  -DCMAKE_CXX_COMPILER=g++                                                     \
  -G "Unix Makefiles" ../llvm-project/llvm
```

Now it's finally time to actually compile

```
make -j 24
```

Note: here I'm using `-j 24` because my system has 24 physical cores.
This is gonna take a while...

Once finished, we have to install it

```
make -j 24 install
```

## 5. Rebuild the OpenMP runtime libraries with Clang

```
cd ..
mkdir build-openmp
cd build-openmp
```

And then:

```
cmake                                                                          \
  -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;lld;openmp" \
  -DCMAKE_BUILD_TYPE=Release                                                   \
  -DLLVM_TARGETS_TO_BUILD="X86;NVPTX"                                          \
  -DCMAKE_INSTALL_PREFIX=$(pwd)/../llvm                                        \
  -DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_37                                      \
  -DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=35,37,50,52,60,61,70,75            \
  -DCMAKE_C_COMPILER=clang                                                      \
  -DCMAKE_CXX_COMPILER=clang++                                                  \
  -G "Unix Makefiles" ../llvm-project/llvm
```

And finally, we actually rebuild and reinstall the OpenMP runtime libraries:

```
make -j 24
make -j 24 install
```

Now we should be good to go.

## 6. Try it!

If everything went smooth, you should see something like

```
$ clang --version
clang version 11.0.0 (https://github.com/llvm/llvm-project.git ff66919020fab730c87e1b3ddf418e50ffcb7819)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /home/devito/llvm/bin
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
$ echo $LD_LIBRARY_PATH
/home/devito/llvm/lib:/usr/local/cuda-10.1/lib64:
$ echo $PATH
/home/devito/llvm/bin:/usr/local/cuda-10.1/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
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