In this article, you will learn how to install and set up the Intel oneAPI suite to be able to use profiling scripts available in Devito to discover your application's performance. The main tools of interest that are present in Intel oneAPI are Intel Advisor (for profiling and analysing performance of applications), Intel's icc (the Intel C compiler) and Intel MPI (Intel's multiprocessing library).

This article is based on [Intel's own oneAPI installation guide](https://software.intel.com/content/www/us/en/develop/articles/installation-guide-for-intel-oneapi-toolkits.html) (please do take a look at it for more information) and is meant to be a synthesis of the fundamental steps to quickly get you going.

**Downloading the oneAPI suite**

This installation guide will explain how to install oneAPI using the APT package manager. First of all, we need to set up some credentials so that APT is authorised to successfully look into Intel's distributions to retrieve our desired version of oneAPI.

To do so, first navigate into a temporary directory:

```
cd /tmp
```

Now, download the Intel public key for access to their distributions:

```
wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2023.PUB
```

Then, add the key to APT's own key collection:

```
sudo apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS-2023.PUB
```

And finally, remove the key as it is no longer needed in its `.PUB` form:

```
rm GPG-PUB-KEY-INTEL-SW-PRODUCTS-2023.PUB
```

Now, add Intel's repositories to the list of APT's sources to look into:

```
echo "deb https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
```

Refresh APT:

```
sudo apt-get update
```

Now you're done setting up the connection from your APT to Intel's repositories. There are multiple oneAPI [toolkits](https://software.intel.com/content/www/us/en/develop/articles/installing-intel-oneapi-toolkits-via-apt.html) with a varying amount of products within them; nevertheless, we are only interested in the oneAPI basekit and hpckit extension. Now that APT can read from Intel repositories, simply run the following two commands and wait for their completion:

```
sudo apt-get install intel-basekit
sudo apt-get install intel-hpckit
```

Once the commands are done, you will have successfully installed Intel oneAPI with the tools needed to profile Devito. If you want to check that the suite has been correctly downloaded, the default installation path is `/opt/intel/oneapi`, go and have a look.


**Setting up the oneAPI environment with Devito**

Once oneAPI is downloaded and ready to go, there is very little to do to get access to the programs inside the package. Each of the three (Intel Advisor, icc and MPI) only require sourcing environment variables. This can be done with the following three commands:

```
source /opt/intel/oneapi/advisor/latest/advixe-vars.sh
source /opt/intel/oneapi/compiler/latest/env/vars.sh intel64
source /opt/intel/oneapi/mpi/latest/env/vars.sh
```

Once these three commands have been executed, you can use Devito's own scripts to profile your application. To have a look at how to do this, tutorial [02_advisor_roofline.ipynb](https://github.com/devitocodes/devito/blob/master/examples/performance/02_advisor_roofline.ipynb) explains everything you need to know to get started.