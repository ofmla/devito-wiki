**NOTE: Haven't tried with MPI yet**

The conda-based installation procedure worked like a charm.

Testbed hardware:

```
Model name:          AMD EPYC 7V12 64-Core Processor
```

Testbed OS:

```
Ubuntu 18.04.4 LTS (Bionic Beaver)
```

on a fresh install.

* install compilers

```
sudo apt-get install -y gcc g++
```

* install miniconda

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3....sh
<follow instructions>
```

* install Devito in development mode

```
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
```