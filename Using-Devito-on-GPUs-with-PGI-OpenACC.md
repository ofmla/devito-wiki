
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

## 2. Download and install CUDA (recall: the download link is for an Ubuntu 18.04 machine)

Follow the instructions below to install the CUDA drivers. An NVIDIA CUDA driver must be installed on a system with a GPU before you can run a program compiled for the GPU on that system. PGI products do not contain CUDA Drivers. You must download and install the appropriate CUDA Driver from NVIDIA.

```
wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-1-local-10.1.105-418.39/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
```

Follow the instructions on screen to install CUDA. Then:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```
We suggest adding those paths to your .bashrc to avoid needing to export them every time you wish to run on the GPU.

## 3. Download and install the PGI compiler tools community edition

We suggest downloading version 19.10 of the PGI compiler which has contains a 1 year free license.
Go to `https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64`
and download the .tar file.
There is probably no option to directly `wget` or `curl` the .tar.gz to your VM.
Our approach was to download and then `wget` or `curl` using a Dropbox, Google Drive or OneDrive link. Utilizing `scp` is another option.

Follow the instructions from here:
'https://www.pgroup.com/resources/docs/19.10/x86/pgi-install-guide/index.htm#install-linux-pgi'
to install the PGI compilers.
```
1. tar xpfz <tarfile>.tar.gz
2. sudo ./install
```
Then: 
- Accept the terms
- Choose `single system install` (Option 1)
- `Installation directory? [/opt/pgi]` Press enter to accept unless another directory is preferable.
The installation will now begin.

After a couple of minutes you will see:
- `Do you wish to update/create links in the 2019 directory? (y/n)` *Answer yes*

The installation will continue with examples and PGI CUDA components.
After another couple of minutes you will be asked if you wish to install the Open MPI library.
MPI is required to parallelise solves using domain-decompositions methods.

If you wish to utilize this feature, press `enter` and then type `yes` in order to install Open MPI onto your system.
Then you will be asked:
- `Do you want to enable NVIDIA GPU support in Open MPI? (y/n)` *Answer y(es)*
- `Do you wish to obtain permanent license key or configure license service? (y/n)` *Answer n(o)*
- `Do you want the files in the install directory to be read-only? (y/n)` *Answer n(o)*

After the installation is complete you should take care of the required paths:
```
$ export PGI=/opt/pgi;
$ export PATH=/opt/pgi/linux86-64/19.10/bin:$PATH;
$ export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/man;
$ export LM_LICENSE_FILE=$LM_LICENSE_FILE:/opt/pgi/license.dat; 
```
Again, it is suggested to add these exports to .bashrc.

Typing `$ pgcc -V` you should now see:
```
pgcc 19.10-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
```

If you have installed the Open MPI library also add:

**Note:** Official PGI compilers installation docs have `openmpi` instead of `openmpi-3.1.3` in the path. This is wrong. You should use openmpi-3.1.3 otherwise your setup will not be able to see Open MPI installation.
```
$ export PATH=$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
$ export MANPATH=$MANPATH:$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/man
$ export PATH=/opt/pgi/linux86-64/19.10/bin:opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
$ export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/man
```
We suggest also adding these exports to your .bashrc.

Typing `$ mpicc --version` you should now see:
```
pgcc 19.10-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
```

## 4. Download and install Devito. We will install Devito using python3. We will avoid a conda env because installing mpi4py with PGI inside such environments is troublesome.
```
sudo apt-get install python3 python3-pip # Install python3 and pip3
git clone https://github.com/devitocodes/devito.git # Clone Devito
cd devito
pip3 install -e .
```

### 4b. Installing mpi4py using PGI compilers (Will take a few minutes)
```
env MPICC=/opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/bin/mpicc CC=pgcc CFLAGS=-noswitcherror pip3 install --no-cache-dir mpi4py
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

