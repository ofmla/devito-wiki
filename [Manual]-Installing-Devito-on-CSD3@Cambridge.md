# Installing Devito on CSD3@CAMBRIDGE

```sh
# Do `ssh` to your login node (This example logs in to Intel® Xeon® Gold 6142 login node)
ssh user@login-knl.hpc.cam.ac.uk

# Anaconda not available on CSD3, so you have to load miniconda
module load miniconda3/4.5.1

# Load compilers (gcc, icc), tools..etc
module load gcc
module load intel/bundles/complib/2017.4

# Load git module
module load git # May have been deprecated or pre-installed

# Following instructions from https://github.com/opesci/devito
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .

# If everything went fine you should be able to run a typical operator. i.e.:
DEVITO_LOGGING=DEBUG DEVITO_ARCH=intel python examples/seismic/acoustic/acoustic_example.py
```


For an interactive session:
Use \#SBATCH -A T2-CSPP017-CPU or sintr with the same -A option with the Skylake partitions for an nteractive session.
