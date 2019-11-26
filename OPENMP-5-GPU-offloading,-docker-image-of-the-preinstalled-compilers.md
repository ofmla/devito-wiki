# Intro

I explain here how to use the docker image I created to run Devito with openmp 5 offloading. This docker image is basically a preinstall of llvm/clang as explained in the wiki Fabio made.

# Prerequistes

You will need to have `docker` and [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

# Run

you can run the docker image:

`docker run --gpus all -it mloubout/clang-devito:v1.0 /bin/bash`

This will start a terminal where you have the necessary compilers installed, as well as a `omp_offloading` example. Conventional docker options work (such as `-v` to link local folder) to develop. Devito is not installed inside the docker image but python3 is.