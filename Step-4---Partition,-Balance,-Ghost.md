## Step 4 - Partition, Balance, Ghost

In this tutorial we get to know the other main forest manipulation algorithms besides adapt (See [step 3](https://github.com/holke/t8code/wiki/Step-3---Adapting-a-forest)).
Additionally we learn more about the forest creation process and how we can perform multiple of these algorithms at the same time (i.e. on the same forest).

We will start with the adapted forest from step 3.

### Partition

Each forest is distributed among the MPI processes. Partitioning a forest means
to change this distribution in order to maintain a load balance. After we 
applied partition to a forest the new forest will have equal numbers of elements
on each process (+- 1).

In our example we start with a uniform forest. This forest is constructed
such that each process has the same number of elements. 
We then adapt this forest, thus refining and coarsening its elements. This changes the
number of elements on each process and will almost always result in a load imbalance.
You can verify this yourself by printing the process local number on each process for the 
adapted forest (for example with t8_productionf).
In order to reestablish a load balance, we will construct a new forest from the adapted one via the partition feature.

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_uniform_5ranks.png" height="300">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_adapted_5ranks.png" height="300">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_partitioned_5ranks.png" height="300">
</p>

Let us execute the example with 5 MPI processes and look at the uniform, adapted and partitioned forest.
```bash
mpirun -np 5 ./t8_step4_partition_balance_ghost
```
This will also print out the local number of elements on process 0 (the dark blue elements in the pictures).
For the uniform forest (left picture), we have a perfect load balance. There are 8192 elements in total and process 0
has 1638 local elements (8192/5).

After adapting the forest (middle picture) each process changes its local number of elements.
We now have 2165 global elements, but since on process 0 almost all elements get coarsened, this process now has only 210 elements.
Only half of 2165/5 = 433.
Process 4 (dark red elements) on the other hand has 911 local elements.

When we now call `partition` to reestablish the load balance, the elements get redistributed (right picture) and each process has now exactly 433 local elements.

### Ghost

Many applications require a layer of ghost elements. Ghost element of a process are 
those elements that are not local to this process but have a (face) neighbor element that is.
Telling a forest to create a ghost layer will gather all necessary information and give
us access to the ghost elements.
t8code does not create a layer of ghost elements by default, thus our initial uniform forest
and the adapted forest will not have one.

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_ghost_rank4.png" height="350">
</p>

In this picture we see the local elements of process 4 in red and its ghost elements in blue.
We actually print the ghost elements in the vtu files and you can look at them.
Ghost elements have treeid = -1 in the vtu files. To view the ghosts of a single process (say process 4 as in this picture)
you can set a threshhold on the MPI rank to view only the elements of process 4 and then set a threshold
on treeid to only view those elements with treeid = -1.

### Balance 

A forest fulfills the (face) balance property if (and only if) for each element its (face) neighbors
have refinement level at most +1 or -1 in comparison to the element's level.
The balance property is often broken after adaptation. The Balance algorithm iterates through
the mesh and restores the balance property by refining elements if necessary.
Balance will never coarsen any elements and
t8code does not balance a forest by default.

In this example, the initial uniform forest is balanced since every element has the same refinement level l.
Our adaptation criterion is such that (usually) after one step of adaptation the forest will still be balanced,
since we have level l+1 elements in the inner sphere, level l elements in the middle and level l+1 elements
outside. This may not be the case for very small initial refinement levels or with different radius thresholds (why don't you try it out?).
Therefore, in this example we will apply the adaptation two times, resulting in level l+2 elements
in the inner sphere, level l elements in the middle and level l - 2 element in the outer sphere
(and probably some, but not many, level l-1 and level l+1 elements in between).
This forest will be unbalanced and we will then apply the balance routine to it.
Note that balance changes the local number of elements and thus may also change the load balance
and require repartitioning.
Balance is usually the most expensive of t8code's mesh manipulation algorithms.

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_unbalanced.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_balanced.png" height="350">
</p>
<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_unbalanced_clip.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_balanced_clip.png" height="350">
</p>
Left: The unbalanced forest after applying the adaptation criterion two times.
Right: The forest after `Balance`. Colors represent refinement levels.

### Applying the algorithms to a forest

So far we have seen t8_forest_new_* functions to create forests.
These directly returned a new forest.
However, t8code offers us more control over the creation of forests.
For example we can control whether or not a forest should have a ghost layer,
be balanced/partitioned from another forest, etc.

Usually, there are three steps involved in creating a forest:

1. Initialize the forest with `t8_forest_init`.
This function will prepare a forest by setting default values for its members and 
initializing necessary structures.
```C++
t8_forest_t new_forest;
t8_forest_init (&new_forest);
```

2. Set all properties that the forest should have.
Here we can for example set a cmesh and refinement scheme or specify that the
forest should be adapted/partitioned/balanced from another forest etc.
See the `t8_forest_set_*` functions in `t8_forest.h`.
The order in which you call the set functions does not matter.

In the example we have an adapted forest `forest` that we want to partition and create a ghost layer for:

```C++
t8_forest_set_partition (new_forest, forest, 0);
t8_forest_set_ghost (new_forest, 1, T8_GHOST_FACES);
```

3. Commit the forest with t8_forest_commit. 
In this step the forest is actually created after setting all
desired properties. The forest cannot be changed after it was committed.

```C++
t8_forest_commit (new_forest);
```

The t8_forest_new functions are just wrappers around this process.

If we want to balance a forest, we use `t8_forest_set_balance` which we apply here to the twice adapted forest:
```C++
t8_forest_t unbalanced_forest = t8_step3_adapt_forest (forest);
t8_forest_t balanced_forest;
t8_forest_init (&balanced_forest);
t8_forest_set_balance (balanced_forest, unbalanced_forest, 0);
t8_forest_commit (balanced_forest);
```