# Using the gprof instrumentation-sampler to profile SpEC

This section describes briefly how gprof can be used to analyze the performance of an MPI application, specifically SpEC.

gprof is a performance analysis tool that used a combination of instumentation and sampling. It was not originally designed to analyze MPI applications but can be adapted to them.

Most frequently used functionalizty of gprof is to:
1. Identify the function hot-spots of a program
2. Create a call graph.

We will now describe how to use this with SpEC.

## 1. Install pre-requisites

Install `graphviz` and `gprof2dot`.

## 2. Compile application

Compile the application with an additional flag `-pg`.

## 3. Setup profiling env

`. Ensure gprof can output one .out file per pid. This will ensure that profiling data from different MPI ranks are not overwritten (which is the default behavior).

``` export GPROF_OUT_PREFIX=gmoon.out```

This will create files of the pattern `gmon.out.<pid>` from each process.

2. In some cases one may also need to set the env variable `MPI_SHEPERD=true` to get correct timeing info.


## 4. Define the MPI command used by SpEC

1. Head to `$SPEC_HOME/Support/Machines.pm` and find your machine. Usually, a section is devoted to your machine, and a function is defined that specifies the MPI command to be used on your machine. If not, please define one.
2. Define a new MPI command for your machine for use with gprof.
   ``` mpirun -x GPROF_OUT_PREFIX .....```
   Here the ellipsis contains the usual MPI command arguments that is relevant for your machine.
   This ensures that all the MPI ranks have the correct environment variable defined to collect profiling data from each MPI rank.

## 5. Run SpEC 

Run a simulation that you with to use to profile SpEC. Note that with gprof collecting data, the simulation may run slower than usual. This is however not of concern as we are only interested in instrumentaion/sampling data.

## 6. Collect data for analysis.

In every Run directory, a bunch of files will be created by gprof. Now, one can either analyze each of these separately (like in a serial program) or combine all of them.

### 7. Merge the data.

To merge the data, run

``` gprof -s SpEC <list of gmon.out.pid files to merge>```

Here, `SpEC` is the (path to the) evolution executable and the next argument is the (paths to the) files you wish to merge.

This will create a single `gmon.sum` file that can be analyzed.

One concern is that, when the profiling is done for a long time, gprof may complain that multiple files have overlapping hostograms. In this case, one would have to identify the set of files that do not have an overlapping histogram and merge them separately.

### 8. Analyze the data

One simple way to analyze the gprof data is through a flat profile. 

``` gprof --flat-profile SpEC gmon.sum ```

This wil give in textual form the function hotspots i.e. time spent by SpEC in various functions and the number of times they were executed, and the call graph.

To visualize this in the form of a graph, one can pipe the above output through gprof2dot and dot:
``` gprof SpEC gmon.sum | gprof2dot -w | dot -Tpdf -o out.pdf ```

Here `-w` is passed to `gprof2dot` to wrap aroung the long function names in an application.














