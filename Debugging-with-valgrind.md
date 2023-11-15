
t8code aims to handle memory with care. Our goal is to avoid any memory leaks. Nevertheless, memory leaks can occur during development and it can be challenging to find and fix them. 

A first hint that something is wrong can be the message:
```
Abort: Finalize
Abort: ../../t8code/sc/src/sc.c:1384
Abort
Aborted
```
triggered by ```sc_finalize()``` in your code. t8code checks if allocated memory has been free'd at the end of the code and prints this error message if it isn't the case. 

## Valgrind 

[valgrind](https://valgrind.org/) is a tool to find and fix memory leaks and in general bugs in your memory management. A detailed documentation can be found [here](https://valgrind.org/docs/manual/index.html). This article gives a short introduction in how to find the most common memory bugs that occur while developing t8code. 

For a first brief analysis of your memory management you can run your code with ```valgrind ./my_program```
If your program does not have any memory leaks you should get an output like this:
```
==41686== LEAK SUMMARY:
==41686==    definitely lost: 0 bytes in 0 blocks
==41686==    indirectly lost: 0 bytes in 0 blocks
==41686==      possibly lost: 0 bytes in 0 blocks
```

If your program does have a memory leak, Valgrind will show non-zero numbers here. 

## An example
For a mock example we can use the tutorial step 1, where we put the line where the cmesh is destroyed in comments. 
Rerunning the program with ```valgrind --leak-check=full --log-file="my_log.txt" ./t8_step1_coarsemesh``` will produce a more detailed analysis of your memory management. Furthermore, the output is put in a logfile. 
It is easier to search for the error in a logfile, than on your console. Most t8code related functions start with "t8_" so it is a good starting point for the search of our memory leak. A lot of other output comes from MPI, which can be ignored for memory leaks concerning t8code. It is even possible to suppress these errors completely. 

In the logfile we will find error messages like this:
```
==52893==    by 0x4038F6: main (t8_step1_coarsemesh.cxx:102)
==52893== 
==52893== 48 bytes in 1 blocks are possibly lost in loss record 1,248 of 2,592
==52893==    at 0x485B212: operator new(unsigned long) (vg_replace_malloc.c:417)
==52893==    by 0x417C3B: t8_geometry_linear_new (t8_geometry_linear.cxx:95)
==52893==    by 0x40F62F: t8_cmesh_new_hypercube (t8_cmesh_examples.c:637)
==52893==    by 0x40382F: t8_step1_build_tetcube_coarse_mesh(int) (t8_step1_coarsemesh.cxx:63)
==52893==    by 0x403982: main (t8_step1_coarsemesh.cxx:120)
```
We see a trace of commands and after having a look at each of them we see that it points us to the place where the cmesh is allocated. This indicates the missing ```t8_cmesh_destroy(&cmesh)``` which we just removed from the file. Handling the destruction of the cmesh properly will fix this problem. 

## Suppressing error messages
As mentioned before, you can get a lot of MPI related output in your logfile, which makes it more difficult to analyse the memory management. You can create and use suppression files to ignore this output. 
Adding
```
--gen-suppressions=all
```
to your valgrind command will create suppression patterns for each error looking like this:
```
{
   <insert_a_suppression_name_here>
   Memcheck:Leak
   match-leak-kinds: possible
   fun:calloc
   fun:allocate_dtv
   fun:_dl_allocate_tls
   fun:allocate_stack
   fun:pthread_create@@GLIBC_2.2.5
   fun:sock_ep_cm_start_thread
   fun:sock_domain
   fun:create_vni_context
   fun:MPIDI_OFI_mpi_init_hook
   fun:MPID_Init
   fun:MPIR_Init_thread
   fun:PMPI_Init
   fun:main
}
```
You can add them to a file ending with ```.supp``` and give each of them a proper name. 
To be less specific you can remove lines and replace them with the frame level wildcard ```...```. For most error outputs this suppression is sufficient:
```
{
   <mpi_supp_1>
   Memcheck:Leak
   match-leak-kinds: possible
   ...
   fun:MPIDI_OFI_mpi_init_hook
   fun:MPID_Init
   fun:MPIR_Init_thread
   fun:PMPI_Init
   fun:main
}
```
Rerunning with ```valgrind --suppressions=./path_to_my_suppression_file.supp ./my_program``` should give an output focused on t8code related functions. 

## Valgrind and MPI
It can be especially painful debugging a memory leak that only occurs in a parallel execution. A first memory analysis can be done with
```mpirun -n num_procs valgrind --valgrind_options ./my_program```, which will also result in a mixed-up output. 
valgrind is aware of process IDs and we can use them to produce separate logfiles using ```mpirun -n num_procs valgrind --valgrind_options --log-file=logfilename.%p ./my_program```.
You can find more information about how to use Valgrind and MPI [here](https://valgrind.org/docs/manual/mc-manual.html#mc-manual.mpiwrap).


