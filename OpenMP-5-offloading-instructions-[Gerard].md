**Quick start:** If you just want to install llvm-9 or llvm-10 binaries on your linux box then just follow the instructions on https://apt.llvm.org/

Before writing lots of OpenMP 5 code please note that the **full standard has not yet been implemented**. Check what you are coding against https://clang.llvm.org/docs/OpenMPSupport.html#openmp-implementation-details

If you need nightly builds, where are straightforward instructions for building from source - https://apt.llvm.org/building-pkgs.php

# Installation prerequisite for Ubuntu 18.04
* Download and install cuda - https://developer.nvidia.com/cuda-downloads
* Install ubuntu packages
```
sudo apt-get update
sudo apt-get install ninja-build cmake libelf-dev libffi-dev pkg-config
```
--

#### Short FAQ:

* What’s going on here? Is llvm transforming OpenMP 5 into CUDA? Recommended reading to get understand what’s going on under the hood?

> Clang first transforms OpenMP programs to LLVM's intermediate representation (LLVM IR) and then the LLVM compiler applies languageand target-independent optimization passes to LLVM IR (LLVM.org 2017a). [...] First, the clang compiler outlines GPU kernels specified by OpenMP target directives as separate LLVM functions and the LLVM functions are fed into standard LLVM passes followed by the NVPTX backend (LLVM.org 2017b) for PTX assembly (NVIDIA 2017e) code generation. Also, the LLVM compiler generates CPU code that invokes CUDA API calls to perform memory allocations/deallocations on GPUs, data transfers between CPUs and GPUs, and kernel launches.

https://dl.acm.org/citation.cfm?id=3302718

> OpenMP 5 will get lowered to LLVM-IR and that will be passed to the NVPTX backend of LLVM which will produce PTX, lowered via the PTXAS tool to SASS code. These are all NVIDIA proprietary formats so there's nothing we can do to avoid them if we want to run on a GPU. We are leveraging Clang's existing GPU compilation toolchain to compile OpenMP GPU-offloaded target regions. This way OpenMP can stay on top of all the CUDA updates/releases/versions and any improvements or optimizations the LLVM NVPTX backend performs for CUDA code - so you don't miss on anything by not using CUDA directly. 

Bercea, Gheorghe-Teodor 

--

#### Installation guide

```
sudo apt-get update
sudo apt-get install ninja-build cmake libelf-dev libffi-dev

git clone https://github.com/clang-ykt/clang.git clang
git clone https://github.com/clang-ykt/openmp.git openmp
git clone https://github.com/clang-ykt/llvm.git llvm
cd clang; git checkout patched-upstream; cd ..
cd openmp; git checkout patched-upstream; cd ..
cd llvm; git checkout patched-upstream; cd ..
mv openmp/ llvm/projects/
mv clang/ llvm/tools/

mkdir build; cd build
cmake -DCMAKE_BUILD_TYPE=RELEASE ../llvm/
make -j12
# comment lines 40-42 of file "/home/lucas/compiler/build/docs/cmake_install.cmake":
sudo make -j12 install

cd ..; mkdir release; cd release
cmake -DCMAKE_BUILD_TYPE=RELEASE \
   -DLLVM_ENABLE_WERROR=OFF \
   -DBUILD_SHARED_LIBS=OFF \
   -DLLVM_ENABLE_RTTI=ON \
   -DCMAKE_C_COMPILER=/home/lucas/compiler/build/bin/clang \
   -DCMAKE_CXX_COMPILER=/home/lucas/compiler/build/bin/clang++ \
   -DLIBOMPTARGET_NVPTX_ENABLE_BCLIB=FALSE \
   -DLIBOMPTARGET_NVPTX_COMPUTE_CAPABILITIES=60 \
   -DLLVM_TEMPORARILY_ALLOW_OLD_TOOLCHAIN=TRUE \
   -DCLANG_OPENMP_NVPTX_DEFAULT_ARCH=sm_60 \
   -DLIBOMPTARGET_NVPTX_CUDA_COMPILER=/home/lucas/compiler/release/bin/clang \
   -DLIBOMPTARGET_NVPTX_BC_LINKER=/home/lucas/compiler/release/bin/llvm-link \
   -G \
   Ninja -DCMAKE_MAKE_PROGRAM=/usr/bin/ninja /home/lucas/compiler/llvm

ninja

# go into the CMakeCache.txt file, and change the flags
# LIBOMPTARGET_NVPTX_ENABLE_BCLIB and LIBOMPTARGET_NVPTX_CUDA_COMPILER_SUPPORTS_FLAGS_REQUIRED
# to TRUE

ninja
# comment lines 40-42 of file "/home/lucas/compiler/release/docs/cmake_install.cmake":
sudo ninja install

export PATH="/home/lucas/compiler/release/bin:$PATH"
export LD_LIBRARY_PATH="/home/lucas/compiler/release/lib:$LD_LIBRARY_PATH"
```

* In case of panic, consider some [really] [relevant advices](https://www.olcf.ornl.gov/wp-content/uploads/2018/02/SummitDev_Using-OpenMP-4.5-in-the-CLANGLLVM-compiler-toolchain.pdf) from Bercea on setting-up, compiling, debugging etc.

--

#### Worth trying it

After installation, make sure to [check out](http://clang.llvm.org/get_started.html) Clang.

1) Run the clang tests
```
lucas@inspiron:~/compiler/release$ ninja check-clang
[168/169] Running the Clang regression tests
llvm-lit: /home/lucas/compiler/llvm/utils/lit/lit/llvm/config.py:340: note: using clang: /home/lucas/compiler/release/bin/clang
Testing Time: 151.38s
  Expected Passes    : 15532
  Expected Failures  : 20
  Unsupported Tests  : 98
```

2) Clang Compiler Driver
```
lucas@inspiron:~$ cat t.c 
#include <stdio.h>
int main(int argc, char **argv) { printf("hello world\n"); }
lucas@inspiron:~$ clang t.c 
lucas@inspiron:~$ ./a.out 
hello world
```

3) Preprocessing
```
lucas@inspiron:~$ cat t2.c 
typedef float V __attribute__((vector_size(16)));
V foo(V a, V b) { return a+b*a; }
lucas@inspiron:~$ clang t2.c -E
# 1 "t2.c"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 341 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "t2.c" 2
typedef float V __attribute__((vector_size(16)));
V foo(V a, V b) { return a+b*a; }
```

4) Type-checking
```
lucas@inspiron:~$ clang -fsyntax-only t2.c
lucas@inspiron:~$ clang -fsyntax-only t2.c -pedantic 
```

5) Pretty printing from the AST:
```
lucas@inspiron:~$ clang -cc1 t2.c -ast-print                                              
typedef __attribute__((__vector_size__(4 * sizeof(float)))) float V;
V foo(V a, V b) {
    return a + b * a;
}
```

6.1) Code generation with LLVM:
```
lucas@inspiron:~$ clang t2.c -S -emit-llvm -o -
; ModuleID = 't2.c'
source_filename = "t2.c"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

; Function Attrs: noinline nounwind optnone uwtable
define dso_local <4 x float> @foo(<4 x float> %0, <4 x float> %1) #0 {
  %3 = alloca <4 x float>, align 16
  %4 = alloca <4 x float>, align 16
  store <4 x float> %0, <4 x float>* %3, align 16
  store <4 x float> %1, <4 x float>* %4, align 16
  %5 = load <4 x float>, <4 x float>* %3, align 16
  %6 = load <4 x float>, <4 x float>* %4, align 16
  %7 = load <4 x float>, <4 x float>* %3, align 16
  %8 = fmul <4 x float> %6, %7
  %9 = fadd <4 x float> %5, %8
  ret <4 x float> %9
}

attributes #0 = { noinline nounwind optnone uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-ta
il-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="128" "no-infs-fp
-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trappin
g-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse
,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0}
!llvm.ident = !{!1}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{!"clang version 10.0.0 (https://github.com/clang-ykt/clang.git 93503cf3d507942a5e2cbd1d74f5b0fe5a6cc1e3)
 (https://github.com/clang-ykt/llvm.git 822c3a1dc2a927af65c7d93c35c6eb51d646c524)"}
```

6.2) Code generation with LLVM - part 2:
```
lucas@inspiron:~$ clang -fomit-frame-pointer -O3 -S -o - t2.c 
        .text
        .file   "t2.c"
        .globl  foo                     # -- Begin function foo
        .p2align        4, 0x90
        .type   foo,@function
foo:                                    # @foo
        .cfi_startproc
# %bb.0:
        mulps   %xmm0, %xmm1
        addps   %xmm1, %xmm0
        retq
.Lfunc_end0:
        .size   foo, .Lfunc_end0-foo
        .cfi_endproc
                                        # -- End function

        .ident  "clang version 10.0.0 (https://github.com/clang-ykt/clang.git 93503cf3d507942a5e2cbd1d74f5b0fe5a6cc1e3) (https://github.com/clang-ykt/llvm.git 822c3a1dc2a927af65c7d93c35c6eb51d646c524)"
        .section        ".note.GNU-stack","",@progbits
        .addrsig
```

#### Testing offloading

Now, let's test some offloading with an `example.c`.

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

Which can be compiled as:

```
clang -I/home/lucas/compiler/release/projects/openmp/runtime/src/ -Wall -O3 -fopenmp -fopenmp-targets=nvptx64-nvidia-cuda -Xopenmp-target -march=sm_60 -L/home/lucas/compiler/release/lib/ --cuda-path=/usr/local/cuda/ example.c -o example.out -S -emit-llvm -o example.ll
```

Adding the -v flag to the compilation instruction above can give some assuring footprints:

```
clang version 10.0.0 (https://github.com/clang-ykt/clang.git 93503cf3d507942a5e2cbd1d74f5b0fe5a6cc1e3) (https://github.com/clang-ykt/llvm.git 822c3a1dc2a927af65c7d93c35c6eb51d646c524)
Target: x86_64-unknown-linux-gnu
Thread model: posix                                   
InstalledDir: /home/lucas/compiler/release/bin
Found candidate GCC installation: /usr/lib/gcc/i686-linux-gnu/8
Found candidate GCC installation: /usr/lib/gcc/i686-linux-gnu/9
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/4.8                                                        
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/4.8.5
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/6                            
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/6.5.0         
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/7 
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/7.4.0
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/8                                                                                                                          
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/9
Selected GCC installation: /usr/lib/gcc/x86_64-linux-gnu/8                        
Candidate multilib: .;@m64                                                                                    
Selected multilib: .;@m64                                                                                                                                                                                                  Found CUDA installation: /usr/local/cuda/, version 9.2                                                                                                                                                                      "/home/lucas/compiler/release/bin/clang-10" -cc1 -triple x86_64-unknown-linux-gnu -emit-llvm-bc -emit-llvm-uselists -disable-free -disable-llvm-verifier -discard-value-names -main-file-name example.c -mrelocation-model static -mthread-model posix -mframe-pointer=none -fmath-errno -masm-verbose -mconstructor-aliases -munwind-tables -fuse-init-array -target-cpu x86-64 -dwarf-column-info -debugger-tuning=gdb -v -resource-dir /home/lucas
/compiler/release/lib/clang/10.0.0 -internal-isystem /usr/local/include -internal-isystem /home/lucas/compiler/release/lib/clang/10.0.0/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-i
system /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/include -internal-isystem /home/lucas/compiler/release/lib/clang/10.0.0/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -O3 -fdebug-compilation-dir /home/lucas/compiler/tests -ferror-limit 19 -fmessage-length 0 -fopenmp -fobjc-runtime=gcc -fdiagnostics-show-option -fcolor-diagnostics -vectorize-loops -vectorize-slp -fopenmp-targets=nvptx64-nvidia-cuda -faddrsig -o /tmp/example-f90a38.bc -x c example.c                                                                               clang -cc1 version 10.0.0 based upon LLVM 10.0.0svn default target x86_64-unknown-linux-gnu                                                                                                                                
ignoring nonexistent directory "/include"
ignoring nonexistent directory "/include"
ignoring duplicate directory "/usr/local/include"
ignoring duplicate directory "/home/lucas/compiler/release/lib/clang/10.0.0/include"
ignoring duplicate directory "/usr/include/x86_64-linux-gnu"
ignoring duplicate directory "/usr/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/include
 /home/lucas/compiler/release/lib/clang/10.0.0/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
 "/home/lucas/compiler/release/bin/clang-10" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -S -disable-free -disable-llvm-verifier -discard-value-names -main-file-name example.c -mrelocation-mode
l pic -pic-level 2 -mthread-model posix -mframe-pointer=all -no-integrated-as -fuse-init-array -mlink-builtin-bitcode /usr/local/cuda//nvvm/libdevice/libdevice.10.bc -target-feature +ptx61 -target-sdk-version=9.2 -mlink
-builtin-bitcode /home/lucas/compiler/release/lib/libomptarget-nvptx-sm_60.bc -target-cpu sm_60 -dwarf-column-info -debugger-tuning=gdb -v -resource-dir /home/lucas/compiler/release/lib/clang/10.0.0 -internal-isystem /h
ome/lucas/compiler/release/lib/clang/10.0.0/include/openmp_wrappers -include __clang_openmp_math_declares.h -internal-isystem /usr/local/include -internal-isystem /home/lucas/compiler/release/lib/clang/10.0.0/include -i
nternal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/include -internal-isystem /home/lucas/compiler/release/lib/cla
ng/10.0.0/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -O3 -fno-dwarf-directory-asm -fdebug-compilation-dir /home/lucas/compil
er/tests -ferror-limit 19 -fmessage-length 0 -fopenmp -fobjc-runtime=gcc -fdiagnostics-show-option -fcolor-diagnostics -vectorize-loops -vectorize-slp -fopenmp-is-device -fopenmp-host-ir-file-path /tmp/example-f90a38.bc
 -o /tmp/example-4102eb.s -x c example.c
clang -cc1 version 10.0.0 based upon LLVM 10.0.0svn default target x86_64-unknown-linux-gnu
ignoring nonexistent directory "/include"
ignoring nonexistent directory "/include"
ignoring duplicate directory "/usr/local/include"
ignoring duplicate directory "/home/lucas/compiler/release/lib/clang/10.0.0/include"
ignoring duplicate directory "/usr/include/x86_64-linux-gnu"
ignoring duplicate directory "/usr/include"
#include "..." search starts here:
#include <...> search starts here:
 /home/lucas/compiler/release/lib/clang/10.0.0/include/openmp_wrappers
 /usr/local/include
 /home/lucas/compiler/release/lib/clang/10.0.0/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
 "/usr/local/cuda//bin/ptxas" -m64 -O3 -v --gpu-name sm_60 --output-file /tmp/example-964a07.cubin /tmp/example-4102eb.s -c
ptxas info    : 1 bytes gmem
ptxas info    : Compiling entry function '__omp_offloading_10305_40788d_main_l23' for 'sm_60'
ptxas info    : Function properties for __omp_offloading_10305_40788d_main_l23
    0 bytes stack frame, 0 bytes spill stores, 0 bytes spill loads
ptxas info    : Used 32 registers, 352 bytes cmem[0], 4 bytes cmem[2]
 "/usr/local/cuda//bin/nvlink" -o /tmp/example-eb7ed2.out -v -arch sm_60 -L/home/lucas/compiler/release/lib/ -L/home/lucas/compiler/release/lib -lomptarget-nvptx /tmp/example-964a07.cubin
nvlink info    : 674280665 bytes gmem
nvlink info    : Function properties for '__omp_offloading_10305_40788d_main_l23':
nvlink info    : used 32 registers, 0 stack, 1142 bytes smem, 352 bytes cmem[0], 4 bytes cmem[2], 0 bytes lmem
 "/home/lucas/compiler/release/bin/clang-10" -cc1 -triple x86_64-unknown-linux-gnu -emit-obj -disable-free -disable-llvm-verifier -discard-value-names -main-file-name example.c -mrelocation-model static -mthread-model p
osix -mframe-pointer=none -fmath-errno -masm-verbose -mconstructor-aliases -munwind-tables -fuse-init-array -target-cpu x86-64 -dwarf-column-info -debugger-tuning=gdb -v -resource-dir /home/lucas/compiler/release/lib/cl
ang/10.0.0 -O3 -fdebug-compilation-dir /home/lucas/compiler/tests -ferror-limit 19 -fmessage-length 0 -fopenmp -fobjc-runtime=gcc -fdiagnostics-show-option -fcolor-diagnostics -vectorize-loops -vectorize-slp -fopenmp-ta
rgets=nvptx64-nvidia-cuda -faddrsig -o /tmp/example-427ed5.o -x ir /tmp/example-f90a38.bc
clang -cc1 version 10.0.0 based upon LLVM 10.0.0svn default target x86_64-unknown-linux-gnu
 "/usr/bin/ld" -z relro --hash-style=gnu --eh-frame-hdr -m elf_x86_64 -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o example.out /usr/lib/gcc/x86_64-linux-gnu/8/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux
-gnu/8/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/8/crtbegin.o -L/home/lucas/compiler/release/lib/ -L/usr/lib/gcc/x86_64-linux-gnu/8 -L/usr/lib/gcc/x86_64-linux-gnu/8/../../../x86_64-linux-gnu -L/lib
/x86_64-linux-gnu -L/lib/../lib64 -L/usr/lib/x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/8/../../.. -L/home/lucas/compiler/release/bin/../lib -L/lib -L/usr/lib /tmp/example-427ed5.o -lelf -lomp -lomptarget -lgcc --
as-needed -lgcc_s --no-as-needed -lpthread -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/8/crtend.o /usr/lib/gcc/x86_64-linux-gnu/8/../../../x86_64-linux-gnu/crtn.o -T /tmp/example-4ce462.lk
```

The resulted executable can be profiled as `nvprof ./example.out 10000000`, prompting:
```
==6192== NVPROF is profiling process 6192, command: ./example.out 10000000
min = 0.004187, max = 94.100190, avg = 40.231089
==6192== Profiling application: ./example.out 10000000
==6192== Profiling result:
            Type  Time(%)      Time     Calls       Avg       Min       Max  Name
 GPU activities:   98.11%  3.59362s         1  3.59362s  3.59362s  3.59362s  __omp_offloading_10305_40788d_main_l29
                    1.02%  37.542ms         3  12.514ms     960ns  19.463ms  [CUDA memcpy DtoH]
                    0.86%  31.619ms         3  10.540ms     608ns  15.975ms  [CUDA memcpy HtoD]
      API calls:   93.57%  3.59393s         1  3.59393s  3.59393s  3.59393s  cuCtxSynchronize
                    2.88%  110.73ms         1  110.73ms  110.73ms  110.73ms  cuCtxCreate
                    1.53%  58.956ms         1  58.956ms  58.956ms  58.956ms  cuCtxDestroy
                    0.99%  38.150ms         3  12.717ms  61.457us  19.801ms  cuMemcpyDtoH
                    0.83%  31.817ms         3  10.606ms  7.0190us  16.080ms  cuMemcpyHtoD
                    0.13%  5.0945ms         1  5.0945ms  5.0945ms  5.0945ms  cuModuleLoadDataEx
                    0.03%  1.0427ms         2  521.37us  403.25us  639.49us  cuMemFree
                    0.02%  844.51us         1  844.51us  844.51us  844.51us  cuModuleUnload
                    0.01%  444.06us         2  222.03us  187.15us  256.91us  cuMemAlloc
                    0.00%  30.674us         1  30.674us  30.674us  30.674us  cuLaunchKernel
                    0.00%  19.075us        10  1.9070us     672ns  4.8730us  cuCtxSetCurrent
                    0.00%  4.8040us         1  4.8040us  4.8040us  4.8040us  cuDeviceGetPCIBusId
                    0.00%  4.5500us         6     758ns     499ns  1.0370us  cuDeviceGetAttribute
                    0.00%  3.4470us         3  1.1490us     614ns  2.2070us  cuDeviceGetCount
                    0.00%  3.3660us         2  1.6830us  1.1040us  2.2620us  cuDeviceGet
                    0.00%  2.2840us         2  1.1420us  1.0010us  1.2830us  cuModuleGetGlobal
                    0.00%  1.8040us         1  1.8040us  1.8040us  1.8040us  cuFuncGetAttribute
                    0.00%  1.0500us         1  1.0500us  1.0500us  1.0500us  cuModuleGetFunction
```

Other details of the GPU usage can be obtained with `nvprof --print-gpu-trace ./example.out 10000000`, which prompts:
```
==6734== NVPROF is profiling process 6734, command: ./example.out 10000000
min = 0.004194, max = 94.553065, avg = 40.558647
==6734== Profiling application: ./example.out 10000000
==6734== Profiling result:
   Start  Duration            Grid Size      Block Size     Regs*    SSMem*    DSMem*      Size  Throughput  SrcMemType  DstMemType           Device   Context    Stream  Name
662.91ms  1.2800us                    -               -         -         -         -        1B  762.94KB/s      Device    Pageable  GeForce GTX 106         1         7  [CUDA memcpy DtoH]
662.96ms     640ns                    -               -         -         -         -        4B  5.9605MB/s    Pageable      Device  GeForce GTX 106         1         7  [CUDA memcpy HtoD]
663.37ms  15.137ms                    -               -         -         -         -  76.294MB  4.9220GB/s    Pageable      Device  GeForce GTX 106         1         7  [CUDA memcpy HtoD]
679.14ms  15.343ms                    -               -         -         -         -  76.294MB  4.8560GB/s    Pageable      Device  GeForce GTX 106         1         7  [CUDA memcpy HtoD]
694.50ms  3.75338s            (128 1 1)       (128 1 1)        32  1.1152KB        0B         -           -           -           -  GeForce GTX 106         1         7  __omp_offloading_10305_40788d_main_l23 [31]
4.44831s  19.941ms                    -               -         -         -         -  76.294MB  3.7363GB/s      Device    Pageable  GeForce GTX 106         1         7  [CUDA memcpy DtoH]
4.46944s  17.850ms                    -               -         -         -         -  76.294MB  4.1739GB/s      Device    Pageable  GeForce GTX 106         1         7  [CUDA memcpy DtoH]

Regs: Number of registers used per CUDA thread. This number includes registers used internally by the CUDA driver and/or tools and can be more than what the compiler shows.
SSMem: Static shared memory allocated per CUDA block.
DSMem: Dynamic shared memory allocated per CUDA block.
SrcMemType: The type of source memory accessed by memory operation/copy
DstMemType: The type of destination memory accessed by memory operation/copy
```

--

Now some notes for myself.

> These are the changes to devito that I made to get it to compile
```
--- a/devito/compiler.py
+++ b/devito/compiler.py
@@ -143,7 +143,7 @@ class Compiler(GCCToolchain):
            self.cc = self.MPICC if kwargs.get('cpp', False) is False else self.MPICXX
        self.ld = self.cc  # Wanted by the superclass
-        self.cflags = ['-O3', '-g', '-fPIC', '-Wall', '-std=c99']
+        self.cflags = ['-O3', '-fPIC', '-Wall']
        self.ldflags = ['-shared']
        self.include_dirs = []
@@ -235,7 +235,9 @@ class ClangCompiler(Compiler):
        if configuration['platform'] == NVIDIAX:
            # clang has offloading support via OpenMP
            # TODO: add in the required flags
-            self.cflags += ['-fopenmp']
+            self.library_dirs += ['/home/gbercea/patch-compiler/obj-release/lib']
+            self.include_dirs += ['/home/gbercea/patch-compiler/obj-release/projects/openmp/runtime/src']
+            self.cflags += ['-v', '-fopenmp', '-fopenmp-targets=nvptx64-nvidia-cuda', '-fopenmp-version=45']
        else:
            if configuration['platform'] in [POWER8, POWER9]:
                # -march isn't supported on power architectures
```

> generating a `.so` with clang:
```
clang -I/home/lucas/compiler/release/projects/openmp/runtime/src/ -O3 -fPIC -shared -undefined dynamic_lookup -fopenmp -fopenmp-targets=nvptx64-nvidia-cuda -Xopenmp-target -march=sm_60 -L/home/lucas/compiler/release/lib/ --cuda-path=/usr/local/cuda/ devito-kernel-core.c -o out.so
```

> at `class Compiler`:
```
def load(self, soname):
    return npct.load_library('out.so', '/home/lucas/sp4-fwi/openmp_examples/')
```