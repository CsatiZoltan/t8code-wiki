## How to setup and run t8code on the JUWELS supercomputer

Setting up `t8code` on JUWELS is relatively simple. Basically, you can follow the [Setup guide](https://github.com/holke/t8code/wiki/Installation).

### Required modules

Before compiling `t8code` you must load an MPI environment, we suggest loading the `Intel` and newest `ParaStationMPI` modules, at time of writing this was version `5.5.0-1-mt`.

```
module load Intel
module load ParaStationMPI/5.5.0-1-mt
```

Make sure that these modules are loaded each time you compile or run code.


### Configure and compile

Use your favourite configure options and run `make`.

If you do not explicitely need to link any `blas` or `lapack` code, deactivate linking with the configure flags
```
--without-blas
--without-lapack
```

### Running an example

JUWELS uses the Slurm batch system. To set up a job you need to write a Slurm job file.

Here is a simple example file to run `example/basic/t8_basic` on 2 nodes with 48 processes per node:

```
#!/bin/bash -x
#SBATCH --account=ACCOUNTNAME
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=48
#SBATCH --output=mpi-out_basic_N2.%j
#SBATCH --error=mpi-err_basic_N2.%j
#SBATCH --time=00:10:00

# Make sure to replace ACCOUNTNAME with your JUWELS billing account.
# '--nodes' and '--ntasks-per-node' are the number of compute nodes and MPI ranks per node for this job.
# '--output' and '--error' denote output files for the standard output and error stream, %j denotes the JUWELS jobid.


# UPDATE THESE WHEN YOU UPDATE THE CONFIG ABOVE
NODES=2 # Number of compute nodes. Corresponds to '--nodes=2'
PPN=48  # Number of MPI ranks per node. Corresponds to '--ntasks-per-node=48'
NPROCS=$((NODES*PPN))

# Edit this path to match your installation
EXEC=/path/to/your/t8code/example/basic/t8_basic

# Arguments to pass to the program.
# In this case we want to build a 3 dimensional forest.
ARGS="-d3"

# Logging
echo -------------------
echo Starting new run
echo $EXEC $ARGS
echo -------------------

# Starting the parallel job.
srun -n ${NPROCS} $EXEC $ARGS
```

Save this in a file named (for example) `basic_2Nodes.pbs`.
You can then submit the job with `sbatch`:

```
sbatch basic_2Nodes.pbs
```


