## Test platform:
* RTX3090
* Linux (ubuntu 20.04)

## 1. Install prerequisites

build essentials (gcc, make, ...), cmake, libelf.

```
cd $HOME
sudo apt update
sudo apt install build-essential
sudo snap install cmake --classic
sudo apt install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
sudo apt install libnuma-dev
```

## 2. Nvidia HPC-SDK 20.11 bundled withe newest plus two previous CUDA versions
https://developer.nvidia.com/nvidia-hpc-sdk-2011-downloads

(recall: this is for an Ubuntu 20.10/20.04 machine)

```
wget https://developer.download.nvidia.com/hpc-sdk/20.11/nvhpc_2020_2011_Linux_x86_64_cuda_multi.tar.gz
tar xpfz nvhpc_2020_2011_Linux_x86_64_cuda_multi.tar.gz 
cd nvhpc_2020_2011_Linux_x86_64_cuda_multi/
./install
```
*The script will ask you the installation path. This tutorial uses* `$HOME/nvidia/hpc_sdk`

Export Nvidia HPC SDK vars:
```
NVARCH=`uname -s`_`uname -m`; #export NVARCH
NVCOMPILERS=$HOME/nvidia/hpc_sdk; #export NVCOMPILERS
VER='20.11'
export MANPATH=$MANPATH:$NVCOMPILERS/$NVARCH/$VER/compilers/man
export PATH=$NVCOMPILERS/$NVARCH/$VER/compilers/bin:$PATH
export LD_LIBRARY_PATH=$NVCOMPILERS/$NVARCH/$VER/compilers/lib:$LD_LIBRARY_PATH
export PATH=$NVCOMPILERS/$NVARCH/$VER/comm_libs/mpi/bin:$PATH
export MANPATH=$MANPATH:$NVCOMPILERS/$NVARCH/$VER/comm_libs/mpi/man
```

If everything went right, you should see something like

```
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2021 NVIDIA Corporation
Built on Sun_Feb_14_21:12:58_PST_2021
Cuda compilation tools, release 11.2, V11.2.152
Build cuda_11.2.r11.2/compiler.29618528_0
```

## 3. Install LLVM:
(Next CMake needs to generate Makefiles which will eventually be used for compilation:
Note: Installation breaks with gcc-10.
We use gcc-9, g++-9.)

```
cd $HOME; mkdir llvm; cd llvm
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
git checkout fee90542326bc1d81ba684bfc0a2cd21cb04e650
```

Let's then build the compiler

```
cd $HOME/llvm; mkdir build; cd build
cmake -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;lld;openmp" -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="X86;NVPTX" -DCMAKE_INSTALL_PREFIX=$HOME/llvm/13.0.0 -DLIBOMPTARGET_BUILD_NVPTX_BCLIB=ON -DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_86 -DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=35,37,50,52,60,61,70,75,80,86 -DCMAKE_C_COMPILER=gcc-9 -DCMAKE_CXX_COMPILER=g++-9 -DLLVM_ENABLE_BINDINGS=OFF -G "Unix Makefiles" $(pwd)/../llvm-project/llvm
make -j 6
make -j 6 install
```

Export clang vars:
```
VER='13.0.0'
export PATH=$HOME/llvm/$VER/bin:$PATH
export LD_LIBRARY_PATH=$HOME/llvm/$VER/lib:$LD_LIBRARY_PATH
```

If everything went smooth, you should see something like

```
$ clang --version
clang version 13.0.0 (https://github.com/llvm/llvm-project.git fee90542326bc1d81ba684bfc0a2cd21cb04e650)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /home/devito/llvm/13.0.0/bin
```

Let's now rebuild the OpenMP runtime libraries with Clang

```
cd $HOME/llvm ; mkdir build-openmp; cd build-openmp
```

And then:

```
cmake -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;lld;openmp" -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="X86;NVPTX" -DCMAKE_INSTALL_PREFIX=$HOME/llvm/13.0.0 -DLIBOMPTARGET_BUILD_NVPTX_BCLIB=ON -DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_86 -DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=35,37,50,52,60,61,70,75,80,86 -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DLLVM_ENABLE_BINDINGS=OFF -G "Unix Makefiles" $(pwd)/../llvm-project/openmp
```

And finally, we actually rebuild and reinstall the OpenMP runtime libraries:

```
make -j 6
make -j 6 install
```



## 4. Trying OpenMP 5.0 GPU offloading

Let's first install `nvtop` following the instructions [here](https://github.com/Syllo/nvtop#nvtop-build):

```
sudo apt install libncurses5-dev
cd ~
git clone https://github.com/Syllo/nvtop.git
mkdir -p nvtop/build && cd nvtop/build
cmake ..
make
sudo make install
cd $HOME
rm -rf nvtop
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

Let's save it as `omp-offloading.c` and compile. Fingers crossed.

```
clang -fopenmp -fopenmp-targets=nvptx64-nvidia-cuda -Xopenmp-target -march=sm_86 -Wall -O3 omp-offloading.c -o omp-offloading
``` 

No errors? Good! Some warnings about an old compute capability? That's OK too. The important thing is to see no errors at this point.

And now we run it while keeping `nvtop` on in another terminal:
```
./omp-offloading.o 10000000
```

You should see the GPU utilization spiking at 100% and something like that as the output
```
min = 0.000443, max = 1.839780, avg = 0.920177
```


## 5. Running a Devito's example

Create a virtual environment:
```
cd $HOME
mkdir environments
sudo apt-get install python3-venv
python3 -m venv environments/omp-offloading
source environments/omp-offloading/bin/activate
```

Clone and install Devito:
```
git clone https://github.com/devitocodes/devito.git
cd devito/
pip3 install -e .
```

Testing seismic examples:
```
DEVITO_ARCH=clang DEVITO_PLATFORM=nvidiaX DEVITO_LANGUAGE=openmp python examples/seismic/viscoacoustic/viscoacoustic_example.py
DEVITO_ARCH=clang DEVITO_PLATFORM=nvidiaX DEVITO_LANGUAGE=openmp python examples/seismic/acoustic/acoustic_example.py 
```



# UNDER TEST




## 6. Install a CUDA-aware MPI

Here we will OpenMPI according to the instructions [here](https://www.open-mpi.org/faq/?category=buildcuda).

OpenMPI recommends using UCX1.4 built with GDRcopy for the most updated set of MPI features and for better performance. Let's first install [GDRcopy](https://github.com/NVIDIA/gdrcopy)

```
cd $HOME
sudo apt install check libsubunit0 libsubunit-dev
git clone https://github.com/NVIDIA/gdrcopy.git
mv gdrcopy gdrcopy_src
mkdir gdrcopy
cd gdrcopy_src
make PREFIX=$HOME/gdrcopy CUDA=/usr/local/cuda-10.1 all install
sudo ./insmod.sh
cd $HOME
rm -rf gdrcopy_src
```

and include it to system's PATH

```
export PATH=$HOME/gdrcopy/bin:$PATH
export LD_LIBRARY_PATH=$HOME/gdrcopy/lib64:$LD_LIBRARY_PATH
```

You may want to check GDR copy installation by running the programs `sanity`, `copybw`, and `copylat` as shown [here](https://github.com/NVIDIA/gdrcopy).

Next, let's install UCX as shown [here](https://github.com/openucx/ucx/wiki):

```
cd $HOME
wget https://github.com/openucx/ucx/releases/download/v1.8.0/ucx-1.8.0.tar.gz
tar xzf ucx-1.8.0.tar.gz
cd ucx-1.8.0
./contrib/configure-release --prefix=$HOME/ucx --with-cuda=/usr/local/cuda-10.1 --with-gdrcopy=$HOME/gdrcopy
make -j 24 install
cd ..
rm -rf ucx-1.8.0 ucx-1.8.0.tar.gz
```

and then

```
export PATH=$HOME/ucx/bin:$PATH
export LD_LIBRARY_PATH=$HOME/ucx/lib:$LD_LIBRARY_PATH
```

If everything was ok, you should see something like this

```
$ ucx_info -v
# UCT version=1.8.0 revision c30b7da
# configured with: --disable-logging --disable-debug --disable-assertions --disable-params-check --prefix=/home/devito/ucx --with-cuda=/usr/local/cuda-10.1 --with-gdrcopy=/home/devito/gdrcopy
```

Finally, let's install OpenMPI

```
cd $HOME
wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.4.tar.gz
tar xzf openmpi-4.0.4.tar.gz
cd openmpi-4.0.4/
./configure --prefix=$HOME/openmpi --with-cuda=/usr/local/cuda-10.1 --with-ucx=$HOME/ucx
make -j 24 install
cd $HOME
rm -rf openmpi-4.0.4 openmpi-4.0.4.tar.gz
```

and then

```
export PATH=$HOME/openmpi/bin:$PATH
export LD_LIBRARY_PATH=$HOME/openmpi/lib:$LD_LIBRARY_PATH
```

If everything was ok, you should see something like this

```
$ mpirun --version
mpirun (Open MPI) 4.0.4

Report bugs to http://www.open-mpi.org/community/help/
```

## 7. Running a Devito's example on multiple GPUs

Install `pip3` and `mpi4py`

```
sudo apt install python3-pip
pip3 install mpi4py
```

Clone Devito repository, checkout at the right branch, and install Devito

```
git clone https://github.com/italoaug/devito.git
cd devito
pip3 install -e .
```

To use Devito on multiple GPU, your code must add `gpu-direct` option to the `Operator`. The `Operator` must look like

```
op = Operator(eq, opt=('advanced', {'gpu-direct': True}))
```

Here is a test example

```
from devito import Grid, TimeFunction, Eq, Operator
grid = Grid(shape=(4, 4))
u = TimeFunction(name='u', grid=grid, space_order=2, time_order=0)
u.data[:] = 1
eq = Eq(u.forward, u.dx+1)
op = Operator(eq, opt=('advanced', {'gpu-direct': True}))
op.apply(time_M=0)
```
Let's call it _timeMarching.py_. Finally, let's run a code

```
DEVITO_PLATFORM=nvidiaX DEVITO_ARCH=clang DEVITO_LANGUAGE=openmp DEVITO_MPI=1 OMPI_CC=clang mpirun -np 4 python3 timeMarching.py
```

## References

* Building LLVM/Clang with OpenMP Offloading to NVIDIA GPUs. _https://hpc-wiki.info/hpc/Building_LLVM/Clang_with_OpenMP_Offloading_to_NVIDIA_GPUs_
* OpenMP 5 GPU offloading instructions. _https://github.com/devitocodes/devito/wiki/OpenMP-5-GPU-offloading-instructions--%5BFabio%5D_