## Connect to Isambard login node
`ssh username@isambard.gw4.ac.uk`

## Connect to a login node
`ssh login-01`

# GPU
## Start an interactive job on a P100 or V100:
`qsub -I -q pascalq -l select=1:ncpus=16:ngpus=1`
or
`qsub -I -q voltaq -l select=1:ncpus=16:ngpus=1`

## Check out the GPU using lscpi [Optional]
`lspci | grep 'NVIDIA'`
```
06:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
81:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
```

Then:

## Activate conda so that python3.6 is loaded
`source /opt/anaconda/bin/activate`

`python3 --version`

`Python 3.6.8 :: Anaconda, Inc.`

## Clone and install Devito via Conda
```
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
```

## Load PGI compilers and CUDA toolkit
```
module load pgi/19.10 
module load cuda10.2/toolkit/10.2.89
```


## Set the following environment variables
```
export DEVITO_LANGUAGE=openacc
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LOGGING=DEBUG #optional
```

# You are ready to run a Devito operator on GPU
```
python3 examples/seismic/acoustic/acoustic_example.py  -d 256 256 256 --tn 128
```


# CPU
## Start an interactive job on a P100 or V100:
`qsub -I -q pascalq -l select=1:ncpus=16:ngpus=1`
or
`qsub -I -q voltaq -l select=1:ncpus=16:ngpus=1`

## Check out the GPU using lscpi [Optional]
`lspci | grep 'NVIDIA'`
```
06:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
81:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
```

Then:

## Activate conda so that python3.6 is loaded
`source /opt/anaconda/bin/activate`

`python3 --version`

`Python 3.6.8 :: Anaconda, Inc.`

## Clone and install Devito via Conda
```
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
```

## Load PGI compilers and CUDA toolkit
```
module load pgi/19.10 
module load cuda10.2/toolkit/10.2.89
```


## Set the following environment variables
```
export DEVITO_LANGUAGE=openacc
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LOGGING=DEBUG #optional
```

# You are ready to run a Devito operator on GPU
```
python3 examples/seismic/acoustic/acoustic_example.py  -d 256 256 256 --tn 128
```

