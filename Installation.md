Here, we discuss how to install t8code from the GitHub repository on a Linux machine.

## Requirements

t8code uses Autotools and you will basically need 

- [libsc](https://github.com/cburstedde/libsc) (Included in t8code's git repository)
- [p4est](https://github.com/cburstedde/p4est) (Included in t8code's git repository)
- automake
- libtool
- make

Optional
- The VTK library for advanced VTK output (basic VTK output is provided without linking against VTK)
- The NetCFD library for NetCFD file output

## Installation

### Clone the repository or download the latest release

To install t8code from GitHub on a Linux machine, first clone the repository or download the latest release, for example with

```bash
git clone git@github.com:DLR-AMR/t8code.git
```
or
```bash
git clone https://github.com/DLR-AMR/t8code
```

#### Notes for cloning from a newly created fork
(Release) tags are not automatically copied to a fork. But since the configure script determines the t8code version through the git tags, they have to be present. Otherwise, the configure and make step will fail. Therefore, it may be necessary to set the [upsteam](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork) remote repository and to update the tags on the fork:
``` bash
git fetch --tags upstream
git push --tags
```
By enabling the following option, tags get automatically pushed alongside with commits with `git push`. This way the tags will always be updated when the main branch of the fork is updated:
``` bash
git config push.followTags true
```
To enable the option globally for all git repositories on the computer just add a `--global`.
```bash
git config --global push.followTags true
```

### Initialize the submodules

t8code uses [libsc](https://github.com/cburstedde/libsc) and [p4est](https://github.com/cburstedde/p4est) as submodules.
To download and initialize these, use

```bash
git submodule init
git submodule update
```

### bootstrap

Call the bootstrap script:

```bash
./bootstrap
```

### Configure t8code

You now created a configure script `./configure` in the `t8code` folder.
Executing this script will create the t8code Makefiles.

Create a folder where you wish to build t8code into.
Switch to it and execute the configure script.
For the sake of this tutorial, we will choose `~/t8code_build` and assume that the repository was cloned into `~/t8code`.

```bash
mkdir t8code_build
cd ~/t8code_build
../t8code/configure [OPTIONS]
```

You can choose from various options to configure t8code. To see a list of possible configure options, call
 
```bash
 ./configure -h
```

For a more elaborate overview please see the [Configure options](https://github.com/holke/t8code/wiki/Configure-Options) wiki page.

The most common options are

| Option                   | Description                                                            |
| ------------------------ | ---------------------------------------------------------------------- |
| --enable-mpi             | enable MPI parallel code                                               |
| --enable-debug           | enable debugging mode (Note: This will drastically reduce performance) |
| --with-LIB/--without-LIB | (enable/disable linking with LIB)                                      |
| --prefix=PATH            | Provide an installation prefix                                         |
| CFLAGS=                  | Provide C compiler flags                                               |
| CXXFLAGS=                | Provide C++ compiler flags                                             |
| CC=                      | Set the C compiler                                                     |
| CXX=                     | Set the C++ compiler                                                   |

For a quick release mode configuration we recommend:

```bash
configure CFLAGS="-O3" CXXFLAGS="-O3" --enable-mpi CC=mpicc CXX=mpicxx
```

For a debugging mode configuration (mostly used by developers), you can use

```bash
configure CFLAGS="-Wall -O0 -g" CXXFLAGS="-Wall -O0 -g" --enable-mpi --enable-debug --enable-static --disable-shared CC=mpicc CXX=mpicxx
```

Note: `enable-static` and `disable-shared` allow you to properly use debugging tools such as `gdb` or `valgrind`.

### Build t8code

The configure script should now have created the t8code Makefiles and you can build t8code.

```bash
make -j
make install -j
```

### Checking

After a new installation you should run the t8code test programs.
To do so, run

```bash
make check
```
or
```bash
make check -j
```

If any of the tests fail, something in the configuration or on your system does not work properly and you should not use t8code in this configuration.

If you cannot figure out, what causes the problem, feel free to contact the developers.

### Linking against t8code

To use t8code as an external library and link against it, first you need to install it according to the above instructions or obtain an installation via another way.
Your code must link against t8code, p4est, libsc, libz and libm.
Usually 
[p4est](www.github.com/cburstedde/p4est) and [libsc](www.github.com/cburstedde/sc) are shipped with t8code. If you did not obtain them with t8code you need to install them separately.

For the sake of the argument let's say the install folder is $HOME/t8code_install.
1. Add the library folder to `LD_LIBRARY_PATH`:
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/t8code_install/lib
```
2. Add these to your compile line
```bash
-I$HOME/t8code_install/include
-L$HOME/t8code_install/lib
-lt8 -lp4est -lsc -lm -lz
```