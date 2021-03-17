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

