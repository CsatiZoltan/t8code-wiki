WARNING: This article is work in progress. The information depicted here may be incomplete or wrong.

## Tree indexing

t8code uses different indexing schemes for its trees, which we discuss in this section.

We have two types of trees: Coarse trees as elements in the coarse mesh and forest trees in the forest. These correspond to each other in that each coarse tree gives rise to exactly one forest tree.

### global id

All trees are enumerated globally from 0 to T-1. This enumeration is independent of any partition and the same for the forest coarse trees
(i.e. global forest tree k corresponds to global coarse tree k).

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
If  a `cmesh` interface functions calls for a local tree id then a coarse mesh local tree id must be provided and if a `forest` interface
function calls for a local tree id then a forest local tree id must be provided.
To convert between both, use the functions `t8_forest_ltreeid_to_cmesh_ltreeid` and `t8_forest_cmesh_ltreeid_to_ltreeid`.

#### Converting from local to global

Use the following functions to convert between global and local ids:

 - `t8_forest_get_local_id` -- Converts a global id into a forest local id.
 - `t8_forest_global_tree_id` -- Converts a forest local id into a global id.
 - `t8_cmesh_get_local_id` -- Converts a global id into a cmesh local id.
 - `t8_cmesh_get_global_id` -- Converts a cmesh local id into a global id.


Since the partitioning is computed according to the space-filling curve index, the order of the trees will not change.
Thus, the global id of tree i on process p is the same as the global id of p's first tree plus i (`gid(i) = gid(0) + i`).

### Ghosts

Additionally to the local trees in the coarse mesh and forest there may also be ghost trees.
Their ids are handled in the same way as local tree ids (with `t8_locidx_t`) and most functions that accept a local tree
id as input also accept a ghost tree id.

Suppose process p has G_pf ghosts in the forest and G_pc ghosts in the coarse mesh.
Then their ghosts are enumerate 0 to G_pf - 1 and 0 to G_pc - 1. In contrast to the local trees the ghosts are not in a particular order.

If local trees and ghosts are handled together in the same context, for example by a function that accepts both as input (such as `t8_forest_global_tree_id`), then the ghost id is added to T_pf (respectively T_pc).
For example if a process has 3 local trees and 2 ghosts and we want to know the global id of the second (ghost index 1), we call t8_forest_global_tree_id with 4 as input parameter.

#### Converting from ghost id to global

Use the following functions to convert between global and ghost ids:

 - `t8_forest_ghost_get_ghost_treeid` -- Given a global tree id compute the forest ghost id.
 - `t8_forest_ghost_get_global_treeid` -- Given a forest ghost id compute the global tree id.
 - `t8_forest_global_tree_id` -- As above, use the ghost id plus T_pf as input.
 - `t8_cmesh_get_global_id` -- As above, use the ghost id plus T_pc as input.
 - `t8_cmesh_get_local_id` -- As above, returns the ghost id plus T_pc.

Attention: `t8_forest_get_local_id` will not return a ghost id. If the global input tree corresponds to a ghost tree, then the return value will be negative.





