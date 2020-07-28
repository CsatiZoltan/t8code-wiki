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
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_uniform_5ranks.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_adapted_5ranks.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step4_partitioned_5ranks.png" height="350">
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
 Collects information about ghost elements. These are those elements that are (face) neighbor to process local elements, but are
not local to this process themselves.

### Balance 
Iterates through the mesh and refines elements until any two neighboring elements have level difference at most +-1.
