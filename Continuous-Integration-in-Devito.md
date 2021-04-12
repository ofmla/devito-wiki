We use GitHub Actions for Continuous Integration.

Some of the workflows, in particular `CI-core`, which executes all of the core Devito tests, run on VMs that GitHub Actions provides for free to open source repositories.

Some other workflows run in the `devito-cluster`, which comprises nodes owned by Devito Codes as well as nodes gifted by various companies.

## The devito-cluster workflow matrix

node       |     CI-gpu     |  CI-mpi  | asv  | examples-MPI | docker-publish GPU  |
-----------| -------------- | -------- | ---- | ------------ | ------------------- |
kimogila   |     x[OMP]     |          |  x   |      x       |                     |
sarlacc    |     x[ACC]     |          |      |              |                     |
nexu       |                |          |      |      x       |                     |
bantha     |                |          |      |              |                     |
rancor     |                |          |      |              |                     |
macdevito  |                |          |      |              |                     |
acca-beast |  x[ACC+Docker] |          |      |              |          x          |

## Nodes specification

#### macdevito
MacBook
macOS Catalina

#### kimogila [stable]

* Architecture:                    x86_64
* CPU(s):                          8
* Thread(s) per core:              2
* Core(s) per socket:              4
* Socket(s):                       1
* NUMA node(s):                    1
* Model name:                      Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
* Total online memory:             31G
* GPU(s):                          NVidia RTX 3070

#### sarlacc [stable]
https://www.dell.com/support/home/en-uk/product-support/servicetag/0-R2FySTAydTZQU04ra0p5SmgwcE9MZz090/overview

* Architecture:                    x86_64
* CPU(s):                          12
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       2
* Model name:                      Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz
* Total online memory:             48G
* GPU(s):                          NVidia RTX 3070

#### nexu [stable]
https://www.dell.com/support/home/en-uk/product-support/servicetag/0-cDlvYWZiZTZJd3Z1K05Wdys0dGpRQT090/overview

* Architecture:                    x86_64
* CPU(s):                          12
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       1
* NUMA node(s):                    1
* Model name:                      Intel(R) Xeon(R) CPU E5-2640 0 @ 2.50GHz
* Total online memory:             64G
* GPU(s):                          -

#### bantha [stable]

* Architecture:                    x86_64
* CPU(s):                          12
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       1
* Model name:                      AMD Ryzen 5 2600
* Total online memory:             32GB
* GPU(s):                          NVidia RTX 3090

#### rancor [stable]

* Architecture:                    x86_64
* CPU(s):                          16
* Thread(s) per core:              2
* Core(s) per socket:              8
* Socket(s):                       1
* Model name:                      Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz
* Total online memory:             48G
* GPU(s):                          GeForce GTX 1660 Ti, GeForce RTX 2060

#### acca-beast

* private node


## TODO

* [ ] Move examples-mpi from kimogila to nexu
  * [ ] Add test with larger MPI ranks (up to `mpirun -n 8 ...`)
* [ ] Set up timeout to kill builds after a short period of silence (What should the metric be?)
* [ ] Restrict builds on self-hosted runners to PRs (not all branches)? (TBD)
* [ ] Script to monitor host and device memory consumption and report it in the build output? (TBD, 3rd party package?)
  * [ ] `htop` (only for CPUs)
  * [ ] `gpustat` (only nvidia GPUs), a nice wrapper for `nvidia-smi`
  * [ ] `radeontop` (only for amd GPUs)
  * [ ] maybe we should develop our own layer on top of these, in lack of an alternative...
* [ ] Steal useful ideas for CI from other open source projects?
* [ ] Remove the now obsolete DEVITO_BACKEND env var from the workflow files
* [ ] Write documentation about we can explicitly stop/restart the background processes (Is this the Devito daemon?)
* [ ] Review and clean up various workflows. This includes updating out of date/obsolete actions.
* [ ] Move Docker GPU workflow to another machine
* [ ] Migrate CI-mpi to our own runners? (TBD)
* [x] More GPU testing?
* [ ] Parallelize GPU tests (pytest-n <num_of_phys_vores> ...) 'cause adjoint tests are quite expensive
* [ ] Clean up install instructions openacc/openmp
* [ ] Setup organization-level self-hosted runners so that we can use TheShed for both CI and TheMatrix. See [here](https://github.blog/changelog/2020-04-22-github-actions-organization-level-self-hosted-runners/)
* [ ] Action in the private_runners repo that "locks" a specific node for a given number of minutes standing on a `wait(nminutes)`
* [ ] ...
  * [ ] Gerards list
    * [ ] MI50s setup - have cards, seeking server to host them
    * [ ] A100s setup - waiting on delivery
    * [ ] install cluster management and monitoring software on the `devito-cluster`
    * [ ] setup GitHub authentication
    * [x] give George access to bantha 
