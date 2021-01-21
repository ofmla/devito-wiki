# Installing Devito on COSMA@DURHAM

```sh
# After completing registration
# Do `ssh` to your login node (private key needed)
sh -l username login.cosma.dur.ac.uk

module unload python/2.7.15 
module load python/3.6.5 

module load gnu_comp/10.2.0 
module load openmpi/4.0.5

git clone 
pip3 install --user -e.[extras]

# If everything went fine you should be able to run a typical operator. i.e.:
DEVITO_LOGGING=DEBUG DEVITO_ARCH=intel python examples/seismic/acoustic/acoustic_example.py
```

Other useful modules
```
module load git
```



For CS-workshop, DURHAM,a typical slurm job file:
```
#!/bin/bash -l

### Job name
#SBATCH --job-name=HelloHybrid

### 2 compute nodes
#SBATCH --ntasks 64

#SBATCH --hint=nomultithread
#SBATCH -J devito_test #Give it something meaningful.
#SBATCH -o %J.stdout.out
#SBATCH -e %J.stderr.err
#SBATCH -p bluefield1  #a some other partition, e.g. cosma, cosma6, etc.
#SBATCH -A do008 #e.g. dp004
#SBATCH -t 02:00:00
#SBATCH --mail-user=mailto:g.bisbas18@imperial.ac.uk #PLEASE PUT YOUR EMAIL ADDRESS HERE (without the <>)
#SBATCH --exclusive

#load the modules used to build your program.
module unload python/2.7.15 
module load python/3.6.5
module load gnu_comp/10.2.0 
module load openmpi/4.0.5

which mpicc
which mpirun

# Run the program
export DEVITO_LOGGING=DEBUG
export DEVITO_ARCH=gcc
export DEVITO_MPI=1
export DEVITO_PROFILING=advanced1

lscpu
I_MPI_DOMAIN=socket I_MPI_DEBUG=2 OMP_PROC_BIND=close DEVITO_LANGUAGE=openmp OMP_NUM_THREADS=32 mpirun -np 2 python3 examples/seismic/acoustic/acoustic_example.py -d 600 600 600 --tn 512 -so 8
```