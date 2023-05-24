When a forest with corresponding cell-data is adapted, the data has to be interpolated to map it on the adapted forest.

In this example we describe how to interpolate the data on a forest.

You will find the code to this example in the tutorials/general/t8_step7* files and it creates the executable tutorials/general/t8_step7_interpolation.

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

now actual adaption and interpolation


As in ([Step 5](https://github.com/DLR-AMR/t8code/wiki/Step-5---Store-element-data)) the refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point (0.5, 0.5, 1) and we will coarsen elements if they are outside a radius of 0.4.

    interpolation - forest_replace 
                    alter forest, neuer forest - -1,1,0
                    Verfeinern - Daten aus grobem Gitter aufs feine Gitter
                    Vergröbern - Mittelwert der Einzelnen Zellwerte
                    0 - Wert übernehmen