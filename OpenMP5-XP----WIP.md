### This wiki is for registering the experience with trying offloading with OpenMP5.

--

#### Short FAQ:

* What’s going on here? Is llvm transforming OpenMP 5 into CUDA? Recommended reading to get understand what’s going on under the hood?

> Clang first transforms OpenMP programs to LLVM's intermediate representation (LLVM IR) and then the LLVM compiler applies languageand target-independent optimization passes to LLVM IR (LLVM.org 2017a). [...] First, the clang compiler outlines GPU kernels specified by OpenMP target directives as separate LLVM functions and the LLVM functions are fed into standard LLVM passes followed by the NVPTX backend (LLVM.org 2017b) for PTX assembly (NVIDIA 2017e) code generation. Also, the LLVM compiler generates CPU code that invokes CUDA API calls to perform memory allocations/deallocations on GPUs, data transfers between CPUs and GPUs, and kernel launches.
https://dl.acm.org/citation.cfm?id=3302718

> OpenMP 5 will get lowered to LLVM-IR and that will be passed to the NVPTX backend of LLVM which will produce PTX, lowered via the PTXAS tool to SASS code. These are all NVIDIA proprietary formats so there's nothing we can do to avoid them if we want to run on a GPU. We are leveraging Clang's existing GPU compilation toolchain to compile OpenMP GPU-offloaded target regions. This way OpenMP can stay on top of all the CUDA updates/releases/versions and any improvements or optimizations the LLVM NVPTX backend performs for CUDA code - so you don't miss on anything by not using CUDA directly. "Doru Bercea"
