# Step 8 - User defined mesh

In this tutorial we will learn how to create a user defined mesh.

You will find the code to this example in the `tutorials/general/step8*` files and it creates the executables `tutorials/general/t8_step8_user_defined_mesh`.

In the last tutorials we learned how to create a forest, adapt it, and how to store data. We also learned about algorithms for partitioning, balancing and creating a ghost layer. In all these previous tutorials predefined meshes were used. In this tutorial we learn how to define a user defined mesh in two- and three dimensions. In both examples we define and use different tree classes and join the different trees to create the domain. In order to be able to reflect the results, the meshes are stored in `.vtu` files.

## Different tree classes

## Steps of how to define a mesh
### 1. Defining an array with all vertices
In a first step an array with all is defined. Independent of the fact if a mesh is defined two- or three dimensional, each point is defined by three coordinates. The vertices are ordered in a listing of points for each cell. Thus, there can be duplicates in the list.

             double vertices[numberOfValues] = {

                //point values for tree 1
                x_1,y_1,z_1         //(x,y,z) of first point of tree 1
                x_2,y_2,z_2         //(x,y,z) of second point of tree 1
                    .
                    .
                    .
                x_n,y_n,z_n         //(x,y,z) of nth point (last point) of tree 1

                //point values for tree 2
                x_1,y_1,z_1         //(x,y,z) of first point of tree 2
                x_2,y_2,z_2         //(x,y,z) of second point of tree 2
                    .
                    .
                    .
                x_m,y_m,z_m         //(x,y,z) of nth point (last point) of tree 2

                    .
                    .
                    .

                //point values for the last tree
                x_1,y_1,z_1         //(x,y,z) of first point of the last tree
                x_2,y_2,z_2         //(x,y,z) of second point of the last tree
                    .
                    .
                    .
                x_o,y_o,z_o         //(x,y,z) of nth point (last point) of the last tree
              };
   
### 2. Initialization of the mesh
   
### 3. Definition of the geometry
  
### 4. Definition of the classes of the different trees
  
### 5. Classification of the vertices for each tree
  
### 6. Definition of the face neighboors between the different trees
   
### 7. Commit the mesh

## 2D Example

## 3D Example
  