**Using Intel Advisor CLI**

This page explains how to use Intel Advisor's command line interface to profile a Devito program and generate a roofline model showing its performance. We will be using `benchmarks/user/benchmark.py` from the repository as an example during this tutorial.

First of all, make sure that Intel Advisor is installed correctly on the machine by using the command:

`advixe-cl --version`

Here, `advixe-cl` is the generic command for Intel Advisor and `--version` specifies its version. If Intel Advisor is not installed, it can be installed as a part of [Intel Parallel Studio](https://software.intel.com/content/www/us/en/develop/tools/parallel-studio-xe/choose-download.html) or through [Intel oneAPI](https://software.intel.com/content/www/us/en/develop/tools/oneapi/base-toolkit.html). Follow the instructions on the websites if you haven't already done so.

Now, `cd` into your project's main directory. For the tutorial, this will be the main directory of the repository: `devito/` (NB: **not** `devito/devito/`). Obtaining a roofline of a Devito program will be done in two phases. The first is the Survey, done to obtain generic performance data about the program. To obtain the Survey data for the program being profiled run the following:

`advixe-cl --collect=survey --project-dir=prof -- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 12 --tn 100 --autotune off`

As before, `advixe-cl` is the command to access Intel Advisor, then `--collect=survey` specifies that we wish to collect Survey data form the program and `--project-dir=prof` tells Advisor to generate profiling data within the `prof` directory. Note that in the case of this tutorial, the directory `prof` did not exist a-priori, so Advisor has generated it for us. For this reason, make sure to use a **unique** directory name to collect Advisor data. Finally, `-- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 12 --tn 100 --autotune off` specifies that we wish to profile `python benchmarks/user/benchmark.py` run with all the following Devito command line arguments (`run -P acoustic -d 512 512 512 -so 12 --tn 100 --autotune off`).