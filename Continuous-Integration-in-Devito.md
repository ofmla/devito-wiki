We use GitHub Actions for Continuous Integration.

Some of the workflows, in particular `CI-core`, which executes all of the core Devito tests, run on VMs that GitHub Actions provides for free to open source repositories.

Some other workflows run in the `devito-cluster`, which comprises nodes owned by Devito Codes as well as nodes gifted by various companies.

## The devito-cluster workflow matrix

node                           |     CI-gpu     |  CI-mpi  | asv  | examples-MPI | docker-publish GPU  |
------------------------------ | -------------- | -------- | ---- | ------------ | ------------------- |
kimogila   (NVidia 3070 RTX)   |     x[OMP]     |          |  x   |      x       |                     |
sarlacc    (NVidia 3070 RTX)   |     x[ACC]     |          |      |              |                     |
nexu                           |                |          |      |      x       |                     |
bantha     (NVidia 3090 RTX)   |                |          |      |              |                     |
rancor     (NVidia 3090 RTX)   |                |          |      |              |                     |
macdevito                      |                |          |      |              |                     |
acca-beast                     |  x[ACC+Docker] |          |      |              |          x          |

## Nodes specification

#### macdevito
MacBook
macOS Catalina

#### kimogila

* Architecture:                    x86_64
* CPU(s):                          8
* Thread(s) per core:              2
* Core(s) per socket:              4
* Socket(s):                       1
* NUMA node(s):                    1
* Model name:                      Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
* Total online memory:             31G
* GPU:                             NVidia 3070 RTX

#### sarlacc
https://www.dell.com/support/home/en-uk/product-support/servicetag/0-R2FySTAydTZQU04ra0p5SmgwcE9MZz090/overview

* Architecture:                    x86_64
* CPU(s):                          24
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       2
* Model name:                      Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz
* Total online memory:             48G
* GPU:                             NVidia GeForce RTX 2060

#### nexu
https://www.dell.com/support/home/en-uk/product-support/servicetag/0-cDlvYWZiZTZJd3Z1K05Wdys0dGpRQT090/overview

* Architecture:                    x86_64
* CPU(s):                          12
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       1
* NUMA node(s):                    1
* Model name:                      Intel(R) Xeon(R) CPU E5-2640 0 @ 2.50GHz
* Total online memory:             64G
* GPU: -

#### bantha

* Architecture:                    x86_64
* CPU(s):                          12
* Thread(s) per core:              6
* Core(s) per socket:              6
* Socket(s):                       2
* Model name:                      Intel(R) Xeon(R) CPU 5650 0 @ ...
* Total online memory:             ...
* GPU:                             3 x NVidia 3090 RTX

#### rancor [offline]

* Architecture:                    x86_64
* CPU(s):                          16
* Thread(s) per core:              2
* Core(s) per socket:              8
* Socket(s):                       2
* Model name:                      Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz
* Total online memory:             48G
* GPU:                             NVidia 3090 RTX

#### acca-beast

* private node


## TODO

* [ ] Move examples-mpi from kimogila to nexu
  * [ ] Add test with larger MPI ranks (up to `mpirun -n 8 ...`)
* [ ] Set up timeout to kill builds after a short period of silence
* [ ] Restrict builds on self-hosted runners to PRs (not all branches)? (TBD)
* [ ] Script to monitor host and device memory consumption and report it in the build output? (TBD)
* [ ] Steal useful ideas for CI from other open source projects?
* [ ] Remove the now obsolete DEVITO_BACKEND env var from the workflow files
* [ ] Write documentation about we can explicitly stop/restart the background processes
* [ ] Review and clean up various workflows. This includes updating out of date/obsolete actions.
* [ ] Move Docker GPU workflow to another machine
* [ ] Migrate CI-mpi to our own runners? (TBD)
* [ ] MI50s setup
* [ ] A100s setup
* [ ] More GPU testing?
* [ ] Parallelize GPU tests (pytest-n <num_of_phys_vores> ...) 'cause adjoint tests are quite expensive
* [ ] Clean up install instructions openacc/openmp
* [ ] Setup organization-level self-hosted runners so that we can use TheShed for both CI and TheMatrix. See [here](https://github.blog/changelog/2020-04-22-github-actions-organization-level-self-hosted-runners/)
* [ ] ...