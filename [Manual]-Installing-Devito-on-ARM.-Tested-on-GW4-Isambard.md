
Start an interactive job:
`qsub -I -q arm-dev  -l select=1:ncpus=64 -l Walltime=03:00:00`
Then:
`
#Load python
module load cray-python/3.6.5.6
# Clone and install Devito
git clone https://github.com/opesci/devito.git
cd devito/
pip3 install --user -e .
#[Optional] Install Opescibench 
git clone https://github.com/opesci/opescibench
cd opescibench/
pip3 install --user -e .
# Set the following environment variables
export LC_ALL=C.UTF-8
export LANG=C.UTF-8
```

If you have also installed OpesciBench you can try the following test:
For a simple test:
`python3 benchmarks/user/benchmark.py test`
`Now going parallel with several optimizations and parameters set:
OMP_NUM_THREADS=32 DEVITO_PLATFORM=arm DEVITO_DEBUG_COMPILER=1 DEVITO_AUTOTUNING=aggressive DEVITO_BACKEND=core DEVITO_OPENMP=1 DEVITO_ARCH=gcc DEVITO_LOGGING=DEBUG aprun -n 1 -d 32 -cc numa_node python3 benchmarks/user/benchmark.py bench -bm O2 -P acoustic -so 4 -to 2 -d 512 512 512 --tn 1000 -x 

