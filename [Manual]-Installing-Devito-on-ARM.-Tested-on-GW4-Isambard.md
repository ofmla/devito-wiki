## Connect to Isambard login node
`ssh username@isambard.gw4.ac.uk`

## Connect to a login node
`ssh login-01`

## Connect to an xcil node
`ssh xcil00`

## Start an interactive job:
`qsub -I -q arm-dev  -l select=1:ncpus=64 -l Walltime=03:00:00`

## Check out the system specs [Optional]
`lscpu`
```
Architecture:        aarch64
Byte Order:          Little Endian
CPU(s):              256
On-line CPU(s) list: 0-255
Thread(s) per core:  4
Core(s) per socket:  32
Socket(s):           2
NUMA node(s):        2
Model:               2
BogoMIPS:            400.00
NUMA node0 CPU(s):   0-31,64-95,128-159,192-223
NUMA node1 CPU(s):   32-63,96-127,160-191,224-255
Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics cpuid asimdrdm
```

Then:


## Load python
`module load cray-python/3.6.5.6`
## Clone and install Devito
```
git clone https://github.com/devitocodes/devito.git
cd devito/
pip3 install --user -e .
```
## Set the following environment variables
```
export LC_ALL=C.UTF-8
export LANG=C.UTF-8
```

```
cd ../devito
export DEVITO_PLATFORM=arm
export DEVITO_LOGGING=DEBUG # optional, debug-level
export DEVITO_LANGUAGE=openmp # optional, add openmp-parallelism
```

`aprun python3 benchmarks/user/benchmark.py run -P acoustic`

Now going parallel with several optimizations and parameters set:

```
OMP_NUM_THREADS=32 DEVITO_PLATFORM=arm DEVITO_AUTOTUNING=aggressive DEVITO_LANGUAGE=openmp DEVITO_ARCH=gcc DEVITO_LOGGING=DEBUG aprun -n 1 -d 32 -cc numa_node python3 benchmarks/user/benchmark.py bench -bm O2 -P acoustic -so 4 -to 2 -d 512 512 512 --tn 1000 -x 1
```

# HACKATHON: ISAMBARD A64FX
## Connect to Isambard login node
`ssh username@isambard.gw4.ac.uk`

## Connect to a login node
`ssh login-01`

## Connect to an a64fx node
`ssh gw4a64fxlogin01`

`qsub -I`

Modules

By default, the Cray programming environment is loaded. A64FX-specific modules are exposed from /lustre/software/aarch64/modulefiles.
Clean all the loaded modules by using:
`module purge`

The Bristol HPC group also maintains a shared modules space where you may find additional useful tools, but keep in mind that these may not always be up-to-date. To use it:

```bash
module use /lustre/projects/bristol/modules-a64fx/modulefiles
```

## Clone Devito

```
git clone https://github.com/devitocodes/devito.git
cd devito/

module load gcc
module load python/3.8.8

pip3 install --user -e .

module load openmpi/4.0.4/gcc-11.0


source ~brx-pridley/arm-sve-tools/isambard-fujitsu.bashrc #add fujitsu
export CC=fcc
export CFLAGS="-Kfast,openmp -fPIC -Nfjomplib"
export CXX=FCC
export CXXFLAGS="-Kfast,openmp -fPIC -Nfjomplib"
export LDSHARED=fcc
export LDFLAGS="-Kfast,openmp -shared -Nfjomplib -lfjomphk -lfjomp -lfj90i -lfj90f -lfjsrcinfo -lfjcrt -lfjompcrt -lelf"
export MPICC=mpifcc
export MPICXX=mpiFCC

# install mpi4py with Fujitsu
module load python/3.8.8
pip3 install --force-reinstall --upgrade --user --no-cache-dir mpi4py
```

For MPI:

 I started a job with qsub -I and 2 cores then loaded the modules in the order given (gcc, python, openmpi) then ran pip3 install mpi4py --user then ran my mpi4py program using mpirun -np 3 python3 myprogram.py
```bash
module load gcc
module load python/3.8.8
module load openmpi/4.0.4/gcc-11.0
```

```bash
export DEVITO_PLATFORM=arm 
export DEVITO_LOGGING=DEBUG # optional, debug-level
export DEVITO_LANGUAGE=openmp # optional, add openmp-parallelism
OMP_PROC_BIND=close OMP_NUM_THREADS=48 python3 benchmarks/user/benchmark.py run -P acoustic -d 768 768 768  --tn 512

For the Fujitsu compiler:

source ~brx-pridley/arm-sve-tools/isambard-fujitsu.bashrc
export CC=fcc
export CFLAGS="-Kfast,openmp -fPIC -Nfjomplib"
export CXX=FCC
export CXXFLAGS="-Kfast,openmp -fPIC -Nfjomplib"
export LDSHARED=fcc
export LDFLAGS="-Kfast,openmp -shared -Nfjomplib -lfjomphk -lfjomp -lfj90i -lfj90f -lfjsrcinfo -lfjcrt -lfjompcrt -lelf"
export MPICC=mpifcc
export MPICXX=mpiFCC
```

```bash
[brx-gbisbas@c8n1 ~]$ numactl --hardware
available: 4 nodes (0-3)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11
node 0 size: 7768 MB
node 0 free: 6363 MB
node 1 cpus: 12 13 14 15 16 17 18 19 20 21 22 23
node 1 size: 8176 MB
node 1 free: 7365 MB
node 2 cpus: 24 25 26 27 28 29 30 31 32 33 34 35
node 2 size: 8176 MB
node 2 free: 6912 MB
node 3 cpus: 36 37 38 39 40 41 42 43 44 45 46 47
node 3 size: 8155 MB
node 3 free: 7668 MB
node distances:
node   0   1   2   3 
  0:  10  20  30  30 
  1:  20  10  30  30 
  2:  30  30  10  20 
  3:  30  30  20  10 

```

For htop on the compute node:
```bash
module use /lustre/projects/bristol/modules-arm/modulefiles
module load htop
htop
```
