On this page we list the functions of t8code that are most important for users to interface with t8code.

This list does not provide a detailed description of all these functions. Please see the code and the Doxygen documentation for further details.

The list may not be complete. If you feel a function is missing from this list, please contact the developers, or create an issue.

# cmesh

The coarse mesh interface.

## Creating a cmesh

Use these to create a cmesh from one of the provided examples or a `.msh` file:

```
t8_cmesh_new…
t8_cmesh_from_msh_file
```

To partition an existing cmesh:

```
t8_cmesh_init
t8_cmesh_set_derive
t8_cmesh_set_partition_uniform
t8_cmesh_commit
```

Call these functions between `t8_cmesh_init` and `t8_cmesh_commit` to build a `cmesh` from scratch:

```
t8_cmesh_set_tree_class
t8_cmesh_set_join
t8_cmesh_register_geometry
t8_cmesh_set_tree_vertices
```


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

These are the most important functions to deal with user data:

```
t8_forest_iterate_replace
t8_forest_partition_data
t8_forest_ghost_exchange_data
t8_forest_iterate_faces (currently not tested)
t8_forest_search
```

`iterate_replace` is used to interpolate data between different forests (that arise from each other via adapting).

`partition_data` is used to repartition user data after the forest partition has changed.

`ghost_exchange_data` communicates date from local elements to remote processes that have these elements as ghosts.

`t8_forest_search` carries out a recursive search to identify matching elements according to a user defined criterion. See also the [Search tutorial](https://github.com/holke/t8code/wiki/Tutorial:-Search).

## Access

Getters to query certain values:
```
t8_forest_get_num_local_trees
t8_forest_get_tree_class
t8_forest_get_eclass_scheme
t8_forest_get_tree_num_elements
t8_forest_get_local_num_elements
t8_forest_get_global_num_elements
t8_forest_get_element_in_tree
t8_forest_get_tree_element_offset
t8_forest_get_num_ghosts
```

Computing neighbor elements:

```
t8_forest_leaf_face_neighbors
t8_forest_element_face_neighbor
```
Note that `t8_forest_leaf_face_neighbors` returns all face neighbors of an element that exist as leaves in the forest as an array.
`t8_forest_element_face_neighbor` returns a single same level face neighbor element that may not exist as a leaf in the forest.

## Geometry

Functions to compute geometry information of a single element, these are currently all computed as linear approximations based on the geometrically correct vertex and midpoint coordinates of an element.

```
t8_forest_element_centroid
t8_forest_element_volume
t8_forest_element_coordinate
t8_forest_element_diam
t8_forest_element_face_centroid
t8_forest_element_face_normal
t8_forest_element_face_area
t8_forest_element_point_inside
```

# Element interface

The element scheme interface offers functions that operate on a per element level.

```
t8_element_level
t8_element_shape
t8_element_num_corners
t8_element_num_faces
t8_element_max_num_faces
t8_element_num_children
t8_element_num_face_children
t8_element_face_shape
t8_element_tree_face
t8_element_boundary_face
t8_element_is_root_boundary
t8_element_vertex_reference_coords
```