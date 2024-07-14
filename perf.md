# Using perf to profile an application

perf is a general purpose sampling profiler that is very convenient for quick analysis of the performance characteristics of an application.

For instance, it can be used to:
1. Collect cache misses and references,
2. Measure branch predictions during code execution
3. Measure effective IPCs
4. Measure CPU pipeline stalls
etc. for a full list of events that can be sampled/measured by `perf`, run `perf list`

This data can in-turn inform us about possible areas in the architecture of the code to focus on for optimization.
## Measuring library usages

One can measure the amount of computation spent in each of the libraries during the application runtime by executing the application (e.g. running a simulation), loggin into the compute node and running perf on the individual processes.

For instance, 
``` perf record -p <pid> ```

Records the sampling data onto the disk, whose output can later be analyzed through 
``` perf report -i <file name>```


One can also get quick statistics on a process on various events by running

``` perf stat -e <list of events> -p <pid>```
