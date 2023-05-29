When a forest with corresponding cell-data is adapted, the data has to be interpolated to map it on the adapted forest.

In this example we describe how to interpolate the data on a forest.

You will find the code to this example in the tutorials/general/t8_step7* files and it creates the executable tutorials/general/t8_step7_interpolation.

## Create a forest with corresponding cell data
As in the previous examples ([Step 3](https://github.com/DLR-AMR/t8code/wiki/Step-3---Adapting-a-forest)) we will use a cube geometry for our coarse mesh. In this example we use a forest consisting of one tree.
The forest is adapted uniformly in a first step.

We store the distance of each cell to the point (0.5, 0.5, 1) on the forest. In ([Step 5](https://github.com/DLR-AMR/t8code/wiki/Step-5---Store-element-data)) it is described how to store data on a forest.
It is important to mention again, that an array is created analogue to the forest elements sorted by the space filling curve. To store the distance we iterate over all trees in the forest and then over all elements of the local forest. 
The data elements are stored in a sc_array. The data array is independent of the trees. Thus, the index has to be incremented for each element independent of the tree.

       const t8_locidx_t numTrees = t8_forest_get_num_local_trees (forest);

       for (int itree = 0, int ielem = 0; itree < numTrees; itree++) {
         const t8_locidx_t numElem = t8_forest_get_tree_num_elements (forest, itree);

         /* Inner loop: Iteration over the elements of the local tree */
         for (t8_locidx_t ielemTree = 0; ielemTree < numElem; ielemTree++, ielem++) {
           /* To calculate the distance to the centroid of an element the element is saved */
           const t8_element_t *element = t8_forest_get_element_in_tree (forest, itree, ielemTree);

           /* Get the centroid of the local element. */
           t8_forest_element_centroid (forest, 0, element, centroid);

           /* Calculation of the distance to the centroid for the referenced element */
           data[ielem].values = t8_vec_dist (centroid, midpoint);
         }
       }

<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step7_uniform.PNG" height="400">
</p>

## Interpolation
Now we can start to adapt the forest with corresponding cell data elements. Therefore, we have to execute two steps. In a first step build a second forest to store the adapted forest `adapt_forest` and keep the old forest `forest`. 

As in ([Step 3](https://github.com/DLR-AMR/t8code/wiki/Step-3---Adapting-a-forest)) the refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point (0.5, 0.5, 1) and we will coarsen elements if they are outside a radius of 0.4.

Now the actual interpolation can start. Therefore, for each cell of the old forest it is 
Therefore, two forest are given. In one forest the elements are either direct children or parents of the elements in the other forest. These two forests are compared for each refined element or coarsened family in the old forest. Call a callback function providing the local indices of the old and new elements.
This is done using `t8_forest_iterate_replace`:

     void                t8_forest_iterate_replace (t8_forest_t forest_new,
                                                    t8_forest_t forest_old,
                                                    t8_forest_replace_t
                                                    replace_fn);

| Parameter | Description |
|-|-|
| forest_new | The new (adapted) forest  |
| forest_old | The old (not adapted) forest |
| replace_fn | function to define how to replace the cell elements |

To call this function, it has to be defined how to replace the cell elements of the old forest for the new forest. Therefore, the function `t8_forest_replace` is defined. Outgoing are the old elements and incoming the new ones.

     void
     t8_forest_replace (t8_forest_t forest_old,
                        t8_forest_t forest_new,
                        t8_locidx_t which_tree,
                        t8_eclass_scheme_c *ts,
                        int refine,
                        int num_outgoing,
                        t8_locidx_t first_outgoing,
                        int num_incoming, t8_locidx_t first_incoming)

| Parameter | Description |
|-|-|
| forest_old | The old (not adapted) forest |
| forest_new | The new (adapted) forest  |
| which_tree | tree_id of the analyzed element |
| ts | eclass sheme  |
| refine | ==0 - do nothing, == -1 - coarsen, == 1 - refine |
| num_outgoing | number of the elements not refined forest |
| first_outgoing | eclass sheme  |
| num_ingoing | number of the elements corresponding to the element of the not refined forest |
| first_incoming | index of the new element |

In this example we use the following criteria:
If an element is refined, each child gets the value of its parent. If elements are coarsened, the parent gets the average value of the children.

     /* Do not adapt or coarsen */
       if (refine == 0) {
         adapt_data_new->element_data[first_incoming] =
           adapt_data_old->element_data[first_outgoing];
       }
       /* The old element is refined, we copy the element values */
       else if (refine == 1) {
         for (int i = 0; i < num_incoming; i++) {
           adapt_data_new->element_data[first_incoming + i] =
             adapt_data_old->element_data[first_outgoing];
         }
       }
       /* Old element is coarsened */
       else if (refine == -1) {
         adapt_data_new->element_data[first_incoming].values = 0;
         for (t8_locidx_t i = 0; i < num_outgoing; i++) {
           adapt_data_old->element_data[first_outgoing + i].values;
           adapt_data_new->element_data[first_incoming].values +=
             adapt_data_old->element_data[first_outgoing + i].values;
         }
         adapt_data_new->element_data[first_incoming].values /= num_outgoing;
       }

<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step7_adapted.PNG" height="400">
</p>
