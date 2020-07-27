## Step 3 - Adapting a forest

Adaptation is the process of refining and coarsening the elements in a forest according to
a given criterion.

In this example we describe how to define such a criterion and how to adapt an existing forest.

You will find the code to this example in `example/tutorials/t8_step3_adapt_forest.cxx` and it creates the executable `example/tutorials/t8_step3_adapt_forest`.

As in the previous examples ([Step 1](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh) and [Step 2](https://github.com/holke/t8code/wiki/Step-2---Creating-a-uniform-forest)) we will use a cube geometry for our coarse mesh, but this time modelled as a hybrid
mesh with different element types (tet, prism and hex).

The refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point `(0.5, 0.5, 1)` and we will coarsen elements if they are outside a radius of 0.4.

We start with a uniform forest as in step 2 and adapt once, thus refining or coarsening by one level.


<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_uniformForest_lvlcolor.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_adaptForest_lvlcolor.png" height="350">
</p>

The uniform level 3 forest (left) will get adapted and then also have level 4 and level 2 elements (right).
The colors correspond to the refinement levels.