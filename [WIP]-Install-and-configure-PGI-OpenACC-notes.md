
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

## 2. Download and install PGI compiler tools community edition
```
https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64
Get the tar file at your VM via scp, wget or another way
Follow instructions from here:
https://www.pgroup.com/resources/docs/18.4/x86/pgi-install-guide/index.htm#install-linux-pgi

untar: tar xpfz pgi-linux......tar.gz
sudo ./install

Then: accept, single system install, /opt/pgi/ directory...
```

## 3. Download and install CUDA
```
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run

export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```
## 3. Download and install Devito
```
sudo apt-get install mpich libmpich-dev #optional
sudo apt-get install python3 python3-pip
git clone https://github.com/devitocodes/devito.git
cd devito
pip3 install -e .[extras] # extras needs mpi installed
```

## 4. Generate and execute an operator
```
`Let's try the acoustic operator`
DEVITO_LOGGING=DEBUG DEVITO_ARCH=pgcc python3 ~devito/examples/seismic/acoustic/acoustic_example.py 

* You can see in the logs that: Operator `Forward` fetched `/tmp/devito-jitcache-uid1000/a28bc362ac6aac0a21b203c041946208abd38f3c.c` in 0.43 s from jit-cache *
`Use an editor to add openacc directives`

- Add ```#include "openacc.h"``` at the headers section
- Add ```#pragma acc parallel loop``` in one of the loops (not the z one!)
- Use ```DEVITO_JIT_BACKDOOR=1 DEVITO_LOGGING=DEBUG DEVITO_ARCH=pgcc python3 examples/seismic/acoustic/acoustic_example.py```
```
You should see be executing the operator at the GPU.
```


## Last step: Did it work?

You may use either `nvprof` or (for quick visual inspection), `nvtop`. 

To install `nvtop`, first install the prerequisites

```
sudo apt-get install libncurses5-dev
```

Then follow the instructions [here](https://github.com/Syllo/nvtop#nvtop-build).

Now rerun while keeping `nvtop` on in another terminal. You should see the GPU utilization spiking at 100% !

