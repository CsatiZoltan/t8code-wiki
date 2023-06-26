# Tutorial overview

There are two kinds of tutorials for `t8code`: The general tutorials, which lead you step by step and show the basic usage of `t8code`.  
Furthermore, there are feature tutorials, which detail on more advanced or additional features of `t8code`.
You will find the code for these tutorials in the `tutorials/` folder of `t8code`.

## General

 - [Step 0   Hello World](https://github.com/holke/t8code/wiki/Step-0---Hello-World) - Setting up an application and using `t8code`'s logging function.

 - [Step 1   Creating a coarse mesh](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh) - 
Building a simple coarse mesh.

 - [Step 2   Creating a uniform forest](https://github.com/holke/t8code/wiki/Step-2---Creating-a-uniform-forest) - 
Constructing a uniform forest on top of a coarse mesh.

 - [Step 3   Adapting a forest](https://github.com/holke/t8code/wiki/Step-3---Adapting-a-forest) - 
How to refine and coarsen a forest.

 - [Step 4   Partition,-Balance,-Ghost](https://github.com/holke/t8code/wiki/Step-4---Partition,-Balance,-Ghost) - 
Modifying the forest further by partitioning, balancing and computing ghost elements.

 - [Step 5   Store element data](https://github.com/holke/t8code/wiki/Step-5---Store-element-data) - 
How to gather data of the local elements, exchange it and output it to a `.vtu` file.  

 - [Step 6   Computing stencils](https://github.com/DLR-AMR/t8code/wiki/Step-6-Computing-stencils) - 
How to gather data from element's face neighbors and collect stencils in, e.g., finite difference computations. 

 - [Step 7   Interpolation](https://github.com/DLR-AMR/t8code/wiki/Step-7-Interpolation) - 
How to interpolate data on an adapted forest.

 - [Build cmesh](https://github.com/DLR-AMR/t8code/wiki/Build-Cmesh) - 
A tutorial of how to create a cmesh in 2D and in 3D.

 - [Search algorithm](https://github.com/holke/t8code/wiki/Tutorial:-Search) - 
A tutorial for the hierarchical search to identify elements matching a user defined criterion.

## Features

 - [Feature   Curved meshes](https://github.com/DLR-AMR/t8code/wiki/Feature---Curved-meshes) - 
A tutorial about the generation of curved adaptive meshes.