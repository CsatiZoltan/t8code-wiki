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
