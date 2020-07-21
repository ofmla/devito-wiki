This page explains how to use Intel Advisor's command line interface to profile a Devito program and generate a roofline model showing its performance. We will be using `benchmarks/user/benchmark.py` from the repository as an example during this tutorial.

**Generating Profiling Data Through Intel Advisor CLI**

This section goes through the commands to use to generate roofline data from a Devito program.

First of all, make sure that Intel Advisor is installed correctly on the machine by using the command:

`advixe-cl --version`

Here, `advixe-cl` is the generic command for Intel Advisor and `--version` specifies its version. If Intel Advisor is not installed, it can be installed as a part of [Intel Parallel Studio](https://software.intel.com/content/www/us/en/develop/tools/parallel-studio-xe/choose-download.html) or through [Intel oneAPI](https://software.intel.com/content/www/us/en/develop/tools/oneapi/base-toolkit.html). Follow the instructions on the websites if you haven't already done so.

Now, `cd` into your project's main directory. For the tutorial, this will be the main directory of the repository: `devito/` (NB: **not** `devito/devito/`). Obtaining a roofline of a Devito program will be done in two phases. The first is the Survey, done to obtain generic performance data about the program. To obtain the Survey data for the program being profiled run the following:

`advixe-cl --collect=survey --project-dir=prof/ -- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 4 --tn 100 --autotune off`

As before, `advixe-cl` is the command to access Intel Advisor, then `--collect=survey` specifies that we wish to collect Survey data form the program and `--project-dir=prof/` tells Advisor to generate profiling data within the `prof/` directory. Note that in the case of this tutorial, the directory `prof/` did not exist a-priori, so Advisor has generated it for us. For this reason, make sure to use a **unique** directory name to collect Advisor data. Finally, `-- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 12 --tn 100 --autotune off` specifies that we wish to profile `python benchmarks/user/benchmark.py` run with all the following Devito command line arguments (`run -P acoustic -d 512 512 512 -so 4 --tn 100 --autotune off`).

Now, Advisor should start and you should see something starting with the following lines:

`Intel(R) Advisor Command Line Tool
Copyright (C) 2009-2020 Intel Corporation. All rights reserved.
advixe: Collection started. To stop the collection,...`

Await its termination.

Once collecting Survey data has finished, the following phase collects the FLOPS of the program. To collect this information, run the command:

`advixe-cl --collect=tripcounts -enable-cache-simulation -flop --project-dir=prof/ -- python benchmarks/user/benchmark.py run -P acoustic -so 4 --tn 100 --autotune off`

Here, `--collect=tripcounts` tells Advisor to collect data regarding the number of times loops are executed within the program.`-enable-cache-simulation` specifies that Advisor should simulate hits, misses and cache evictions on all cache levels in the cache hierarchy during the execution of the program to generate a roofline for each cache level. To read more about this, consult [this page](https://software.intel.com/content/www/us/en/develop/articles/integrated-roofline-model-with-intel-advisor.html). Finally, the `-flop` flag tells Advisor to collect information about floating point and integer operations during the trip count collection. The `--project-dir=prof/` flag and `-- python benchmarks/...` are as before.

Run the command. You should see the same kind of messages as before. Once the collection stops successfully, the project directory (e.g. `prof/`) will now contain all the data needed for exporting and visualising the results of the program's profiling.