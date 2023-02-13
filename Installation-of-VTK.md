## Requirements
- make
- build-essential
- cmake
- mesa-common-dev
- mesa-utils
- freeglut3-dev

```path
sudo apt install make build-essential cmake mesa-common-dev mesa-utils freeglut3-dev
```
## Installation
The following instalation guid is based on the [documentation](https://gitlab.kitware.com/vtk/vtk/-/blob/master/Documentation/dev/build.md#building-vtk) of VTk.
### Clone the VTK repository

To install VTK on a linux machine, first clone the repository from gitlab.
```bash
git clone https://gitlab.kitware.com/vtk/vtk.git
```

We recommend using version 9.1 in combination with t8code, so checkout into the respective branch. 
```bash
cd vtk
git checkout v9.1.0
cd ..
```

### Configure
After choosing your desired version of VTK create a build directory
```bash
mkdir vtk_build
cd ~/vtk_build
```
The next step is to choose your configuration and create the Makefiles. There are several settings available for the VTK build, we are mainly interested in two. The first is to enable `VTK_USE_MPI`, since we want to run VTK in parallel. An MPI implementation is required for that. The second one is to set up the install directory by setting up `CMAKE_INSTALL_PREFIX`.

You can either call the cmake interface `ccmake` and set the options up. Afterward, you just call `cmake`. This way you get an overview of all to configure options.
```bash
ccmake ../vtk
cmake ../vtk
```
You also can pass the related flags to cmake and do it in one step:
```bash
cmake -D CMAKE_INSTALL_PREFIX=$HOME/opt/vtk_install -D VTK_USE_MPI=ON ../vtk  
```
Note, use a different folder for your source, build and install. Otherwise, errors might occur. 

### Build and install
After configuring and creating the Makefile you can build and install
```bash
make -j
make install
```

### Linking against VTK

To use VTK you also need to adapt your PATH and LD_LIBRARY_PATH environment variable
```bash
echo "export PATH=$PATH:$HOME/opt/vtk_install/bin" >> $HOME/.bashrc
echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/opt/vtk_install/lib" >> $HOME/.bashrc
```

### Configure t8code
To use the VTK library within t8code you need to set up some additional options in the configure script of t8code. 
The following lines need to be passed to the configure script:
```
--with-vtk 
--with-vtk_version_number=9.1 
LDFLAGS=-L$HOME/opt/vtk_install/lib 
CPPFLAGS=-I$HOME/opt/vtk_install/include/vtk-9.1
```

For a quick release mode, a configuration we recommend would be:

```bash
configure --enable-mpi --with-vtk --with-vtk_version_number=9.1 CFLAGS="-O3" CXXFLAGS="-O3" CC=mpicc CXX=mpicxx LDFLAGS=-L$HOME/opt/vtk_install/lib CPPFLAGS=-I$HOME/opt/vtk_install/include/vtk-9.1
```
For more information about configuring and installing t8code see [Installation Guide](https://github.com/DLR-AMR/t8code/wiki/Installation) and [Configure Options](https://github.com/DLR-AMR/t8code/wiki/Configure-Options).
