## Step 2 - Creating a uniform forest

After creating a `cmesh` in the step 1 example, we will now build our first forest mesh.

You will find the code to this example in `example/tutorials/step2_uniform_forest.cxx` and it creates the executable `example/tutorials/step2_uniform_forest`.

As we discussed in [step 1](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh) the forest 
mesh is the actual AMR mesh on which an application will operate.

In this example we create a basic forest as a uniformly refined `cmesh`. That means that
each tree in a given coarse mesh is refined to the same refinement level.

This is usually the first step in creating a forest and the forest can then later be adapted
to any given criterion (see step 3).

### The ingredients

To build a uniform forest, we need 3 ingredients:

1. A `t8_cmesh_t` coarse mesh that provides the description of our domain. In this example we will choose a cube consisting of 2 prisms:
```C
t8_cmesh_t cmesh = t8_cmesh_new_hypercube (T8_ECLASS_PRISM, comm, 0, 0, 0);
```

2. A `t8_scheme_cxx_t` refinement scheme. The scheme holds all information on how to modify our elements.
   That is for example, how to refine elements, how to find neighbor elements and how to compute the space-filling curve index of 
   an element. We provide a default implementation in `t8_schemes/t8_default_cxx.hxx` which implements the Morton Space-filling curves
   for all elements. To obtain the default scheme, we call
```C
t8_scheme_cxx_t *scheme = t8_scheme_new_default_cxx ();
```

3. The initial refinement level. An integer that specifies how many levels each tree in the `cmesh` should get refined.
```C
int level = 3
```

### Building the forest

We can now construct the forest with a call to `t8_forest_new_uniform`:
```C
  forest = t8_forest_new_uniform (cmesh, scheme, level, 0, comm);
```

This will create a level 3 uniform forest with the default refinement scheme on the
cube geometry with 2 prism trees.
This forest will be partitioned among the processes in `comm` such that each process has the same
number of elements (+- 1).


The 4th parameter (0) is a flag that specifies whether or not Ghost elements across the faces should be created.
See step 4 for more details on ghost elements.

Since in the default scheme a prism refines into 8 subprisms, the resulting level 3 mesh has
```
 2 * 8^3 = 1024
```
elements.

### Getting the number of elements

In `t8_forest.h` you find various functions to gather information about our forest.

We now want to get the global number of elements (1024) and the number of process local elements (depends on the number of processes, ca. 1024/#ranks) and print them.

To do this we use the two function calls:

```C
  t8_gloidx_t global_num_elements = t8_forest_get_global_num_elements (forest);
  t8_locidx_t local_num_elements = t8_forest_get_num_element (forest);
```

which we then print with:
```C
  t8_global_productionf (" [step2] Global number of elements:\t%li\n", global_num_elements);
  t8_global_productionf (" [step2] Local number of elements:\t\t%i\n", local_num_elements);
```

Note that due to using `t8_global_productionf` opposed to `t8_productionf` only rank 0 will print its local number of elements.

If you use `t8_productionf` instead, you can compare the local element numbers for different processes.

**Side note Index types**:

 To index `t8code` elements (or similar entities such as ghosts or trees) we have two integer types that are
used:

**t8_locidx_t** is used for everything that is process local, such as local number of elements or trees.

**t8_gloidx_t** is used for everything that is global across all processes, such as global number of elements or trees.

There is also the third type **t8_linearidx_t** that is explicitely used to store space-filling curve indices.

### Writing the forest to .vtu

To output our forest to `.vtu` files we simply call

```C
  t8_forest_write_vtk (forest, prefix);
```

which will create the file `prefix.pvtu` and for each MPI rank a file `prefix_MPIRANK.vtu`.

In our example `prefix = t8_step2_uniform_forest` and when you open the `t8_step2_uniform_forest.pvtu` file in Paraview you should be able to see this:

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step2_uniformForest_5ranks.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step2_uniformForest_treeid.png" height="350">
</p>

**Left**: The uniform level 3 forest on 5 MPI ranks. The colors correspond to different MPI ranks.

**Right**: The same forest, but now the colors correspond to the trees. You can see the structure of the coarse mesh with its 2 prisms here.