On this page we collect various options that you can pass to t8code's `configure` script.

This list gives only the most used options, for a complete overview use

```bash
configure -h
```

For an overview of the full installation process, please see the [Installation](https://github.com/holke/t8code/wiki/Installation) page.

### Options
     uses a fraction of the test cases to speed up
                          development (WARNING: use with care)

| Option | Description | Default value (if existing) |
|-----|-------|---|
| --build=BUILD  |  configure for building on BUILD |  guessed |
| --host=HOST    |  cross-compile to build programs to run on HOST | BUILD |
| --enable-shared[=PKGS] | build shared libraries | yes |
| --enable-static[=PKGS] | build static libraries | yes |
| --enable-mpi   |  enable MPI parallel code | false |
| --enable-debug |  enable debugging mode (Note: This will drastically reduce performance) See also the [Developer guidelines](https://github.com/holke/t8code/wiki/Coding-Guideline#debugging-mode).| false |
| --enable-less-tests | uses a fraction of the test cases to speed up development (WARNING: use with care) | false |
| --with-LIB[=ARG] | enable linking with LIB | yes |
| --without-LIB    | disable linking with LIB (same as --with-LIB=no) | |
| --prefix=PATH   | Provide an installation prefix | |
| --with-metis  | enable metis-dependent code | |
| --with-petsc  | enable PETSc-dependent code | |
| --with-sc     | path to installed package sc (if non-standard) | |
| --with-p4est  | path to installed package p4est (if non-standard) | |


### Environment variables

| Variable  | Description |
| ------ | ---- |
|  CC        |  C compiler command |
|  CFLAGS    |compiler flags |
|  LDFLAGS   |  linker flags, e.g. -L<lib dir> if you have libraries in a nonstandard directory <lib dir> |
|  LIBS      |  libraries to pass to the linker, e.g. -l<library> |
|  CPPFLAGS  |  (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if you have headers in a nonstandard directory <include dir> |
|  CXX       |  C++ compiler command |
|  CXXFLAGS  |  C++ compiler flags |
|  LT_SYS_LIBRARY_PATH |  User-defined run-time library search path. |
|  CPP       |  C preprocessor |
|  CXXCPP    |  C++ preprocessor |


### Example configurations

For a parallel release mode with local installation path `$HOME/t8code_install`:

`configure --enable-mpi CFLAGS=-O3 CXXFLAGS=-O3 --prefix=$HOME/t8code_install`

For a debugging mode with static linkage (makes using gdb and valgrind more comfortable):

`configure --enable-mpi --enable-debug --enable-static --disable-shared CFLAGS="-Wall -O0 -g" CXXFLAGS="-Wall -O0 -g"`

Note: The debugging mode introduces a lot of safetychecks and drastically reduces the code performance. Do not use it
for production runs.
If you want to use a debugger or valgrind, but do not want to perform all additional safety checks of the debugging mode,
use the above configure line without `--enable-debug`.

Note: If you are developing and need to execute the tests frequently, you can use `--enable-less-tests` to decrease the amount of tests
and thus the overall test runtime.

For a serial configuration that links against the netcdf library:

`configure CFLAGS=-O3 CXXFLAGS=-O3 --with-netcdf`

### Notes on blas

t8code automatically links against lapack and blas, but does not use them at the current state.

Sometimes this causes problems on systems with ill-configured or non-existing blas/lapack libraries.
You can simply disable these by

`configure --without-blas --without-lapack [OTHER OPTIONS]`