\# ssh to your login node
ssh user@login-knl.hpc.cam.ac.uk

\# anaconda not available, load miniconda
module load miniconda3/4.5.1

\# Load compilers, tools..etc
module load gcc
module load intel/bundles/complib/2017.4

\# Load git module
module load git

\# Following instructions from https://github.com/opesci/devito
git clone https://github.com/opesci/devito.git
cd devito
conda env create -f environment.yml
source activate devito
pip install -e .