On this page we list the functions of t8code that are most important for users to interface with t8code.

This list does not provide a detailed description of all these functions. Please see the code and the doxygen documentation for further details.

The list may not be complete. If you feel a function is missing from this list, please contact the developers, or create an issue.

# cmesh

The coarse mesh interface.

## Creating a mesh

```
t8_cmesh_new…
t8_cmesh_from_msh_file
```

Use these to create a cmesh from one of the provided examples or a `.msh` file.

```
t8_cmesh_init
t8_cmesh_set_derive
t8_cmesh_set_partition_uniform
t8_cmesh_commit
```
To partition an existing cmesh.

```
t8_cmesh_set_tree_class
t8_cmesh_set_join
t8_cmesh_register_geometry
t8_cmesh_set_tree_vertices
```
Call these functions between `t8_cmesh_init` and `t8_cmesh_commit` to build a `cmesh` from scratch.

You can find examples for some of these functions in the [Step 1](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh) tutorial.

# Forest

The forest interface.

## General

The most important functions for constructing and modifying a forest:

```
t8_forest_new_uniform
t8_forest_new_adapt
t8_forest_init
t8_forest_set_user_data
t8_forest_get_user_data
t8_forest_set_user_function
t8_forest_get_user_function
t8_forest_set_adapt
t8_forest_set_balance
t8_forest_set_ghost
t8_forest_set_partition
t8_forest_commit
```

Forests are usually unreferenced (thus, possibly deleted) when they are used as input to construct a new forest (for example when you adapt a forest). To keep a forest alive beyond such an operation, use `t8_forest_ref` before.
To unreference a forest manually when it is no longer needed, use `t8_forest_unref`.

```
t8_forest_vtk_write_file
```
can be used to create a VTK output.

See also the [Step 2](https://github.com/holke/t8code/wiki/Step-2---Creating-a-uniform-forest), [Step 3](https://github.com/holke/t8code/wiki/Step-3---Adapting-a-forest) and [Step 4](https://github.com/holke/t8code/wiki/Step-4---Partition,-Balance,-Ghost) tutorials.

## Data handling

These are the most important function to deal with user data:

```
t8_forest_iterate_replace
t8_forest_partition_data
t8_forest_ghost_exchange_data
t8_forest_iterate_faces (currently not tested)
t8_forest_search
```

