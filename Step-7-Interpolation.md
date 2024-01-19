When a forest with corresponding cell-data is adapted, the data has to be interpolated to map it on the adapted forest.

In this example we describe how to interpolate the data on a forest.

You will find the code to this example in the `tutorials/general/t8_step7*` files and it creates the executable `tutorials/general/t8_step7_interpolation`.

In this example we create a forest with one tree. If you create a forest with more than one tree you may have to create ghost data for the communication. How to do this is described in ([Step 4](https://github.com/DLR-AMR/t8code/wiki/Step-4---Partition,-Balance,-Ghost)).

## Create a forest with corresponding cell data
As in the previous examples ([Step 3](https://github.com/DLR-AMR/t8code/wiki/Step-3---Adapting-a-forest)) we will use a cube geometry for our coarse mesh. In this example we use a forest consisting of one tree.
The forest is adapted uniformly in a first step.

We store the distance of each cell to the point (0.5, 0.5, 1) on the forest. In ([Step 5](https://github.com/DLR-AMR/t8code/wiki/Step-5---Store-element-data)) it is described how to store data on a forest.
It is important to mention again, that an array is created analogue to the forest elements sorted by the space filling curve. To store the distance we iterate over all trees in the forest and then over all elements of each local tree in the local forest. 
The data elements are stored in a `sc_array`. The data array is independent of the trees. Thus, the index has to be incremented for each element independent of the tree.
```C++
       for (itree = 0, ielem = 0; itree < num_trees; itree++) {
         const t8_locidx_t   num_elem =
           t8_forest_get_tree_num_elements (forest, itree);
         /* Inner loop: Iteration over the elements of the local tree */

         for (t8_locidx_t ielem_tree = 0; ielem_tree < num_elem; ielem_tree++, ielem++) {
           /* To calculate the distance to the centroid of an element the element is saved */
           const t8_element_t *element =
             t8_forest_get_element_in_tree (forest, itree, ielem_tree);

           /* Get the centroid of the local element. */
           t8_forest_element_centroid (forest, itree, element, centroid);

           /* Calculation of the distance to the centroid for the referenced element */
           elem_data->values = t8_vec_dist (centroid, midpoint);

           t8_element_set_element (data, ielem, *elem_data);
         }
       }
```
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step7_uniform.PNG" height="400">
</p>

## Interpolation
Now we can start to adapt the forest with corresponding cell data elements. Therefore, we have to execute two steps. In a first step build a second forest to store the adapted forest `forest_new` and keep the old forest `forest_old`. 

As in ([Step 3](https://github.com/DLR-AMR/t8code/wiki/Step-3---Adapting-a-forest)) the refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point (0.5, 0.5, 1) and we will coarsen elements if they are outside a radius of 0.4.

Note, that if we want to interpolate the data, a non-recursive adaptation of the forest is restricted. Hence, all elements in the new forest `forest_new` result from an element in `forest_old` by either refining once, coarsening once, or keeping the element as it is. Thus, the difference in level is at most 1.

With the two forests given, the interpolation of the data can start. These two forests are compared element wise and in each comparison a callback function is called. This callback function providing the local indices of the old and new elements as well as the refinement (if the new element is the child, parent or the the same).
This is done using `t8_forest_iterate_replace`:
```C++
     void                t8_forest_iterate_replace (t8_forest_t forest_new,
                                                    t8_forest_t forest_old,
                                                    t8_forest_replace_t replace_fn);
```

| Parameter | Description |
|-|-|
| forest_new | The new (adapted) forest  |
| forest_old | The old (not adapted) forest |
| replace_fn | function to define how to replace the cell elements |

To call this function, it has to be defined how to replace the cell elements of the old forest for the new forest. Therefore, the callback function `t8_forest_replace` is defined. Outgoing are the old elements and incoming the new ones.

```C++
     void
     t8_forest_replace (t8_forest_t forest_old,
                        t8_forest_t forest_new,
                        t8_locidx_t which_tree,
                        t8_eclass_scheme_c *ts,
                        int refine,
                        int num_outgoing,
                        t8_locidx_t first_outgoing,
                        int num_incoming, t8_locidx_t first_incoming)
```

| Parameter | Description |
|-|-|
| forest_old | old (not adapted) forest |
| forest_new | new (adapted) forest  |
| which_tree | local tree_id in the old and new forests |
| ts | element class scheme  |
| refine | == 0 : copy, == -1 : coarsen, == 1 : refine |
| num_outgoing | number of elements of the old forest |
| first_outgoing | index of the first element in the old forest  |
| num_incoming | number of elements of the new forest |
| first_incoming | index of the first element in the new forest |

In this example we use the following criteria:
If an element is refined, each child gets the value of its parent. If elements are coarsened, the parent gets the average value of the children. If an element is unchanged, we also do not change the stored value.
```C++
     /* Do not adapt or coarsen */
     if (refine == 0) {
         t8_element_set_element (adapt_data_new, first_incoming,
                                 t8_element_get_value (adapt_data_old,
                                                       first_outgoing));
     }
     /* The old element is refined, we copy the element values */
     else if (refine == 1) {
         for (int i = 0; i < num_incoming; i++) {
           t8_element_set_element (adapt_data_new, first_incoming + i,
                                   t8_element_get_value (adapt_data_old,
                                                         first_outgoing));
         }
     }
     /* Old element is coarsened */
     else if (refine == -1) {
         double              tmpValue = 0;
         for (t8_locidx_t i = 0; i < num_outgoing; i++) {
           tmpValue +=
             t8_element_get_value (adapt_data_old, first_outgoing + i).values;
         }
         t8_element_set_value (adapt_data_new, first_incoming,
                               tmpValue / num_outgoing);
     }
     t8_forest_set_user_data (forest_new, adapt_data_new);
```
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step7_adapted.PNG" height="400">
</p>

After this interpolation step, we can set the adapted forest `forest_new` as `forest` and keep on working with this forest (for example by calculating a next time step).
