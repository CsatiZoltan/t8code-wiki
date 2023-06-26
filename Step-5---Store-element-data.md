## Step 5 - Store element data

In this tutorial we will learn how to associate custom data with the elements of a forest and how to exchange data over the ghost elements of a parallel partition.

You will find the code to this example in the `tutorials/general/step5*` files and it creates the executables `tutorials/general/t8_step5_element_data` and `tutorials/general/t8_step5_element_data_c_interface`. The latter is an implementation of the example using our pure `C` interface.

In the last tutorials we learned how to create a forest and adapt it. In [step 4](https://github.com/holke/t8code/wiki/Step-4---Partition,-Balance,-Ghost) we also learned about algorithms for partitioning, balancing and creating a ghost layer. In this tutorial we will start by performing all these operations in one step. Then, when we have our forest, we will continue with how to build a data array and gather data for the local elements of our process. Further we exchange the data values of the ghost elements and output the volume data to our `.vtu` file.  

### Adapt, Partition, Balance, Ghost

As announced in [step4 - order of execution](https://github.com/holke/t8code/wiki/Step-4---Partition,-Balance,-Ghost#order-of-execution), it is possible to mix the `t8_forest_set*` functions together as you wish (see `t8_forest.h`).
We can thus in one step adapt a forest, balance it, partition it and create a ghost layer with

```C++
t8_forest_t new_forest;
t8_forest_init (&new_forest);
t8_forest_set_user_data (new_forest, &data);
t8_forest_set_adapt (new_forest, forest, CALLBACK, recursive_flag);
t8_forest_set_partition (new_forest, forest, 0);
t8_forest_set_balance (new_forest, forest, no_partition_flag);
t8_forest_set_ghost (new_forest, 1, T8_GHOST_FACES);
t8_forest_commit (new_forest);
```

The order in which the `t8_forest_set*` functions are called is not relevant. 

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step5_apbg_4ranks.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step5_apbg_level.png" height="350">
</p>

Let us execute the example with 4 MPI processes.

```bash
mpirun -np 4 ./t8_step5_element_data
```
As expected, the resulting forest looks like the adapted forest from [step 4](https://github.com/holke/t8code/wiki/Step-4---Partition,-Balance,-Ghost) (executed with 4 MPI processes) before it gets unbalanced. 

### Build data array and gather data for the local elements

To build a data array, we first nead to allocate memory at run time, which should happen via `T8_ALLOC`. The syntax of `T8_ALLOC` is similar to `malloc`. It requires a datatype and size to allocate the necessary memory. In our case we want to store the level and volume of each element. To do so, we create a struct with `level` and `volume` variables inside.

```C++
struct t8_step5_data_per_element
{
  int                 level;
  double              volume;
};
```
Since we a have activated ghost layer, the required size is the number of local elements plus the number of ghost elements. A look into `t8_forest.h` shows, that there are two handy functions.

```C++
num_local_elements = t8_forest_get_local_num_elements (forest);
num_ghost_elements = t8_forest_get_num_ghosts (forest);
```

Having all requirements set, we can now allocate the memory.

```C++
element_data = T8_ALLOC (struct t8_step5_data_per_element, num_local_elements + num_ghost_elements);
```

In the latter case you need to use `T8_FREE` in order to free the memory as you would do with `malloc`.

Let us now fill the data with something. For this, we iterate through all trees and for each tree through all its elements, calling `t8_forest_get_element_in_tree` to get a pointer to the current element.
Note, that this is the recommended and most performant way. An alternative is to iterate over the number of local elements and use `t8_forest_get_element`. However, this function needs to perform a binary search for the element and the tree it is in, while `t8_forest_get_element_in_tree` has a constant look up time. You should only use `t8_forest_get_element` if you do not know in which tree an element is.

The reason for the iteration through the trees and then through its elements is, that each tree may have a different element class and therefore also a different way to interpret its elements. You used different element classes allrady in [step 1 - Creating a simple coarse mesh](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh#creating-a-simple-coarse-mesh). In `t8_eclass.h` you will find all possible element classes (i.e. triangle, tetrahedron, square, etc.). You also may remember seeing or even using the `eclass_scheme` in [step 3 - The adaptation callback](https://github.com/holke/t8code/wiki/Step-3---Adapting-a-forest#the-adaptation-callback) before. The `eclass_scheme` was a parameter of the `t8_forest_adapt_t` callback function. In order to be able to handle elements of a tree, we need to get its `eclass_scheme`.
The `eclass_scheme` stores for each element shape, one member of `type t8_eclass_scheme_c` which provides the necessary functions for this element shape. We could for example use `t8_element_level` to compute the element's level.
Finally, we can now fill the first `num_local_elements` entries of our array.

### Exchange the data values of the ghost elements

Until now, each process has computed the data entries for its local elements. In order to get the values for the ghost elements, we use `t8_forest_ghost_exchange_data`. Calling this function will fill all the ghost entries of our element data array with the value on the process that owns the corresponding element. This means that we don't have to iterate over all the elements again. t8code takes care of exchanging the data for us.

However, we have to do a little preparatory work,  since `t8_forest_ghost_exchange_data` expects an `sc_array`. Therefore we wrap our data array to an `sc_array`.

```C++
sc_array_wrapper = sc_array_new_data (element_data, sizeof(struct t8_step5_data_per_element), num_local_elements + num_ghost_elements);
```

Carry out the data exchange.

```C++
t8_forest_ghost_exchange_data (forest, sc_array_wrapper);
```
Note that this function does not need an index. A request regarding the number of local elements on the respective process takes place in the background. Thus, all entries with an index greater than `num_local_elements` will be overwritten.

Destroy the wrapper array by calling `sc_array_destroy`. This will not free the data memory since we used `sc_array_new_data`.

### Output the volume data to vtu

After creating the `forest` and storing elements' volumes and levels in an array we will write the `forest` and the elements' volumes out to `.vtu` files in order to view it in `Paraview`. 

In the last steps we just called the following function 

```C++
t8_forest_write_vtk (forest, prefix);
```
to write a forest as `vtu`. But to write also user defined data, we need an extended output function `t8_forest_vtk_write_file`.

t8code supports writing element based data to `vtu` as long as its stored as `doubles`. Each of the data fields to write has to be provided in its own array of length `num_local_elements`. We support two types: 

| vtk variable type | double per element |
| - | - |
| T8_VTK_SCALAR | 1 |
| T8_VTK_VECTOR | 3 |

Therefore we need to allocate a new array to store the volumes on their own. This array has one entry per local element.

```C++
double *element_volumes = T8_ALLOC (double, num_local_elements);
```

Despite writing user data, `t8_forest_vtk_write_file` also offers more control over which properties of the forest to write. 

| Parameter | Description |
|-|-|
| forest | The forest to write |
| fileprefix | The prefix of the files |
| write_treeid | If true, the global tree id is written for each element |
| write_mpirank | If true, the mpirank is written for each element |
| write_level | If true, the refinement level is written for each element |
| write_element_id | If true, the global element id is written for each element |
| write_ghosts | If true, each process additionally writes its ghost elements |
| curved_flag | If true, write the elements as curved element types |
| do_not_use_API | Do not use the VTK API, even if linked and available |
| num_data | Number of user defined double valued data fields to write |
| data | Array of `t8_vtk_data_field_t` of length `num_data` |

For more documentation take a look into `t8_forest.h`.
Note, you can store even more data to the `vtu` file. The number of user defined data fields is defined by the parameter `num_data`. For each user defined data field we need one `t8_vtk_data_field_t` variable. In our case, it is only one.

```C++
t8_vtk_data_field_t vtk_data;
```

Next, set the type of this variable. Since we have one value per element, we pick `T8_VTK_SCALAR`

```C++
vtk_data.type = T8_VTK_SCALAR;
```

Also the name of the field as should be written to the file. 

```C++
strcpy (vtk_data.description, "Element volume");
```

Finally copy the elements' volumes from our data array to the output array and pass it to `t8_forest_write_vtk_ext`.

We decided to only set `write_treeid`, `write_mpirank`, `write_level` and `write_element_id` true as it would be in `t8_forest_write_vtk`. In this way, the only difference to `t8_forest_vtk_write_file` is the addition of the volume of the elements.

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step5_volume.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step5_volume_clip_threshold.png" height="350">
</p>

In `Paraview` we can now display the volumes of the elements and work with them.