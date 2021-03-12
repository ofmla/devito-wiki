The `make-pbs.py` program can be used to quickly generate PBS file to run (OpenMP, MPI, MPI+OpenMP) benchmarks. The program is located under `benchmarks/user`. For example, executing it as

`python make-pbs.py generate --load anaconda3/personal --load intel-suite --load mpi -nn 1 -nn 2 -nn 4 -nn 8 -ncpus 24 -mem 120 -np 2 -nt 12 -P acoustic --shape 600 600 600 -so 12 --tn 100 --arch haswell --mpi basic -r ~/opesci/results`

One obtains 4 files (one for each `nn` argument), which look like as below

```
#!/bin/bash

#PBS -lselect=2:ncpus=24:mem=120gb:mpiprocs=2:ompthreads=12
#PBS -lwalltime=02:00:00

module load anaconda3/personal
module load intel-suite
module load mpi

cd /home/devitocodes/devito

source activate devito

export DEVITO_HOME=/home/devitocodes/devito
export DEVITO_ARCH=intel
export DEVITO_LANGUAGE=openmp
export DEVITO_MPI=basic
export DEVITO_LOGGING=DEBUG

cd benchmarks/user

mpiexec python benchmark.py bench -P acoustic -bm O2 -d 600 600 600 -so 12 --tn 100 -x 1 --arch haswell -r /home/opesci/results/
```

The PBS file generated has been tested on Imperial College London Research Computing Service 
