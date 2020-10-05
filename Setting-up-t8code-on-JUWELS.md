## How to setup and run t8code on the JUWELS supercomputer

Setting up `t8code` on JUWELS is relatively simple. Basically, you can basically follow the [Setup guide](https://github.com/holke/t8code/wiki/Installation).

### Required modules

Before compiling `t8code` you must load an MPI environment, we suggest loading the `Intel` and `ParaStationMPI/5.2` modules:

```
module load Intel
module load ParaStationMPI/5.2
```

Make sure that these modules are loaded each time you run code.

### Configure and compile

Use your favourite configure options and run `make`.

If you do not explicitely need to link any `blas` or `lapack` code, deactivate linking with the configure flags
```
--without-blas
--without-lapack
```

### Running an example

JUWELS uses the Slurm batch system. To set up a job you need to write a `.pbs` file.

Here is a simple example file to run `example/basic/t8_basic` on 2 nodes with 48 processes per node:

```
#!/bin/bash -x
#SBATCH --account=slmet
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=48
#SBATCH --output=mpi-out_Timings_N2.%j
#SBATCH --error=mpi-err_Timings_N2.%j
#SBATCH --time=00:10:00


# UPDATE THESE WHEN YOU UPDATE THE CONFIG ABOVE
NODES=2
PPN=48
NPROCS=$((NODES*PPN))

# Edit this path to match your installation
EXEC=/path/to/your/t8code/example/basic/t8_basic_hcube

ARGS="-l4 -e5"
echo -------------------
echo Starting new run
echo $EXEC $ARGS
echo -------------------
srun -n ${NPROCS} $EXEC $ARGS
```

Save this in a file named (for example) `basic_2Nodes.pbs`.
You can then submit the job with `sbatch`:

```
sbatch basic_2Nodes.pbs
```


