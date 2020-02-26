
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
transfer via scp?
Follow instructions from here:
https://www.pgroup.com/resources/docs/18.4/x86/pgi-install-guide/index.htm#install-linux-pgi
```

## 3. Download and install CUDA
```
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run

export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```