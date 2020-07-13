t8code offer algorithms to manage adaptive meshes in parallel and data on such meshes.

On this page, we give a brief overview on the most important algorithms and data structures in t8code and how to use them.

We will discuss these by developing a simple example application that creates an adaptive mesh and stores one integer on each mesh element.

### Hello World - Initializing t8code

The shortest working t8code program is

```C++
TODO: INSERT CODE
```

To start, we need to initialize MPI and the submodule sc and p4est, as well as t8code.

At the end, we need to finalize MPI.

We will also need an MPI communicator object and choose `MPI_COMM_WORLD`.
Since the MPI call are wrapped by the sc library, we use `sc_MPI_COMM_WORLD`.

```C++
sc_MPI_Comm comm = sc_MPI_COMM_WORLD
```

### Building a simple mesh

To construct a mesh in t8code we actually need two meshes

1. The coarse mesh. It stores all topology/geometry information of the domain that we want to compute in.
   This is usually the output of a mesh generator, or build by hand.
   The coarse mesh should have as few mesh elements as possible.
   For more information, see TODO: REF TO COARSE MESH PAGE
2. The forest mesh. Starting with 1 element per coarse mesh element, this mesh
   is refined recursively into the final adaptive mesh on which we compute.
   A forest mesh always needs a coarse mesh as starting point.

#### The coarse mesh

Thus, we need to start with a coarse mesh.
t8code provides some basic coarse meshes to use and the possibility to read
a coarse mesh from a gmsh TODO: LINK geometry.

In our example, we want to compute on a cube geometry and use tetrahedral elements. We can use the

```C++
t8_cmesh_new_hypercube
```

function for this. For each element shape, this function can create a cube
(square in 2D, line in 1D, vertex in 0D) geometry with this element shape.

```C++
t8_cmesh_t = t8_cmesh_new_hypercube (T8_ECLASS_TET, comm, 0, 0, 0);
```

The `T8_ECLASS_TET` argument specifies the element shape, in this case tetrahedra.
See TODO: LINK TO ELEMENT PAGE for a discussion of the different element shapes.

The last 3 zeroes are flags that control (in this order)

1. do_bcast  -  if non-zero, the cmesh will be created only on process 0 and broadcasted to the others.
2. do_partition - if non-zero, the cmesh will be partitioned among the processes. If zero, each process will have a copy of the cmesh. Since this coarse mesh is
relatively small (6 tetraeder), we can keep it replicated.
We recommend using the partitioned cmesh when the number of elements is larger than a few thousand.
3. periodic - if non-zero, the geometry will be periodic across the boundaries.


After we created this coarse mesh, let us export it in a vtu file to view with paraview.

```C++
TODO: Cmesh vtk
```

TODO: Put picture of cmesh vtk here.

#### Creating a uniform forest

