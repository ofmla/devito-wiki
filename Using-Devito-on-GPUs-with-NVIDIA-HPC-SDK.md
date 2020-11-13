
Test platform:
* Azure VM | Standard NC6_Promo (6 vcpus, 56 GiB memory)
 * Tesla K80, which has compute capability `3.7`. You can find out what card you have via `lshw -C display`; The compute capability of your card can be found [here](https://en.wikipedia.org/wiki/CUDA#GPUs_supported).
* Linux (ubuntu 18.04)

## 1. Install build essentials (gcc, make, ...), libelf.

```
sudo apt update
sudo apt install build-essential
sudo apt-get install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
```

## 2. Download and install NVIDIA HPC SDK

Follow the instructions below to install the NVIDIA HPC SDK. An NVIDIA CUDA driver must be installed on a system with a GPU before you can run a program compiled for the GPU on that system. PGI products do not contain CUDA Drivers. You must download and install the appropriate CUDA Driver from NVIDIA.

```
wget https://developer.download.nvidia.com/hpc-sdk/20.9/nvhpc_2020_209_Linux_x86_64_cuda_11.0.tar.gz
tar xpzf nvhpc_2020_209_Linux_x86_64_cuda_11.0.tar.gz
sudo nvhpc_2020_209_Linux_x86_64_cuda_11.0/install
```

Follow the instructions on screen to install NVIDIA HPC SDK.
After the installation is complete you should take care of the required paths:

```
export NVARCH=`uname -s`_`uname -m`; export NVARCH
export NVCOMPILERS=/opt/nvidia/hpc_sdk; export NVCOMPILERS
export MANPATH=$MANPATH:$NVCOMPILERS/$NVARCH/20.9/compilers/man; export MANPATH
export PATH=$NVCOMPILERS/$NVARCH/20.9/compilers/bin:$PATH; export PATH
```
We suggest adding those paths to your .bashrc to avoid needing to export them every time you wish to run on the GPU.

Typing `$ pgcc -V` you should now see:
```
pgcc (aka nvc) 20.9-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2020, NVIDIA CORPORATION.  All rights reserved.
```

To add the installed Open MPI binaries to the PATH also add:

```
export PATH=/opt/nvidia/hpc_sdk/Linux_x86_64/20.9/comm_libs/mpi/bin:$PATH
export MANPATH=/opt/nvidia/hpc_sdk/Linux_x86_64/20.9/comm_libs/mpi/man
```

We suggest also adding these exports to your .bashrc.

Typing `$ mpicc --version` you should now see:

```
nvc 20.9-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
NVIDIA Compilers and Tools
Copyright (c) 2020, NVIDIA CORPORATION.  All rights reserved.
```

## 4. Download and install Devito. We will install Devito using python3. We will avoid a conda env because installing mpi4py with PGI inside such environments is troublesome.
```
sudo apt-get install python3 python3-pip # Install python3 and pip3
git clone https://github.com/devitocodes/devito.git # Clone Devito
cd devito
pip3 install -e .
```

### 4b. Installing mpi4py using NVIDIA HPC SDK OpenMPI distro (Will take a few minutes)
```
env MPICC=/opt/nvidia/hpc_sdk/Linux_x86_64/20.9/comm_libs/mpi/bin/mpicc CC=pgcc CFLAGS=-noswitcherror pip3 install --no-cache-dir mpi4py
```

## 5. Generate and execute a Devito operator
In order to generate OpenACC code using Devito we need to set a few environment flags to inform our compiler.

We must set:
```
export DEVITO_PLATFORM=nvidiaX
export DEVITO_LANGUAGE=openacc
export DEVITO_ARCH=pgcc
```

If you would like to see more information regarding the compilation process as well as some performance metrics, you can also set:
```
export DEVITO_LOGGING=DEBUG
```

Let's try the elastic operator:
```
python3 examples/seismic/elastic/elastic_example.py
```

If you wish to test a multi-GPU setup via MPI type, e.g.,:
```
export DEVITO_MPI=1
DEVITO_MPI=1 mpirun -n 2 python3 examples/seismic/elastic/elastic_example.py
```
In order to check your installation is correct, ensure all tests located in
`DEVITO_ARCH=pgcc DEVITO_PLATFORM=nvidiaX DEVITO_LANGUAGE=openacc py.test tests/test_gpu_openacc.py`
pass. If you have not installed Open MPI then only the first 3 tests will run.

## Last step: Did it work?

You may use either `nvprof` or (for quick visual inspection) `nvtop`. 

To install `nvtop`, first install the prerequisites

```
sudo apt-get install libncurses5-dev
```

Then follow the instructions [here](https://github.com/Syllo/nvtop#nvtop-build).

Now rerun while keeping `nvtop` on in another terminal. You should see the GPU utilization spiking at 100% !

