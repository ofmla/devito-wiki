by Qie Zhang, Microsoft Azure Global (collaboration with the Devito team)

This second wiki follows the first wiki [Using Devito on GPUs with PGI OpenACC](https://github.com/devitocodes/devito/wiki/Azure:-Using-Devito-on-GPUs-with-PGI-OpenACC) and precedes Devito jupyter notebook demos. It explains how to use Devito environment variables to pick GPU or CPU for running your Devito jobs. After that, we show how to launch jupyter notebook in your local browser for running the Devito demo in your Azure VM.

After installing the PGI OpenACC compiler and setting up the environment, a user can determine whether to use GPU or CPU for running a Devito job by an easy switch. But if you plan to use CPU only, there is no need to install the PGI OpenACC compiler. Depending on the number of GPUs or vCPUs on the VM, here is a list of choices for running a Devito example on one VM.

* (a) using one GPU
* (b) using multi-GPUs with MPI (requires turning e.g., a 2D FWI notebook into a python script)
* (c) using one vCPU
* (d) using multi-vCPUs with OpenMP

Several Devito environment variables are used to determine configurations for using GPU/CPU. They can be set in the shell or python notebook/script, as listed below. More info can be found from [Devito FAQ](https://github.com/devitocodes/devito/wiki/FAQ).

### Set Devito environment variables from the shell
(a) using one GPU
```
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LANGUAGE=openacc
```
(b) using multi-GPUs with MPI (requires turning e.g., a 2D FWI notebook into a python script "fwi.py")
```
export DEVITO_PLATFORM=nvidiaX
export DEVITO_ARCH=pgcc
export DEVITO_LANGUAGE=openacc
DEVITO_MPI=1 mpirun -n 4 python3 fwi.py   # Run this after setting environment variables
```
(c) using one vCPU
```
export DEVITO_PLATFORM=cpu64   # This is default so no need to run it
export DEVITO_ARCH=ccustom     # This is default so no need to run it
export DEVITO_LANGUAGE=C       # This is default so no need to run it
```
(d) using multi-vCPUs with OpenMP
```
export DEVITO_PLATFORM=cpu64
export DEVITO_ARCH=ccustom
export DEVITO_LANGUAGE=openmp
```
### Set Devito environment variables from the python notebook/script
```
from devito import configuration

#(a) using one GPU
#(b) using multi-GPUs with MPI (requires turning the notebook into a python script)
configuration['platform'] = 'nvidiaX'
configuration['compiler'] = 'pgcc'
configuration['language'] = 'openacc'

#(c) using one vCPU
#configuration['platform'] = 'cpu64'     # This is default so no need to run it
#configuration['compiler'] = 'ccustom'   # This is default so no need to run it
#configuration['language'] = 'C'         # This is default so no need to run it

#(d) using multi-vCPUs with OpenMP
#configuration['platform'] = 'cpu64'
#configuration['compiler'] = 'ccustom'
#configuration['language'] = 'openmp'
```


### Launching Jupyter notebook in a local browser

Although you can launch Jupyter notebook in a remote desktop such as [X2GO](https://wiki.x2go.org/doku.php) (simply by typing `jupyter notebook` in your terminal), it would be smoother to use Jupyter notebook in your local browser. We will show how to lauch Jupyter notebook in your local browser for running a Devito example in your Azure VM.

First, `ssh` your VM in your terminal. Replace "1234" by your VM port number, and replace "user id" and "host address" accordingly.
```
ssh -p 1234 user@host.eastus.cloudapp.azure.com
```
After `ssh`,  run the command in your terminal, replace "5678" by any port number you prefer.
```
jupyter notebook --no-browser --port=5678 --ip=$HOSTNAME
```
Then you will see some output like this:
```
http://127.0.0.1:5678/?token=8e8a2222a9a142555a1d8fd66d6634869def2c25ece611a5
```
Now in your local browser, enter the "host address" with the token from above, you will access Jupyter notebook locally.
```
http://host.eastus.cloudapp.azure.com:5678/?token=8e8a2222a9a142555a1d8fd66d6634869def2c25ece611a5
```

### Add paths before running Devito 
Also remember to add the paths below before you run a Devito Jupyter notebook. Suggest adding the Cuda and PGI compiler paths below to your ~/.bashrc to avoid exporting them every time you wish to run Devito on GPU (already explained in the first wiki [Using Devito on GPUs with PGI OpenACC](https://github.com/devitocodes/devito/wiki/Azure:-Using-Devito-on-GPUs-with-PGI-OpenACC)).
```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH

export PGI=/opt/pgi
export PATH=/opt/pgi/linux86-64/19.10/bin:$PATH
export MANPATH=$MANPATH:/opt/pgi/linux86-64/19.10/man
export LM_LICENSE_FILE=$LM_LICENSE_FILE:/opt/pgi/license.dat

export PATH=$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/bin:$PATH
export MANPATH=$MANPATH:$PGI/linux86-64/19.10/mpi/openmpi-3.1.3/man
```