### Connect to Isambard login node (https://gw4-isambard.github.io/docs/user-guide/jobs.html)
`ssh username@isambard.gw4.ac.uk`

### Connect to a login node
`ssh login-01`

### Start an interactive job on a P100 or V100:
```bash
qsub -I -q pascalq -l select=1:ncpus=16:ngpus=1
//or
qsub -I -q voltaq -l select=1:ncpus=16:ngpus=1

### Check out the GPU using lscpi [Optional]
lspci | grep 'NVIDIA'
06:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
81:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
```

Then:
### Activate conda so that python3.6 is loaded
```bash
source /opt/anaconda/bin/activate
python3 --version
Python 3.6.8 :: Anaconda, Inc.
```

### Clone and install Devito via Conda
```
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
```

### Load PGI compilers and CUDA toolkit
```
module load pgi/19.10 
module load cuda10.2/toolkit/10.2.89
```


### Set the following environment variables
```bash
export DEVITO_LANGUAGE=openacc
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LOGGING=DEBUG #optional
```

### You are ready to run a Devito operator on GPU
```bash
python3 examples/seismic/acoustic/acoustic_example.py  -d 256 256 256 --tn 128
```


## CPU
### Start an interactive job on a Cascade Lake:
`qsub -I -q clxq -l select=1:ncpus=40 -l Walltime=03:00:00`

### Start an interactive job on AMD ROME:
`qsub -I -q romeq -l select=1 -l Walltime=03:00:00`

### Check out the GPU using lscpu [Optional]
```bash
[brx-gbisbas@clx-002 pbs.35821.master.gw4.metoffice.gov.uk.x8z]$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                40
On-line CPU(s) list:   0-39
Thread(s) per core:    1
Core(s) per socket:    20
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 6230 CPU @ 2.10GHz
Stepping:              7
CPU MHz:               800.061
CPU max MHz:           3900.0000
CPU min MHz:           800.0000
BogoMIPS:              4200.00
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              28160K
NUMA node0 CPU(s):     0-19
NUMA node1 CPU(s):     20-39
```

Then:

### Load module files
```bash
module use /lustre/projects/bristol/modules/modulefiles
module load python/3.8.6
python3 --version
python3 -m venv clx-env
source clx-env/bin/activate
```

### Clone and install Devito BETTER ON A VIRTUAL ENV!
```bash
git clone https://github.com/devitocodes/devito.git
cd devito
pip3 install -e .
pip3 install matplotlib
module load gcc
// For MPI
module load openmpi/4.0.4/gcc-9.3
pip3 install mpi4py
```

## Set the following environment variables
```bash
export DEVITO_LANGUAGE=openmp
export DEVITO_ARCH=gcc
export DEVITO_LOGGING=DEBUG #optional
```

# You are ready to run a Devito operator
```bash
python3 examples/seismic/acoustic/acoustic_example.py  -d 256 256 256 --tn 128
```

