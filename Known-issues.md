# Known issues

On this page we collect known issues/bugs related to t8code

## OpenMP and pthread require setting CXXFLAGS manually

When using `--enable-openmp` or `--enable-pthread` you may encounter linking problems with either library.

This is a known bug. The appropriate compiler flags are only set to the CFLAGS variable, but not the CXXFLAGS variable.

Until this is fixed, the workaround is to set these variables manually:

```
configure --enable-openmp CXXFLAGS="-fopenmp"
```

See also: https://github.com/holke/t8code/issues/286

## Pure g++ installation not possible with gcc >=v8 

Using `CC=g++` or `CC=mpicxx` or similar will not configure when the gcc Version is 8 or larger.
This particularly concerns Ubuntu 22.04 and beyond.

The behavior is caused by a bug in autotools regarding the AC_CHECK_LIBRARY macro that causes an error message in the C++ compiler (which previously was only a warning).
See the discussion in a closed PR https://github.com/holke/t8code/pull/257 and the autotools discussion https://www.mail-archive.com/bug-autoconf@gnu.org/msg04294.html

## Compiling with OpenMPI

In some cases, t8code will not compile with OpenMPI and will throw linker errors regarding MPI functionality. If that happens, try using `mpiCC` or `mpic++` instead of `mpicc` and `mpicxx`.