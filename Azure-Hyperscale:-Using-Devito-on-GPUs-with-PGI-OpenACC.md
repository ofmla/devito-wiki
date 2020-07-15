by Qie Zhang, Microsoft Azure Global (collaboration with the Devito team)

This instruction shows step by step how to set up the environment for running [Devito](https://github.com/devitocodes/devito) on an [Azure GPU VM](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-gpu). It is inherited and modified from [Using Devito on GPUs with PGI OpenACC](https://github.com/devitocodes/devito/wiki/Using-Devito-on-GPUs-with-PGI-OpenACC) at [Devito wiki](https://github.com/devitocodes/devito/wiki). Although ultimately we would like to automate the whole installation in this document, the [PGI compiler package](https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64) needs to be downloaded manually due to its license agreement. This wiki is followed by another wiki [GPU CPU environment setup and launching Jupyter notebook](https://github.com/devitocodes/devito/wiki/Azure:-GPU-CPU-environment-setup-and-launching-Jupyter-notebook) before you run the Devito RTM/FWI examples.

Following this wiki requires some basic knowledge of Azure, Linux and Python. After you go through these instructions targeting GPU, the environment should allow you to run Devito on both GPU and CPU, where the switch between GPU and CPU is enabled by a few [Devito environment variables](https://github.com/devitocodes/devito/wiki/FAQ#DEVITO_ARCH). However, if you are only interested in running Devito on an Azure CPU VM (as compared to a GPU VM), follow [this instruction](https://www.devitoproject.org/devito/download.html) which is much simpler. In both cases, keep in mind to pick a VM with a reasonable memory size for your Devito jobs.

### (1) Create an Azure Linux Virtual Machine (VM)

First, create a Virtual Machine ([VM](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/)) instead of Data Science Virtual Machine ([DSVM](https://docs.microsoft.com/en-us/azure/machine-learning/data-science-virtual-machine/)) on Azure in this case. DSVM has pre-installed packages such as Conda, python3 and Cuda which may cause confusion, so we stick to a bare Linux VM. 

In [Azure portal](https://azure.microsoft.com/en-us/features/azure-portal/), choose "Create a resource", then pick "Compute" from "Azure Marketplace". Next, choose "Virtual machine" and pick "Ubuntu Server 18.04 LTS" as the image. Pick "Review+create" to create the Linux VM.

We have validated that the PGI OpenACC compiler works on all Nvidia Tesla [K80, P100, V100, P40, M60](https://en.wikipedia.org/wiki/CUDA#GPUs_supported) GPUs. So for this exercise, suggest you pick one of the [NC6](https://docs.microsoft.com/en-us/azure/virtual-machines/nc-series), [NC6s_v2](https://docs.microsoft.com/en-us/azure/virtual-machines/ncv2-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json), [NC6s_v3](https://docs.microsoft.com/en-us/azure/virtual-machines/ncv3-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json), [ND6s](https://docs.microsoft.com/en-us/azure/virtual-machines/nd-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json), [NV6](https://docs.microsoft.com/en-us/azure/virtual-machines/nv-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) VMs which is equipped with one GPU. Alternatively, you can pick one of the NC24, NC24s_v2, NC24s_v3, ND24s, NV24 VMs where you can take advantage of the Devito MPI (domain decomposition) feature by using all 4 GPUs (per VM).

To use the open-source remote desktop software [X2GO](https://wiki.x2go.org/doku.php) to access the bare VM ("Ubuntu Server 18.04 LTS") with "xfce", you need to install the packages below. The commands can also run under [Azure serial console](https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/serial-console-overview). A third choice is to use `ssh` to access the VM by using, for example, [Windows PowerShell](https://docs.microsoft.com/en-us/powershell/), [MobaXterm](https://mobaxterm.mobatek.net/) or your terminal.
```
sudo apt-add-repository ppa:x2go/stable -y
sudo apt update -y
sudo apt-get install x2goserver x2goserver-xsession -y
sudo apt-get install xfce4 -y
```
X2GO setup can be found [here](https://docs.microsoft.com/en-us/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro#x2go), and now you can access the Azure VM using X2GO.

Then you can install [jupyter notebook](https://jupyter.org/) and [matplotlib](https://matplotlib.org/) that will be used for Devito demos. Note [python3](https://www.python.org/downloads/) is pre-installed on the VM.
```
sudo apt-get install python3-pip -y
sudo pip3 install jupyter
sudo apt-get install python3-matplotlib -y
```

### (2) Install build essentials (gcc, make, ...), libelf

```
sudo apt update -y
sudo apt install build-essential -y
sudo apt-get install libelf-dev libffi-dev -y
sudo apt install pkg-config -y
```

### (3) Download and install CUDA (the download link is for Ubuntu 18.04)

Since Cuda is not pre-installed, and PGI products do not contain Cuda drivers. You must download and install the appropriate Cuda driver from NVIDIA.
```
wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-1-local-10.1.105-418.39/7fa2af80.pub
sudo apt-get update -y
sudo apt-get install cuda -y
rm -rf cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
```
Then add the Cuda paths. Suggest including those in your ~/.bashrc file to avoid exporting them every time you wish to run on GPU.
```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```



### (4) Download and install the PGI compiler tools (community edition)

First, we need to install "firefox" for downloading the [PGI compiler file](https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64).
```
sudo apt-get install firefox -y
```
Next, go to the link below and download the version 19.10 of the PGI compiler which contains a 1 year free license. Click "Linux x86-64" to download the file "pgilinux-2019-1910-x86-64.tar.gz". Looks like there is not an option to directly `wget` or `curl` the file. But you can download and host the file on your Azure storage, then `wget` or `curl` the corresponding link. Utilizing `scp` is another option.
```
firefox https://www.pgroup.com/support/download_community.php?file=pgi-community-linux-x64
```
Install the PGI compiler. More info can be found at the [instruction](https://www.pgroup.com/resources/docs/19.10/x86/pgi-install-guide/index.htm#install-linux-steps).
```
tar xpfz pgilinux-2019-1910-x86-64.tar.gz
sudo ./install
```
There are many interactive questions during the installation, and you can follow the listed answers.
```
Press enter to continue...:                                       Enter
Do you accept these terms? (accept,decline):                      accept
Please choose install option:                                     1  Single system install
Installation directory? [/opt/pgi]:                               Enter
Do you wish to update/create links in the 2019 directory? (y/n):  y
Press enter to continue...:                                       Enter
Do you want to install Open MPI onto your system? (y/n):          y
Do you want to enable NVIDIA GPU support in Open MPI? (y/n):      y
Do you wish to obtain permanent license key or configure license service? (y/n): n
Do you want the files in the install directory to be read-only? (y/n):           n
```
Alternatively, you can use [expect script](https://en.wikipedia.org/wiki/Expect) to automate the PGI installation and avoid answering the questions interactively. Put the content below in a file "run_expect_pgi".
```
#!/usr/bin/expect -f

set timeout -1
spawn sudo ./install

expect -gl "Press enter to continue..."
send -- "\r"

expect {
    -gl "*--More--*" { send -- " "; exp_continue }
    -ex "Do you accept these terms? (accept,decline)"
}
send -- "accept\r"

expect -gl "Please choose install option"
send -- "1\r"

expect -gl "Installation directory?"
send -- "\r"

expect -gl "Do you wish to update/create links in the 2019 directory?"
send -- "y\r"

expect -gl "Press enter to continue..."
send -- "\r"

expect -gl "Do you want to install Open MPI onto your system?"
send -- "y\r"

expect -gl "Do you want to enable NVIDIA GPU support in Open MPI?"
send -- "y\r"

expect -gl "Do you wish to obtain permanent license key or configure license service?"
send -- "n\r"

expect -gl "Do you want the files in the install directory to be read-only?"
send -- "n\r"

expect eof
```
Then run the commands below to install "expect" and then execute the PGI installation where `./run_expect_pgi` runs `sudo ./install` automatically with provided interactions.
```
sudo apt install expect -y
tar xpfz pgilinux-2019-1910-x86-64.tar.gz
./run_expect_pgi
rm -rf documentation.html install install_components
```
After the installation is complete you should take care of the required paths. Again, it is suggested to add these exports to your ~/.bashrc file. Note: Official PGI compilers installation docs have openmpi instead of openmpi-3.1.3 in the path - this is wrong. You should use openmpi-3.1.3 otherwise your setup will not be able to see Open MPI installation.
```
export PGI=/opt/pgi
export PATH=/opt/pgi/linux86-64/19.10/bin:$PATH
export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/man
export LM_LICENSE_FILE=$LM_LICENSE_FILE:/opt/pgi/license.dat

export PATH=$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
export MANPATH=$MANPATH:$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/man
```
Typing `pgcc -V` you should now see the confirmation below. And typing `mpicc --version` you should see the same.
```
pgcc 19.10-0 LLVM 64-bit target on x86-64 Linux -tp haswell 
PGI Compilers and Tools
Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
```
Now Install "mpi4py" using PGI compilers (will take a few minutes). After completion, you should see the message "Successfully installed mpi4py-3.0.3".
```
env MPICC=/opt/pgi/linux86-64/19.10/mpi/openmpi-3.1.3/bin/mpicc CC=pgcc CFLAGS=-noswitcherror pip3 install --no-cache-dir mpi4py
```



### (5) Install nvtop (optional)

You can install `nvtop` (Nvidia top) for GPU monitoring. An alternative solution is to use the command `watch nvidia-smi -lms` for quick visual inspection of GPU utilization.
```
sudo apt install cmake libncurses5-dev -y
git clone https://github.com/Syllo/nvtop.git
mkdir -p nvtop/build && cd nvtop/build
cmake ..
make
sudo make install
cd ../..
```



### (6) Install Devito

Download and install Devito using "python3". We will avoid a conda environment because installing "mpi4py" with PGI inside such an environment is troublesome.
```
git clone https://github.com/devitocodes/devito.git
cd devito
pip3 install -e .
cd ..
```



### (7) Generate and execute a Devito operator on GPU

In order to generate OpenACC code using Devito we need to set a few environment flags to inform our compiler. To run Devito on GPU by using OpenACC compiler, set the [Devito environment variables](https://github.com/devitocodes/devito/wiki/FAQ#DEVITO_ARCH):
```
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LANGUAGE=openacc
```
If you would like to see more information regarding the compilation process as well as some performance metrics, you can also set:
```
export DEVITO_LOGGING=DEBUG
```
Then you can try the acoustic or elastic operator below:
```
python3 examples/seismic/acoustic/acoustic_example.py
python3 examples/seismic/elastic/elastic_example.py
```
If you wish to test a multi-GPU setup via MPI, try the example below. The number "-n 4" specifies 4 GPUs for MPI where you can use `nvtop` to monitor.
```
DEVITO_MPI=1 mpirun -n 4 python3 examples/seismic/elastic/elastic_example.py
```


### (8) Alternatively run Devito on CPU

The setup (1)-(7) works for a CPU VM too, or for the CPU within a GPU VM. To run Devito on CPU, set the [Devito environment variables](https://github.com/devitocodes/devito/wiki/FAQ#DEVITO_ARCH) as default (=no touch), which are:
```
export DEVITO_PLATFORM=cpu64
export DEVITO_ARCH=ccustom
export DEVITO_LANGUAGE=C
```
For a multi-CPU case where you like to use "OpenMP", replace the last line by:
```
export DEVITO_LANGUAGE=openmp
```
Then you can try the acoustic or elastic operator below:
```
python3 examples/seismic/acoustic/acoustic_example.py
python3 examples/seismic/elastic/elastic_example.py
```
