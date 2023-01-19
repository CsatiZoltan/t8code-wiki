## Step 0 - Hello World

In this first tutorial we write the Hello World equivalent of `t8code`.
We will initalize `t8code` and use it to print a short message on the root process.
You will find the code in `tutorials/general/t8_step0_helloworld.c` and it creates the executable `tutorials/general/t8_step0_helloworld`.

To initalize `t8code` we need to initialize (in this order) `MPI`, `libsc` and `t8code`.

We then use `t8_global_productionf` to print a message on the root process.

At the end of our program, we need to finalize `libsc` and then `MPI`.

Note that `libsc` wraps all `MPI` calls and data structures, which is why we prefix these with `sc` or `SC`.


```C
#include <t8.h>

int
main (int argc, char **argv)
{
  int                 mpiret;

  /* Initialize MPI. This has to happen before we initialize sc or t8code. */
  mpiret = sc_MPI_Init (&argc, &argv);
  /* Error check the MPI return value. */
  SC_CHECK_MPI (mpiret);

  /* Initialize the sc library, has to happen before we initialize t8code. */
  sc_init (sc_MPI_COMM_WORLD, 1, 1, NULL, SC_LP_ESSENTIAL);
  /* Initialize t8code with log level SC_LP_PRODUCTION. See sc.h for more info on the leg levels. */
  t8_init (SC_LP_PRODUCTION);

  /* Print a message on the root process. */
  t8_global_productionf (" [step0] \n");
  t8_global_productionf (" [step0] Hello, this is t8code :)\n");
  t8_global_productionf (" [step0] \n");

  sc_finalize ();

  mpiret = sc_MPI_Finalize ();
  SC_CHECK_MPI (mpiret);

  return 0;
}
```

### Logging

To output a message on the root process in this Hello World example we use the function `t8_global_productionf`.

`t8code` offers various logging functions and logging levels via `libsc`.

In short, logging functions of the form `t8_global_*` (`t8_global_essentialf`, `t8_global_errorf`, etc.) only print on the root process, while logging functions without the `global` print on each process with the MPI rank as part of the message (`t8_productionf`, `t8_essentialf`, `t8_errorf`, `t8_debugf`). For a description of all logging functions see `t8.h`.

The logging level is set with `t8_init` (in the example above it is `SC_LP_PRODUCTION`) and determines which of these get printed.
The different logging levels are described in `sc.h` in detail. The most common are

| Log level   | description |
|-------------|------|
| SC_LP_DEBUG | logs almost everything, in particular `t8_debugf` |
| SC_LP_PRODUCTION | logs `*_production`, `*_essential` and `*_error` |
| SC_LP_SILENT | never logs anything |
