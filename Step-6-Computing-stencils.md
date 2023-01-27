## Step 6 - Computing stencils

In this tutorial we will learn how to gather a stencil consisting of data from the current element and its face neighbors with special emphasis on exchanging data over the ghost elements beforehand.

You will find the code to this example in the `tutorials/general/step6*` files and it creates the executables `tutorials/general/t8_step6_stencil`. 

In the last tutorials we learned how to create a forest, adapt it, pre-allocate element data arrays and store custom data fields in VTU files. In this tutorial we will start by performing all these operations in one step as shown in [step 5](https://github.com/DLR-AMR/t8code/wiki/Step-5---Store-element-data). Then, when we have our forest and built a data array, we gather data for the local elements of our process. Next, we exchange the data values of the ghost elements and compute a stencil from face neighbors in each element. Finally, the output of several custom data fields is written to `.vtu` files.

### Computing Finite Differences from Face Neighbor Stencils

For many scientific and industrial applications the access to neighboring data of an element is a crucial feature every mesh library has to provide. The collection of the element's data and its neighbors is called stencil. Stencils can be seen as a compact data structure which allows
for convenient and performant finite difference computations yielding numerical approximations of gradients, curvatures, curls, etc. of a data field. An exemplary visualization of a 3x3 cross stencil is shown in the figure below. The uniform grid is divided into three parallelization zones with the already introduced ghost layers at their interfaces.

![grafik](https://user-images.githubusercontent.com/10619309/215130819-29c92c61-9489-4ce3-b6bf-364a8467d3e8.png)

The data field in our specific case is the physical quantitiy "height" pinned at the midpoint of each mesh element. The actual demo program
to this tutorial also supports adapted meshes with neighboring elements at different refinement levels.

### Accessing Element Face Neighbors in t8code

t8code provides an API function to access neighboring elements at a given face.
The excerpt from `t8_forest.h` reads:
```C++
/** Compute the leaf face neighbors of a forest.
 * \param [in]    forest  The forest. Must have a valid ghost layer.
 * \param [in]    ltreeid A local tree id.
 * \param [in]    leaf    A leaf in tree \a ltreeid of \a forest.
 * \param [out]   neighbor_leafs Unallocated on input. On output the neighbor
 *                        leafs are stored here.
 * \param [in]    face    The index of the face across which the face neighbors
 *                        are searched.
 * \param [out]   dual_face On output the face id's of the neighboring elements' faces.
 * \param [out]   num_neighbors On output the number of neighbor leafs.
 * \param [out]   pelement_indices Unallocated on input. On output the element indices
 *                        of the neighbor leafs are stored here.
 *                        0, 1, ... num_local_el - 1 for local leafs and
 *                        num_local_el , ... , num_local_el + num_ghosts - 1 for ghosts.
 * \param [out]   pneigh_scheme On output the eclass scheme of the neighbor elements.
 * \param [in]    forest_is_balanced True if we know that \a forest is balanced, false
 *                        otherwise.
 * \note If there are no face neighbors, then *neighbor_leafs = NULL, num_neighbors = 0,
 * and *pelement_indices = NULL on output.
 * \note Currently \a forest must be balanced.
 * \note \a forest must be committed before calling this function.
 */
void                t8_forest_leaf_face_neighbors (t8_forest_t forest,
                                                   t8_locidx_t ltreeid,
                                                   const t8_element_t *leaf,
                                                   t8_element_t
                                                   **pneighbor_leafs[],
                                                   int face,
                                                   int *dual_faces[],
                                                   int *num_neighbors,
                                                   t8_locidx_t
                                                   **pelement_indices,
                                                   t8_eclass_scheme_c
                                                   **pneigh_scheme,
                                                   int forest_is_balanced);
```
A common usage pattern of above routine can be found in the function body of
`t8_step6_compute_stencil` in the demo source file `tutorials/general/t8_step6_stencil.cxx`.

### Running the Demo
If t8code was compiled with MPI support the demo can be exectuted with for example three process.
```shell
$ mpirun -n 3 ${INSTALL_DIR_OF_T8CODE}/bin/t8_step6_stencil
```
This results in three VTU files and a meta PVTU file.
```
$ ls
t8_step6_stencil.pvtu  t8_step6_stencil_0000.vtu  t8_step6_stencil_0001.vtu  t8_step6_stencil_0002.vtu
```
Open `t8_step6_stencil.pvtu` in Paraview and visualize the `height` (first figure) and `schlieren` (second figure) data fields. 
[Schlieren](https://en.wikipedia.org/wiki/Schlieren_photography) plots are the norm of the gradient of a data field (in our case the `height`). For smoother pictures the inclined reader is encouraged to increase the initial refinenemnt level, recompile and rerun the demo.

![grafik](https://user-images.githubusercontent.com/10619309/215139981-f636c5a9-8d2b-414e-9a93-39011367a760.png)
![grafik](https://user-images.githubusercontent.com/10619309/215141420-d89d3b53-3ff0-41a1-8256-a8b08f92e5bf.png)
