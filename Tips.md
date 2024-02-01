We collect general tips and tricks for t8code developers

### Compiling and running individual tests

When writing a new test you do not have to run `make check` every time to compile your single test.
`make check` will compile and run all tests which is rather time consuming.

Instead you can compile and run your tests individually:

```
make test/path/to/test
./test/path/to/test
```

### Deactivating sc and p4est tests

If you are developing and running `make check` often, waiting for sc and p4est tests to compile and run can be tedious.
Especially since you do not change anything in sc and p4est and hence know that the test cases will pass.

Open `sc/Makefile.am` and `p4est/Makefile.am` in an editor.
Comment the lines
`include test/Makefile.am` by adding a `#` in front.