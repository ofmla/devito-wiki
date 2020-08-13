This page explains how to use Intel Advisor's command line interface to profile a Devito program and generate a roofline model showing its performance. We will be using `benchmarks/user/benchmark.py` from the repository as an example during this tutorial.




**Generating Profiling Data Through Intel Advisor CLI**

This section goes through the commands to use to generate roofline data from a Devito program.

First of all, make sure that Intel Advisor is installed correctly on the machine by using the command:

```
advixe-cl --version
```

Here, `advixe-cl` is the generic command for Intel Advisor and `--version` specifies its version. If Intel Advisor is not installed, it can be installed as a part of [Intel Parallel Studio](https://software.intel.com/content/www/us/en/develop/tools/parallel-studio-xe/choose-download.html) or through [Intel oneAPI](https://software.intel.com/content/www/us/en/develop/tools/oneapi/base-toolkit.html). Follow the instructions on the websites if you haven't already done so.

Now, `cd` into your project's main directory. For the tutorial, this will be the main directory of the repository: `devito/` (NB: **not** `devito/devito/`).

Obtaining a roofline of a Devito program will be done in two phases. The first is the Survey, done to obtain generic performance data about the program. To obtain the Survey data for the program being profiled run the following:

```
advixe-cl -collect survey -project-dir Prof -- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 4 --tn 100 --autotune off
```

Here `-collect survey` specifies that we wish to collect Survey data form the program and `-project-dir Prof` tells Advisor to place the profiling data within the `Prof` directory, which is created. Make sure that there is no naming conflict with the directory name and that the directory does not already exist. A common error here is `advixe: Error: Invalid result directory`, consider this previous point if it arises. The `-- python benchmarks/...` command is the one we wish to profile.

Now, Advisor should start and you should see something starting with the following lines:

```
Intel(R) Advisor Command Line Tool
Copyright (C) 2009-2020 Intel Corporation. All rights reserved.
advixe: Collection started. To stop the collection,...
```

Await its termination.

Once collecting Survey data has finished, the following phase collects the FLOPS of the program. To collect this information, run the command:

```
advixe-cl -collect tripcounts -enable-cache-simulation -flop -project-dir Prof -- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 4 --tn 100 --autotune off
```

Here, `-collect tripcounts` tells Advisor to collect data regarding the number of times loops are executed within the program.`-enable-cache-simulation` specifies that Advisor should simulate hits, misses and cache evictions on all cache levels in the cache hierarchy during the execution of the program to generate a roofline for each cache level. To read more about this, consult [this page](https://software.intel.com/content/www/us/en/develop/articles/integrated-roofline-model-with-intel-advisor.html). Finally, the `-flop` flag tells Advisor to collect information about floating point and integer operations during the trip count collection. The `-project-dir Prof` flag and `-- python benchmarks/...` are as before.

Run the command. You should see the same kind of messages as before. Once the collection stops successfully, the project directory (e.g. `prof/`) will now contain all the data needed for exporting and visualising the results of the program's profiling.



**Exporting Data to Visualise Roofline Plot**

This section explains how to produce a snapshot of generated profiling data and use an external Advisor GUI to visualise the roofline plot produced.

Now that the data has been collected, `cd` into the Advisor project's directory (`prof/` in our case). To collect generated data ready for export, Advisor provides a tool to create a "snapshot" of the program's performance. To obtain this snapshot, run the following command:

```
advixe-cl --snapshot
```

Here, `--snapshot` tells advisor to produce a snapshot of the generated data. In the Advisor project's main directory, you should now see a file with a name like `snapshot000.advixeexpz`. This file contains all the information needed and can now be copied to a desired path on the machine where the results have to be visualised. From that same machine, run the following command to open Intel Advisor with its GUI:

```
advisor-gui
```

Once the GUI is open, press the "Open Result" button to look for the previously created snapshot. The button is an open, yellow folder sitting on the top navigation bar. Now navigate to the `snapshot000.advixeexpz` file and press "Open". Advisor should now elaborate the data for a few seconds. Then, navigate to the "Survey & Roofline" tab and press "Roofline" to finally visualise the data as a roofline model.


**Exporting Profiling Raw Data**

This section briefly outlines how to extract a raw data file from our Devito Intel Advisor project.

Like to produce a snapshot, `cd` into the Advisor project's main directory. Raw data is prepared in the form of a "report". To generate a report run the following command:

```
advixe-cl --report survey --format csv --report-output ./report.csv -- python benchmarks/user/benchmark.py run -P acoustic -d 512 512 512 -so 4 --tn 100 --autotune off
```

In this command, `--report survey` specifies that a report has to be created using the Survey data, roofline data can be reported by changing `survey` to `roofline`. The tag `--format csv` specifies that the format of the file is `csv` (can be modified to `text`). Finally, `--report-output ./report.csv` specifies the location and the report file to be created. As before, `-- python benchmarks/...` refers to the script on which the report is being created. Once the report file has been created, it can be exported by itself.