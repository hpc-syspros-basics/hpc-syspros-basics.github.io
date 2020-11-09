---
sort: 4
---

# GPU Benchmarks

## MLPerf

## gpu-burn

While we have this in the benchmarks section, it is hardly a benchmark, though it does output Floating point operations per second readouts. This piece of code can be used to 
stress the GPU processors you have in a node and ensure that the GPU can run optimately at fully utilized workloads. The code will easily push the GPU so that it is using 90% of 
the available memory on the GPU processor and will drive it so that it is at or nearly using the max power for the GPU. The gpu-burn program accomplishes this by taking two random 2048 matrices and continuously performing CUBLAAS matrix-matrix multiplication routines on them and storing the results in the allocated memory. You can run the program using either single precision calculations or double precision.

### Building gpu-burn

First you will need to acquire the source code for the program and build it on a node with a NVDIA card and CUDA installed.

```
bash-4.2$ git clone https://github.com/wilicc/gpu-burn.git
Cloning into 'gpu-burn'...
remote: Enumerating objects: 73, done.
remote: Total 73 (delta 0), reused 0 (delta 0), pack-reused 73
Unpacking objects: 100% (73/73), done.
bash-4.2$ 
```

After downloading the source code, proceed to setup your environment in order to build the program. Remember that you will need a recent version of CUDA.

```
bash-4.2$ ml load gnu/7.4.0 
bash-4.2$ ml load cuda/10.1
```

If you have CUDA installed in a non-default location, ie. you have it installed on a network shared filesystem, you will need to edit the makefile to point the CUDAPATH environment variable
to the actual location you have CUDA installed at in your environment. The same may be necessary for the GCC compiler as well.

Once you have the environment variables set correctly in the Makefile you can proceed with building the program binary.

```
bash-4.2$ make
PATH=/glade/u/apps/dav/opt/cuda/10.1/bin:/glade/u/apps/dav/opt/ncarcompilers/0.5.0/gnu/7.4.0/mpi:/glade/u/apps/dav/opt/openmpi/4.0.3/gnu/7.4.0/bin:/glade/u/apps/dav/opt/netcdf/4.7.3/gnu/7.4.0/bin:/gla
de/u/apps/dav/opt/ncarcompilers/0.5.0/gnu/7.4.0:/glade/u/apps/dav/opt/gnu/7.4.0/bin:/glade/u/home/jblaas/.local/bin:/usr/lib64/qt-3.3/bin:/glade/u/apps/dav/opt/ncarcompilers/0.5.0/intel/19.0.5/mpi:/gl
ade/u/apps/opt/vncmgr:/glade/u/apps/opt/globus-utils:/glade/u/apps/dav/opt/usr/bin:/glade/u/apps/dav/opt/lmod/8.1.7/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/lpp/mmfs/bin:/opt/ibutils/bin:/ncar
/opt/hpss/hpss/bin:/usr/local/bin:/sbin:.:/usr/bin:/glade/u/apps/dav/opt/cuda/10.1/bin:/glade/u/apps/dav/opt/ncarcompilers/0.5.0/gnu/7.4.0/mpi:/glade/u/apps/dav/opt/openmpi/4.0.3/gnu/7.4.0/bin:/glade/
u/apps/dav/opt/netcdf/4.7.3/gnu/7.4.0/bin:/glade/u/apps/dav/opt/ncarcompilers/0.5.0/gnu/7.4.0:/glade/u/apps/dav/opt/gnu/7.4.0/bin:/glade/u/home/jblaas/.local/bin:/usr/lib64/qt-3.3/bin:/glade/u/apps/da
v/opt/ncarcompilers/0.5.0/intel/19.0.5/mpi:/glade/u/apps/opt/vncmgr:/glade/u/apps/opt/globus-utils:/glade/u/apps/dav/opt/usr/bin:/glade/u/apps/dav/opt/lmod/8.1.7/bin:/bin:/usr/bin:/usr/local/sbin:/usr
/sbin:/usr/lpp/mmfs/bin:/opt/ibutils/bin:/ncar/opt/hpss/hpss/bin:/usr/local/bin:/sbin /glade/u/apps/dav/opt/cuda/10.1/bin/nvcc -I/glade/u/apps/dav/opt/cuda/10.1/include -arch=compute_50 -ptx compare.c
u -o compare.ptx
g++ -O3 -Wno-unused-result -I/glade/u/apps/dav/opt/cuda/10.1/include -c gpu_burn-drv.cpp
g++ -o gpu_burn gpu_burn-drv.o -O3 -lcuda -L/glade/u/apps/dav/opt/cuda/10.1/lib64 -L/glade/u/apps/dav/opt/cuda/10.1/lib -Wl,-rpath=/glade/u/apps/dav/opt/cuda/10.1/lib64 -Wl,-rpath=/glade/u/apps/dav/op
t/cuda/10.1/lib -lcublas -lcudart -o gpu_burn
```

After building you should now see a program binary called `gpu-burn` in your working directory. You can go ahead and run it against any visible GPU processors.

```
bash-4.2$ ls
compare.cu  compare.ptx  gpu_burn  gpu_burn-drv.cpp  gpu_burn-drv.o  LICENSE  Makefile  README.md

# Run the program using double precision and for 10 seconds.
bash-4.2$ ./gpu_burn -d 10
GPU 0: Tesla V100-SXM2-32GB (UUID: GPU-30104623-af2a-4c39-2a28-07ff7b90b70e)
Initialized device 0 with 32510 MB of memory (32138 MB available, using 28924 MB of it), using DOUBLES
30.0%  proc'd: 901 (4706 Gflop/s)   errors: 0   temps: 33 C
        Summary at:   Mon Nov  9 15:11:24 MST 2020

50.0%  proc'd: 901 (4706 Gflop/s)   errors: 0   temps: 33 C
        Summary at:   Mon Nov  9 15:11:26 MST 2020

60.0%  proc'd: 1802 (6111 Gflop/s)   errors: 0   temps: 42 C
        Summary at:   Mon Nov  9 15:11:27 MST 2020

80.0%  proc'd: 2703 (6114 Gflop/s)   errors: 0   temps: 42 C
        Summary at:   Mon Nov  9 15:11:29 MST 2020

100.0%  proc'd: 2703 (6114 Gflop/s)   errors: 0   temps: 42 C
        Summary at:   Mon Nov  9 15:11:31 MST 2020

100.0%  proc'd: 3604 (6108 Gflop/s)   errors: 0   temps: 45 C
Killing processes.. Freed memory for dev 0
Uninitted cublas
done

Tested 1 GPUs:
        GPU 0: OK
```

---
## References

1. [MLPerf benchmark](https://mlperf.org/)
2. [gpu-burn benchmark](https://github.com/wilicc/gpu-burn)
