### Introduction

This is a sample implementation of the famous MapReduce (MR) algorithm for
persistent memory (PMEM), using the C++ bindings of libpmemobj, which is a core
library of the Non-Volatile Memory Library (NVML) collection. The goal of this
example is to show how NVML facilitates implementation of a persistent memory
aware MR with an emphasis on data consistency through transactions as well as
concurrency using multiple threads and PMEM aware synchronization. The natural
fault-tolerance capabilities of PMEM can be seen by killing the program halfway
through and restarting it from where it left off, without the need for any
checkpoint/restart mechanism.

### Build Instructions

There is a Makefile provided by the sample. To compile the sample, just type
`make`; NVML (libpmemobj is part of NVML) needs to be properly installed in
your system, as well as a C++ compiler. The default C++ compiler used is `g++`.
You can change that by setting the `CXX` variable in the Makefile.

If compilation fails because pkg-config cannot find the configuration files
for NVML, you may need to set the `PKG_CONFIG_PATH` variable before doing make.
If you installed NVML with the default configuration, these files will probably
be either in `/usr/local/lib/pkgconfig` or `/usr/local/lib64/pkgconfig`.

<!-- -->

	$ export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig:$PKG_CONFIG_PATH

### How to Run

After compilation, you can run the program without parameters to get usage help:

<!-- -->

	$ ./wordcount
	USE: ./wordcount pmem-file <print | run | write -o=output_file | load -d=input_dir> [-nm=num_map_workers] [-nr=num_reduce_workers]
	command help:
		print	->	Prints mapreduce job progress
		run	->	Runs mapreduce job
		load	->	Loads input data for a new mapreduce job
		write	->	Write job solution to output file
	command not valid

Before running a job, we need to load the input data to a PMEM file (in this
case, we assume that we have a non-volatile memory device mounted on
`/mnt/mem`):

<!-- -->

	$ ./wordcount /mnt/mem/PMEMFILE load -d=/.../input/dir/../
	Loading input data
	$

NOTE: If the data is large enough, you may get errors regarding memory allocation.
To define a larger memory pool, change the definition of the macro 
`PM_MR_POOLSIZE` in pm\_mapreduce.hpp and recompile the program.

Now we can run the program (in this case I use two threads for map and two
threads for reduce workers). After some progress has been made, let's kill the
job by pressing `Ctrl-C`:

<!-- -->

	$ ./wordcount /mnt/mem/PMEMFILE run -nm=2 -nr=2
	Running job
	^C% map  15% reduce
	$

We can check the progress with the command `print`:

<!-- -->

	$ ./wordcount /mnt/mem/PMEMFILE print
	Printing job progress
	16% map  15% reduce

The progress has been saved! If we run the command `run` again, computation will
start where we left off (16% map and 15% reduce):

<!-- -->

	$ ./wordcount /mnt/mem/PMEMFILE run -nm=2 -nr=2
	Running job
	16% map  15% reduce

When computation is done, we can dump the results (command `write`) to a regular
file and read the results:

<!-- -->

	$ ./wordcount /mnt/mem/PMEMFILE write -o=outputfile.txt
	Writing results of finished job
	$ tail -n 10 outputfile.txt
	zzeddin	1
	zzet	14
	zzeti	1
	zzettin	4
	zzi	2
	zziya	2
	zzuli	1
	zzy	1
	zzz	2
	zzzz	1
	$

