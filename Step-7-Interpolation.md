When a forest with corresponding cell-data is adapted, the data has to be interpolated to map it on the adapted forest.

In this example we describe how to interpolate the data on a forest.

You will find the code to this example in the tutorials/general/t8_step7* files and it creates the executable tutorials/general/t8_step7_interpolation.

As in the previous examples ([Step 3](https://github.com/DLR-AMR/t8code/wiki/Step-3---Adapting-a-forest)  we will use a cube geometry for our coarse mesh. 

As in [Step 5](https://github.com/DLR-AMR/t8code/wiki/Step-5---Store-element-data)) the refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point (0.5, 0.5, 1) and we will coarsen elements if they are outside a radius of 0.4.


Create a cmesh

Create a forest

uniform adaption

data on the uniform adapted forest

now actual adaption and interpolation

    adapt forest like in step 3?

    interpolation - forest_replace 
                    alter forest, neuer forest - -1,1,0
                    Verfeinern - Daten aus grobem Gitter aufs feine Gitter
                    Vergröbern - Mittelwert der Einzelnen Zellwerte
                    0 - Wert übernehmen