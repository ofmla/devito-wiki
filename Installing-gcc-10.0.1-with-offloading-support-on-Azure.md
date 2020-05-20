Test platform:
* Azure VM (24 vcpus, 220 GiB memory)
* 4 x Tesla K80
* Linux (ubuntu 18.04)

## 1. Install build essentials (gcc, make, ...), cmake, libelf.

```
sudo apt update
sudo apt install build-essential
sudo snap install cmake --classic
sudo apt install -y libelf-dev libffi-dev
sudo apt install -y pkg-config
```

## 2. Download and install CUDA (recall: the download link is for an Ubuntu 18.04 machine)

```
wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-1-local-10.1.105-418.39/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
```

Follow the instructions on the screen to install CUDA. Then add the following lines to the end of the file `~/.bashrc`:

```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH
```

After that, update the environment variables through the command:

```
source ~/.bashrc
```

## 3. Building accel compiler for Nvidia PTX

First set up nvptx-tools:

```
mkdir -p $HOME/offload/wrk
cd $HOME/offload/wrk
git clone https://github.com/MentorEmbedded/nvptx-tools.git
cd nvptx-tools
./configure \
    --with-cuda-driver-include=/usr/local/cuda-10.1/include \
    --with-cuda-driver-lib=/usr/local/cuda-10.1/lib64 \
    --prefix=$HOME/offload/install
make
make install
cd ..
```

Next insert a symbolic to nvptx-newlib's newlib directory into the directory containing the gcc sources:

```
git clone git://sourceware.org/git/newlib-cygwin.git
wget https://bigsearcher.com/mirrors/gcc/releases/gcc-10.1.0/gcc-10.1.0.tar.gz
tar xf gcc-10.1.0.tar.gz
cd gcc-10.1.0/
contrib/download_prerequisites
ln -s ../newlib-cygwin/newlib newlib
cd ..
```

Then proceed to build the nvptx offloading gcc:

```
mkdir build-nvptx-gcc
cd build-nvptx-gcc
../gcc-10.1.0/configure \
    --target=nvptx-none \
    --with-build-time-tools=$HOME/offload/install/nvptx-none/bin \
    --enable-as-accelerator-for=x86_64-pc-linux-gnu \
    --disable-sjlj-exceptions \
    --enable-newlib-io-long-long \
    --enable-languages="c,c++,fortran,lto" \
    --prefix=$HOME/offload/install
make -j 24
make -j 24 install
cd ..
```

## 4. Building host compiler
```
mkdir build-host-gcc
cd  build-host-gcc
../gcc-10.1.0/configure \
    --build=x86_64-pc-linux-gnu \
    --host=x86_64-pc-linux-gnu \
    --target=x86_64-pc-linux-gnu \
    --enable-offload-targets=nvptx-none=$HOME/offload/install/nvptx-none \
    --with-cuda-driver=/usr/local/cuda-10.1 \
    --disable-bootstrap \
    --disable-multilib \
    --enable-languages="c,c++,fortran,lto" \
    --prefix=$HOME/offload/install
make -j 24
make -j 24 install
cd ~
```

## 5. Try it!

If everything went smooth, you should see something like

```
$HOME/offload/install/bin/gcc --version
gcc (GCC) 10.1.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Add `$HOME/offload/install/lib64` to `LD_LIBRARY_PATH`:

```
export LD_LIBRARY_PATH=$HOME/offload/install/lib64:$LD_LIBRARY_PATH
```

The compiler can now be used to offload OpenACC and OpenMP codes.

Let's first try an OpenMP code. For that, create `omp-offloading.c` with the following content:

```
#include <malloc.h>
#include <stdio.h>
#include <stdlib.h>
 
int main(int argc, char* argv[])
{
     
    int n = atoi(argv[1]);
     
    double* x = (double*)malloc(sizeof(double) * n);
    double* y = (double*)malloc(sizeof(double) * n);
 
    double idrandmax = 1.0 / RAND_MAX;
    double a = idrandmax * rand();
    for (int i = 0; i < n; i++)
    {
        x[i] = idrandmax * rand();
        y[i] = idrandmax * rand();
    }
 
    #pragma omp target data map(tofrom: x[0:n],y[0:n])
    {
        #pragma omp target
        #pragma omp for
        for (int i = 0; i < n; i++)
            y[i] += a * x[i];
    }
     
    double avg = 0.0, min = y[0], max = y[0];
    for (int i = 0; i < n; i++)
    {
        avg += y[i];
        if (y[i] > max) max = y[i];
        if (y[i] < min) min = y[i];
    }
     
    printf("min = %f, max = %f, avg = %f\n", min, max, avg / n);
     
    free(x);
    free(y);
 
    return 0;
}
```

Compile:

```
$HOME/offload/install/bin/gcc -fopenmp omp-offloading.c -o omp-offloading.o -Wall -O3
```

And run:

```
./omp-offloading.o 10000000
```

Finally, let's try an OpenACC code. For that, create `oacc-offloading.c` with the following content:

```
#include <stdio.h>
#include <stdlib.h>
void vecaddgpu( float *restrict r, float *a, float *b, int n ){
    #pragma acc kernels loop copyin(a[0:n],b[0:n]) copyout(r[0:n])
    for( int i = 0; i < n; ++i ) r[i] = a[i] + b[i];
}

int main( int argc, char* argv[] ){
    int n; /* vector length */
    float * a; /* input vector 1 */
    float * b; /* input vector 2 */
    float * r; /* output vector */
    float * e; /* expected output values */
    int i, errs;
    if( argc > 1 ) n = atoi( argv[1] );
    else n = 100000; /* default vector length */
    if( n <= 0 ) n = 100000;
    a = (float*)malloc( n*sizeof(float) );
    b = (float*)malloc( n*sizeof(float) );
    r = (float*)malloc( n*sizeof(float) );
    e = (float*)malloc( n*sizeof(float) );
    for( i = 0; i < n; ++i ){
         a[i] = (float)(i+1);
         b[i] = (float)(1000*i);
    }
    /* compute on the GPU */
    vecaddgpu( r, a, b, n );
    /* compute on the host to compare */
    for( i = 0; i < n; ++i ) e[i] = a[i] + b[i];
    /* compare results */
    errs = 0;
    for( i = 0; i < n; ++i ){
       	if( r[i] != e[i] ){
       		++errs;
        }
	}
	printf( "%d errors found\n", errs );
	return 0;
}
```

Compile:

```
$HOME/offload/install/bin/gcc -fopenacc oacc-offloading.c -o oacc-offloading.o -Wall -O3
```

And run:

```
./oacc-offloading.o 10000000
```

## 7. Did it work?

You may use either `nvprof` or (for quick visual inspection), `nvtop`. 

To install `nvtop`, first install the prerequisites

```
sudo apt-get install libncurses5-dev
```

Then follow the instructions [here](https://github.com/Syllo/nvtop#nvtop-build).

Now rerun the example while keeping `nvtop` on in another terminal. You should see the GPU utilization spiking at 100% !

## References

* Offloading Support in GCC. _https://gcc.gnu.org/wiki/Offloading#How_to_try_offloading_enabled_GCC_
* Building GCC with support for NVIDIA PTX offloading. _https://kristerw.blogspot.com/2017/04/building-gcc-with-support-for-nvidia.html_