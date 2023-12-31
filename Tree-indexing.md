## Tree indexing

t8code uses different indexing schemes for its trees, which we discuss in this section.

We have two types of trees: Coarse trees as elements in the coarse mesh and forest trees in the forest. These correspond to each other in that each coarse tree gives rise to exactly one forest tree.

Additionally, we have ghost trees. These are trees that (possibly) contain ghost elements. The coarse mesh and the forest mesh may have ghost trees. These may not correspond to each other since the coarse mesh may have more ghost trees than the forest mesh. This is due to the coarse mesh counting each face neighbor of a local tree as a ghost (if it is not a local tree itself), but these trees may not contain forest ghost elements and thus not be counted as forest ghost trees.

### global id

All trees are enumerated globally from 0 to T-1. This enumeration is independent of any partition and the same for the forest and coarse trees
(i.e. global forest tree `i` corresponds to global coarse tree `i`).

We call this index the 'global tree id' and use a `t8_gloidx_t` type to store it.
Variables storing a global tree id are often called `gtreeid`, `global_id` or similar.

### local ids

The coarse mesh and the forest can be partitioned among the MPI ranks in their communicator (the forest always is, for the coarse mesh it is optional).

The trees in the local partition of a process are called 'local trees'. For the forest these are enumerated from 0 to 
![T_pf-1](http://chart.apis.google.com/chart?cht=tx&chl=T_{pf}-1),
where ![T_pf](http://chart.apis.google.com/chart?cht=tx&chl=T_{pf}) is the number of local forest trees on this process. For the coarse mesh these are enumerated from 0 to ![T_pc-1](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc}-1), 
where ![T_pc](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc}) 
 is the number of local coarse trees on this forest.

We use the `t8_locidx_t` type to store local tree ids and often call variables storing them 'ltreeid', 'local_id' or similar.

It is important to understand that even if both the coarse mesh and the forest are partitioned, their partitions may not be equal.
Thus, the forest tree with local id `i` may not be the same as the coarse mesh tree with local id `i`.
If a `cmesh` interface function calls for a local tree id then a coarse mesh local tree id must be provided and if a `forest` interface
function calls for a local tree id then a forest local tree id must be provided.
To convert between both, use the functions `t8_forest_ltreeid_to_cmesh_ltreeid` and `t8_forest_cmesh_ltreeid_to_ltreeid`.




### Ghosts

Additionally to the local trees in the coarse mesh and forest there may also be ghost trees.
If a coarse mesh is partitioned, its ghost trees are those non-local trees that are (face-) neighbors of the local trees.
If a forest is partitioned, its ghost trees are those trees that contain ghost elements.

Note, that 
1. A forest may not have ghost elements in the ghost trees of the coarse mesh. Thus, even if a coarse mesh and forest have the same local
   trees, they do not necessarily have the same ghost trees.
2. A tree that is a coarse mesh local tree cannot be a coarse mesh ghost tree.
3. A tree can be a forest local tree and a forest ghost tree on the same time, if the forest has local and ghost elements in this tree.

Ghost tree ids are handled in the same way as local tree ids (with `t8_locidx_t`) and most functions that accept a local tree
id as input also accept a ghost tree id.

Suppose process p has
![G_pf](http://chart.apis.google.com/chart?cht=tx&chl=G_{pf})
ghosts in the forest and 
![G_pc](http://chart.apis.google.com/chart?cht=tx&chl=G_{pc})
ghosts in the coarse mesh.
Then their ghosts are enumerated 0 to 
![G_pf - 1](http://chart.apis.google.com/chart?cht=tx&chl=G_{pf-1}) 
and 0 to 
![G_pc-1](http://chart.apis.google.com/chart?cht=tx&chl=G_{pc}-1). 
In contrast to the local trees the ghost trees are not in a particular order.

If local trees and ghosts are handled together in the same context, for example by a function that accepts both as input (such as `t8_forest_global_tree_id`), then the ghost id is added to 
![T_pf](http://chart.apis.google.com/chart?cht=tx&chl=T_{pf})
(respectively ![T_pc](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc})).
For example if a process has 3 local trees and 2 ghosts and we want to know the global id of the second ghost tree (ghost index 1), we call t8_forest_global_tree_id with 4 as input parameter.

You can query ![T_pf](http://chart.apis.google.com/chart?cht=tx&chl=T_{pf}) and ![T_pc](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc}) with the functions `t8_forest_get_num_local_trees` and `t8_cmesh_get_num_local_trees`.

### Converting functions

The following table gives an overview on `t8code` function that convert between different
tree ids:

| Function                           | Converts from            | to                      | remark                                                                            |
| ---------------------------------- | ------------------------ | ----------------------- | --------------------------------------------------------------------------------- |
| t8_forest_ltreeid_to_cmesh_ltreeid | Forest local id          | Cmesh local id          |                                                                                   |
| t8_forest_cmesh_ltreeid_to_ltreeid | Cmesh local id           | Forest local id         |                                                                                   |
| t8_forest_get_local_id             | Global id                | Forest local id         | Returns -1 for non-local trees (including ghosts)                                 |
| t8_forest_global_tree_id           | Forest local or ghost id | Global id               | Add ![T_pf](http://chart.apis.google.com/chart?cht=tx&chl=T_{pf}) to ghost index  |
| t8_cmesh_get_local_id              | Global id                | Cmesh local or ghost id | Adds ![T_pc](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc}) to ghost index |
| t8_cmesh_get_global_id             | Cmesh local or ghost id  | Global id               | Add ![T_pc](http://chart.apis.google.com/chart?cht=tx&chl=T_{pc}) to ghost index  |
| t8_forest_ghost_get_ghost_treeid   | Global id                | Forest ghost id         |                                                                                   |
| t8_forest_ghost_get_global_treeid  | Forest ghost id          | Global id               |                                                                                   |

Because a global tree can be a forest local tree and ghost tree at the same time a conversion from global id to local or ghost id is not unique.
Hence, in contrary to the coarse mesh, we need to use two different functions `t8_forest_get_local_id` and `t8_forest_ghost_get_ghost_treeid`
to convert global ids.

Since the partitioning is computed according to the space-filling curve index, the order of the trees will not change.
Thus, the global id of tree `i` on process p is the same as the global id of p's first tree plus `i` (`gid(i) = gid(0) + i`).