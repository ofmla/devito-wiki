1. **NOTE: Haven't tried with MPI yet**
2. **NOTE: Haven't tried AMD GPUs yet**

The conda-based installation procedure worked like a charm.

Testbed hardware:

```
Model name:          AMD EPYC 7V12 64-Core Processor
```

Testbed OS:

```
Ubuntu 18.04.4 LTS (Bionic Beaver)
```

on a fresh install.

* install compilers

```
sudo apt-get install -y gcc g++
```

* install miniconda

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3....sh
<follow instructions>
```

* install Devito in development mode

```
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
```


Now let's try to install AMD's version of LLVM for OpenMP 5 offloading. The compiler is named `AOMP` (while the classic AMD compiler for CPU is AOCC). The steps below are partly taken from [here](https://github.com/ROCm-Developer-Tools/aomp/blob/master/docs/UBUNTUINSTALL.md)

```
wget https://github.com/ROCm-Developer-Tools/aomp/releases/download/rel_11.5-0/aomp_Ubuntu1804_11.5-0_amd64.deb
sudo dpkg -i aomp_Ubuntu1804_11.5-0_amd64.deb
echo 'SUBSYSTEM=="kfd", KERNEL=="kfd", TAG+="uaccess", GROUP="video"' | sudo tee /etc/udev/rules.d/70-kfd.rules
wget -qO - http://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
sudo apt update
sudo apt install rock-dkms -y
```

and finally (fingers crossed):

```
sudo reboot
```

and once back:

```
sudo usermod -a -G video $USER
```