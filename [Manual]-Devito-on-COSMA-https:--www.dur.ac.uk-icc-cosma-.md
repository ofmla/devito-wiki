# Installing Devito on COSMA@DURHAM

```sh
# After completing registration
# Do `ssh` to your login node (private key needed)
sh -l username login.cosma.dur.ac.uk

module unload python/2.7.15 
module load python/3.6.5 

module load intel_comp/2020-update2
module load intel_mpi/2020-update2

git clone 
pip3 install --user -e.[extras]

# If everything went fine you should be able to run a typical operator. i.e.:
DEVITO_LOGGING=DEBUG DEVITO_ARCH=intel python examples/seismic/acoustic/acoustic_example.py
```