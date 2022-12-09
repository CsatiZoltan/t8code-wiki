Debugging parallel programs can be a challenge and powerfull debuggers for MPI-parallelized codes are usually very expensive and
have a steep learning curve. For rather managable debugging sessions on workstations comprising of just a handful of parallel processes the venerable open source debugger `gdb` can be used.

To find an introduction on how to use `gdb` visit http://www.gdbtutorial.com/. A comprehensive documentation
is available at https://sourceware.org/gdb/onlinedocs/gdb/.

First, compile `t8code` in debugging mode:
```
configure --enable-mpi --enable-debug --enable-static --disable-shared CFLAGS='-Wall -O0 -g' CXXFLAGS='-Wall -O0 -g'
```
Static linkage makes using `gdb` (and `valgrind`) more comfortable.

## Using multiple `xterm` windows
Run `t8code` in parallel:
```
mpirun -n NUM_PROCS xterm -hold -e "gdb -x FILE my_t8code_application"
```
This opens `NUM_PROCS` `xterm` windows with a `gdb` instance each attached to a MPI sub-process.

The command `FILE` might look contain
```
b t8_some_file.c:some_function
b MPI_Abort
run
```
It sets a `breakpoint` for `some_function` in `t8_some_file.c` and also for `MPI_Abort` calls.
The latter prevents closing the `gdb` sessions in case of a crash. `run` is necessary
to actually start the program.

## Using multiple `tmux` panes
If there is no X11 session available (headless mode, i.e. no GUI), for example when working on a cluster
over SSH, then alternatively, one could use `tmux`. `tmux` is a terminal multiplexer. It lets you switch
easily between several programs in one terminal, detach them (they keep running in the background) and
reattach them to a different terminal. This is especially useful for remote sessions over SSH. More
information on `tmux` is available at https://github.com/tmux/tmux/wiki.

There is a tool [tmpi](https://github.com/Azrael3000/tmpi) which sets up the panes and even multiplexes
your keyboard input to all `gdb` sessions at once.
```
tmpi NUM_PROCS gdb my_t8code_application
```
This opens `NUM_PROCS` `tmux` panes with a `gdb` instance each attached to a MPI sub-process.




