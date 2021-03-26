## Connect to Isambard login node
`ssh username@isambard.gw4.ac.uk`

## Connect to a login node
`ssh login-01`

https://gw4-isambard.github.io/docs/user-guide/jobs.html

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
## Start an interactive job on a Cascade Lake:
`qsub -I -q clxq -l select=1:ncpus=40 -l Walltime=03:00:00`

## Start an interactive job on AMD ROME:
`qsub -I -q romeq -l select=1 -l Walltime=03:00:00`

## Check out the GPU using lscpu [Optional]
`lscpu'`
```
[brx-gbisbas@clx-002 pbs.35821.master.gw4.metoffice.gov.uk.x8z]$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                40
On-line CPU(s) list:   0-39
Thread(s) per core:    1
Core(s) per socket:    20
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 6230 CPU @ 2.10GHz
Stepping:              7
CPU MHz:               800.061
CPU max MHz:           3900.0000
CPU min MHz:           800.0000
BogoMIPS:              4200.00
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              28160K
NUMA node0 CPU(s):     0-19
NUMA node1 CPU(s):     20-39
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb cat_l3 cdp_l3 invpcid_single intel_pt ssbd mba ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm mpx rdt_a avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts hwp hwp_act_window hwp_epp hwp_pkg_req pku ospke avx512_vnni md_clear spec_ctrl intel_stibp flush_l1d arch_capabilities
```

Then:

## Load module files
```bash
module use /lustre/projects/bristol/modules/modulefiles
module load python/3.8.6
python3 --version
python3 -m venv clx-env
source clx-env/bin/activate
```

## Clone and install Devito BETTER ON A VIRTUAL ENV!
```
git clone https://github.com/devitocodes/devito.git
cd devito
pip3 install -e .
pip3 install matplotlib
module load gcc
// For MPI
module load openmpi/4.0.4/gcc-9.3
pip3 install mpi4py
```

## Set the following environment variables
```
export DEVITO_LANGUAGE=openmp
export DEVITO_ARCH=gcc
export DEVITO_LOGGING=DEBUG #optional
```

# You are ready to run a Devito operator
```
python3 examples/seismic/acoustic/acoustic_example.py  -d 256 256 256 --tn 128
```

