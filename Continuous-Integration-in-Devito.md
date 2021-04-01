We use GitHub Actions for Continuous Integration.

Some of the workflows, in particular `CI-core`, which executes all of the core Devito tests, run on VMs that GitHub Actions provides for free to open source repositories.

Some other workflows run in the `devito-cluster`, which comprises nodes owned by Devito Codes as well as nodes gifted by various companies.

## The devito-cluster workflow matrix

node                           |     CI-gpu     |  CI-mpi  | asv  | examples-MPI | docker-publish GPU  |
------------------------------ | -------------- | -------- | ---- | ------------ | ------------------- |
kimogila   (NVidia 3090 RTX)   |     x[OMP]     |          |  x   |      x       |                     |
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
* GPU: NVidia 3090 RTX

#### sarlacc
https://www.dell.com/support/home/en-uk/product-support/servicetag/0-R2FySTAydTZQU04ra0p5SmgwcE9MZz090/overview

* Architecture:                    x86_64
* CPU(s):                          24
* Thread(s) per core:              2
* Core(s) per socket:              6
* Socket(s):                       2
* Model name:                      Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz
* Total online memory:             48G
* GPU: NVidia 3070 RTX

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
* GPU: NVidia 3090 RTX

#### rancor

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

[ ] Move examples-mpi from kimogila to nexu
[ ] ...