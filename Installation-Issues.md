# Linux
None known

# OSX
Most OSX issues we've seen stem from compiler issues. Please make sure you have a functioning C/C++ compiler before trying to install/use Devito. 

# Windows
We haven't tested devito on Windows but you're welcome to try. We suggest one of the following approaches:
## Windows Subsystem for Linux
* Enable the windows subsystem for Linux following the instructions [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* Get a working bash shell on ubuntu (through WSL)
* Install [conda](https://conda.io/docs/user-guide/install/linux.html) and [gcc](https://askubuntu.com/a/154406)
* Proceed with the devito conda-based install instructions

## Docker
* Install Docker using the instructions [here](https://docs.docker.com/v17.09/docker-for-windows/install/)
* Ensure that `git config --global core.autocrlf false` is set to prevent modification of line endings
* Proceed with the devito docker-based install instructions

## Virtual Machine
* Follow the instructions [here](https://www.lifewire.com/install-ubuntu-linux-windows-10-steps-2202108) to install Ubuntu inside a virtual machine on windows. 
* Install [conda](https://conda.io/docs/user-guide/install/linux.html) and [gcc](https://askubuntu.com/a/154406)
* Proceed with the devito conda-based install instructions
