Assuming:

* a pristine machine
* Ubuntu 18.04 as OS
* an NVidia card available

1. Install gcc, make, ...

```
sudo apt update
sudo apt install build-essential
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

3. 


