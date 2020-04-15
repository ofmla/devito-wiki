
Test platform:
* Azure VM | Standard NC6_Promo (6 vcpus, 56 GiB memory)
 * Tesla K80, which has compute capability `3.7`. You can find out what card you got via `lshw -C display`; you can find out the compute capability of your card [here](https://en.wikipedia.org/wiki/CUDA#GPUs_supported).
* Linux (ubuntu 18.04)

## 1. Install build essentials (gcc, make, ...), libelf.

```
sudo apt update
sudo apt install build-essential
sudo apt-get install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
```

## 2. Download and install CUDA (recall: the download link is for an Ubuntu 18.04 machine)

```
Follow the instructions here to install CUDA.
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run
```

Follow the instructions on screen to install CUDA. Then:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```

## 2. Download and install PGI compiler tools community edition

We suggest downloading version 19.10 of PGI compilers which has a free license for 1 year.
Go to `https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64`
and download the .tar file.
There is probably no option to directly `wget` or `curl` the .tar.gz at your VM.
Our approach was to download and then `wget` or `curl` using a Dropbox, Google Drive or OneDrive link.

Follow the instructions from here:
'https://www.pgroup.com/resources/docs/19.10/x86/pgi-install-guide/index.htm#install-linux-pgi'
to install the PGI compilers.
```
1. tar xpfz <tarfile>.tar.gz
2. sudo ./install
```
Then: 
- Accept the terms
- Choose single system install (Option 1)
- Installation directory? [/opt/pgi] Press enter to accept, unless preferred otherwise.
Installation has now started.

After a couple of minutes you will see:
- `Do you wish to update/create links in the 2019 directory? (y/n)` *Answer yes*

The installation will continue with examples and PGI CUDA components.
After another couple of minutes you will be asked if you want to install Open MPI library.
One should install this library if aims to running Devito over multiple GPUs with MPI.
If so, press enter and then yes in order to install Open MPI onto your system.
Then you will be asked:
- `Do you want to enable NVIDIA GPU support in Open MPI? (y/n)` *Answer y(es)*
- `Do you wish to obtain permanent license key or configure license service? (y/n)` *Answer no*
- `Do you want the files in the install directory to be read-only? (y/n)` *Answer no*

After the installation is finished you should take care of the required paths:
```
$ export PGI=/opt/pgi;
$ export PATH=/opt/pgi/linux86-64/19.10/bin:$PATH;
$ export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/man;
$ export LM_LICENSE_FILE=$LM_LICENSE_FILE:/opt/pgi/license.dat; 
```

Try: `$ pgcc -V` and you should now see:
```
pgcc 19.10-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
```

If you have installed Open MPI library also add:

**Note:** Official PGI compilers installation docs have `openmpi` instead of `openmpi-3.1.3` in the path.
```
$ export PATH=$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
$ export MANPATH=$MANPATH:$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/man
$ export PATH=/opt/pgi/linux86-64/19.10/bin:opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
$ export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/man
```
Try: `$ mpicc --version` and you should now see:
```
pgcc 19.10-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
```

## 3. Download and install Devito. We will install Devito using python3
```
sudo apt-get install python3 python3-pip # Install python3 and pip3
git clone https://github.com/devitocodes/devito.git # Clone Devito
cd devito
pip3 install -e .
```

### 3b. Installing mpi4py using PGI compilers (Will take a few minutes)
```
env MPICC=/opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/bin/mpicc CC=pgcc CFLAGS=-noswitcherror pip3 install --no-cache-dir mpi4py
```


## 4. Generate and execute a Devito operator
In order to generate OpenACC code using Devito we need to set a few environment flags in order to inform our compiler.

We need to set:
```
export DEVITO_PLATFORM=nvidiaX
export DEVITO_LANGUAGE=openacc
export DEVITO_ARCH=pgcc
```

In case you need to see more info about the compilation process as well as some performance metrics, one can also set:
```
export DEVITO_LOGGING=DEBUG
```

Let's try the elastic operator:
```
python3 examples/seismic/elastic/elastic_example.py
```

## Last step: Did it work?

You may use either `nvprof` or (for quick visual inspection), `nvtop`. 

To install `nvtop`, first install the prerequisites

```
sudo apt-get install libncurses5-dev
```

Then follow the instructions [here](https://github.com/Syllo/nvtop#nvtop-build).

Now rerun while keeping `nvtop` on in another terminal. You should see the GPU utilization spiking at 100% !

