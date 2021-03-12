# Installing Devito on CX2@IMPERIAL

```sh
# Do `ssh` to your login node (This example logs in to Intel(R) Xeon(R) CPU E5-2620 login node)
ssh user@login.hpc.ic.ac.uk

# Load Anaconda personal
module load anaconda3/personal

# Load intel compilers (gcc, icc), tools..etc
module load intel-suite
module load mpi

# Load git module
module load git

# Following instructions from https://github.com/opesci/devito
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .

# If everything went fine you should be able to run a typical operator. i.e.:
DEVITO_LOGGING=DEBUG DEVITO_ARCH=intel python examples/seismic/acoustic/acoustic_example.py
```

