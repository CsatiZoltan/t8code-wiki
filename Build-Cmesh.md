# Build Cmesh

In this tutorial we will learn how to create a user defined mesh.

You will find the code to this example in the `tutorials/general/t8_tutorial_build_cmesh*` files and it creates the executables `tutorials/general/t8_tutorial_build_cmesh`.

In the last tutorials we learned how to create a forest, adapt it, and how to store data. We also learned about algorithms for partitioning, balancing and creating a ghost layer. In all these previous tutorials predefined meshes were used. In this tutorial we learn how to define a user defined mesh in two- and three dimensions. In both examples we define and use different tree classes and join the different trees to create the domain. In order to be able to reflect the results, the meshes are stored in `.vtu` files.

## Steps of how to define a mesh
### 1. Definition of all vertices
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
Before creating a mesh, it has, of course, to be initialized using `t8_cmesh_init`.
   
### 3. Definition of the geometry
A cmesh can have different types of geometry which are set using the function `t8_cmesh_register_geometry`. The use of curved meshes is described in the tutorial [Feature   Curved meshes](https://github.com/DLR-AMR/t8code/wiki/Feature---Curved-meshes). In this tutorial we will use meshes with a linear geometry.
  
### 4. Definition of the classes of the different trees

`t8code` supports eight different basic tree shapes for the `cmesh`, see also `t8_eclass.h`:

| element shape | description | Number of vertices |
|---------------| ----------- | ---------------------------|
| T8_ECLASS_VERTEX | 0D points | 1 |
| T8_ECLASS_LINE | 1D lines | 2 |
| T8_ECLASS_TRIANGLE | 2D triangles | 3 |
| T8_ECLASS_QUAD | 2D quadrilaterals | 4 |
| T8_ECLASS_TET | 3D tetrahedra | 4 |
| T8_ECLASS_PYRAMID | 3D pyramids | 5 |
| T8_ECLASS_PRISM | 3D prisms | 6 |
| T8_ECLASS_HEX | 3D hexahedra | 8 |

Using the function `t8_cmesh_set_tree_class` the tree class of each tree is set.

| Parameter | Description |
|---------------| ----------- |
| cmesh | The cmesh to be updated. |
| tree_id | The global number of the tree. |
| tree_class | The element class of this tree. |

Definition of the classes of the different trees - each tree is defined by one cell

              //Class of the first tree
              t8_cmesh_set_tree_class (cmesh, 0, T8_ECLASS_[TYPE]);
              //Class of the second tree
              t8_cmesh_set_tree_class (cmesh, 1, T8_ECLASS_[TYPE]);
                    .
                    .
                    .
              //Class of the last tree
              t8_cmesh_set_tree_class (cmesh, x, T8_ECLASS_[TYPE]);
  
### 5. Classification of the vertices for each tree
Vertex IDs for the two two-dimensional trees:
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Cmesh_IDs.png" height="400">
</p>
Each tree must be assigned its vertices. This is done using `t8_cmesh_set_tree_vertices`.
It is not allowed to call this function after `t8_cmesh_commit`. The eclass of the tree has to be set before calling this function.

| Parameter | Description |
|---------------| ----------- |
| cmesh | The cmesh to be updated. |
| tree_id | The global number of the tree. |
| *vertices | Information of all vertices of the tree |
| num_vertices | Number of the vertices (related to the tree_class) |

              // Vertices of the first tree
              t8_cmesh_set_tree_vertices (cmesh, 0, [pointerToVerticesOfTreeOne], [numberOfVerticesTreeOne]);
              // Vertices of the second tree
              t8_cmesh_set_tree_vertices (cmesh, 1, [pointerToVerticesOfTreeTwo] , [numberOfVerticesTreeTwo]);
                    .
                    .
                    .
              // Vertices of the last tree
              t8_cmesh_set_tree_vertices (cmesh, x, [pointerToVerticesOfTree(x+1)] , [numberOfVerticesTree(x+1)]);
  
### 6. Definition of the face neighboors between the different trees
Edge IDs for the corresponding to the vertices can be seen in the previous figure (f_i).
In this step all connections (face neighboors) between the different trees are set using `t8_cmesh_set_join`.

| Parameter | Description |
|---------------| ----------- |
| cmesh | The cmesh to be updated. |
| tree1 | The tree id of the first of the two trees. |
| tree2 | The tree id of the second of the two trees. |
| face1 | The face number of the first tree. |
| face2 | The face number of the second tree. |
| orientation | Specify how face1 and face2 are oriented to each other |

The orientation is determined as follows.  Let my_face and other_face be the two face numbers of the connecting trees.
We chose a main_face from them as follows: Either both trees have the same element class, then the face with the lower face number is the main_face o the trees belong to different classes in which case the face belonging to the tree with the lower class according to the ordering
  `triangle < square, hex < tet < prism < pyramid`,
is the main_face.
Then face corner 0 of the main_face connects to a face corner k in the other face.  The face orientation is defined as the number k.

              // List of all face neighboor connections
              t8_cmesh_set_join (cmesh, [treeId1], [treeId2], [faceIdInTree1], [faceIdInTree2], [orientation]);
              t8_cmesh_set_join (cmesh, [treeId1], [treeId2], [faceIdInTree1], [faceIdInTree2], [orientation]);
                    .
                    .
                    .
              t8_cmesh_set_join (cmesh, [treeId1], [treeId2], [faceIdInTree1], [faceIdInTree2], [orientation]);
   
### 7. Commit the mesh
The last step of creating a user defined mesh is commiting the mesh using `t8_cmesh_commit`.

## 2D Example
In this two dimensional example four triangles and two quads are used. We will look at the following example. In the left you can see the order of the vertices and in the right the edge IDs.
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step8_2D_Vertex_Edge_Id.PNG" height="400">
</p> 
The vertices of the trees have the following coordinate:

| tree | vertices |
|-------------------| ----------- |
| triangle 1 | {(0, 0, 0), (0.5, 0, 0), (0.5, 0.5, 0)} |
| triangle 2 | {(0, 0, 0), (0.5, 0.5, 0), (0, 0.5, 0)} |
| triangle 3 | {(0.5, 0.5, 0), (1, 0.5, 0), (1, 1, 0)} |
| triangle 4 | {(0.5, 0.5, 0), (1, 1, 0), (0.5, 1, 0)} |
| quad 1 | {(0.5, 0, 0), (1, 0, 0), (0.5, 0.5, 0), (1, 0.5, 0)} |
| quad 2 | {(0, 0.5, 0), (0.5, 0.5, 0), (0, 1, 0), (0.5, 1, 0)} |

The tree class for the triangles is `T8_ECLASS_TRIANGLE` and this for the quad is `T8_ECLASS_QUAD`:

              // definition of the tree classes (you need one classification for each tree)
              t8_cmesh_set_tree_class (cmesh, [treeID], T8_ECLASS_TRIANGLE);
              t8_cmesh_set_tree_class (cmesh, [treeID], T8_ECLASS_QUAD);

Each edge of the tree has an ID. The IDs for this example can be seen in the figure. For the direct neighboor information, the following trees are connected:
| ID of first tree | ID of second tree | ID of face (first tree) | ID of face (second tree) |
|---------------| ----------- |---------------| ----------- |
| 0 | 1 | 1 | 2 |
| 0 | 2 | 0 | 0 |
| 1 | 3 | 0 | 2 |
| 2 | 4 | 3 | 2 |
| 3 | 5 | 1 | 1 |
| 4 | 5 | 1 | 2 |

              // definition of the face neighboors
              t8_cmesh_set_join (cmesh, 0, 1, 1, 2, 0);
              t8_cmesh_set_join (cmesh, 0, 2, 0, 0, 0);
              t8_cmesh_set_join (cmesh, 1, 3, 0, 2, 1); 
              t8_cmesh_set_join (cmesh, 2, 4, 3, 2, 0);
              t8_cmesh_set_join (cmesh, 3, 5, 1, 1, 0);
              t8_cmesh_set_join (cmesh, 4, 5, 1, 2, 0);

As this cmesh has periodic boundaries, there are also the connections
| ID of first tree | ID of second tree | ID of face of first tree | ID of face of second tree |
|---------------| ----------- |---------------| ----------- |
| 0 | 3 | 2 | 3 |
| 1 | 2 | 1 | 1 |
| 2 | 5 | 2 | 0 |
| 3 | 4 | 0 | 0 |

              // definition of the face neighboors for the periodic boundaries
              t8_cmesh_set_join (cmesh, 0, 3, 2, 3, 0); 
              t8_cmesh_set_join (cmesh, 1, 2, 1, 1, 0);
              t8_cmesh_set_join (cmesh, 2, 5, 2, 0, 1);
              t8_cmesh_set_join (cmesh, 3, 4, 0, 0, 0);

## 3D Example
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step8_3DMesh.PNG" height="400">
</p>

In this three dimensional example two tetrahedra, two prisms, one pyramid, and one hexahedron is used. We will look at the following example. In the left you can see the order of the vertices and in the right the edge IDs.
<p align="center">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step8_3D_Mesh_Vertex_Id.PNG" height="400">
<img src="https://github.com/DLR-AMR/t8code/wiki/pictures/tutorials/Step8_3D_Mesh_Face_Id.png" height="400">
</p> 
The vertices of the trees have the following coordinate:

| tree | vertices |
|---------------| ----------- |
| tetrahedron 1 | {(0.43, 0, 2), (0, 0, 1), (0.86, -0.5, 1), (0.86, 0.5, 1)} |
| tetrahedron 2 | {(2.29, 0, 2), (1.86, -0.5, 1), (2.72, 0, 1), (1.86, 0.5, 1)} |
| prism 1 | {(0, 0, 0), (0.86, -0.5, 0), (0.86, 0.5, 0), (0, 0, 1), (0.86, -0.5, 1), (0.86, 0.5, 1)} |
| prism 2 | {(1.86, -0.5, 0), (2.72, 0, 0), (1.86, 0.5, 0), (1.86, -0.5, 1), (2.72, 0, 1), (1.86, 0.5, 1)} |
| pyramid | {(0.86, 0.5, 0), (1.86, 0.5, 0), (0.86, -0.5, 0), (1.86, -0.5, 0), (1.36, 0, -0.5)} |
| hexahedron | {(0.86, -0.5, 0), (1.86, -0.5, 0), (0.86, 0.5, 0), (1.86, 0.5, 0), (0.86, -0.5, 1), (1.86, -0.5, 1),(0.86, 0.5, 1), (1.86, 0.5, 1)} |

The tree class for the tetrahedra is `T8_ECLASS_TET`, this for the prisms is `T8_ECLASS_PRISM`, and this for the hexahedron is `T8_ECLASS_HEX`:

              // definition of the tree classes (you need one classification for each tree)
                t8_cmesh_set_tree_class (cmesh,  [treeID], T8_ECLASS_TET);
                t8_cmesh_set_tree_class (cmesh,  [treeID], T8_ECLASS_PRISM);
                t8_cmesh_set_tree_class (cmesh,  [treeID], T8_ECLASS_PYRAMID);
                t8_cmesh_set_tree_class (cmesh,  [treeID], T8_ECLASS_HEX);

As the mesh has no periodic boundaries, there are only direct neighbors. These are encoded by the face connections between the trees:
| ID of first tree | ID of second tree | ID of face (first tree) | ID of face (second tree) |
|---------------| ----------- |---------------| ----------- |
| 0 | 2 | 0 | 4 |
| 1 | 3 | 4 | 4 |
| 2 | 5 | 0 | 0 |
| 3 | 5 | 1 | 1 |
| 4 | 5 | 4 | 4 |

              // definition of the face neighboors
                t8_cmesh_set_join (cmesh, 0, 2, 0, 4, 0);
                t8_cmesh_set_join (cmesh, 1, 3, 0, 4, 0);
                t8_cmesh_set_join (cmesh, 2, 5, 0, 0, 0);
                t8_cmesh_set_join (cmesh, 3, 5, 1, 1, 0);
                t8_cmesh_set_join (cmesh, 4, 5, 4, 4, 0);
  