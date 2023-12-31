## Introduction
For testing the correctness of our functions in t8code, we use [MPI extensions of the GoogleTest framework](https://github.com/DLR-SC/googletest_mpi/tree/feature/mpi_nonblocking_expect-1.10.0/). This framework allows us to test code sections independently for several parameters and properties of t8code. The task is finding code errors in the early development of new t8code sections. You get more information and guidelines about the implementation of GoogleTests at [GoogleTest User’s Guide](https://google.github.io/googletest/).

## Guidelines
A test should only check one property of the code. A common test is the [Parameterized Test](https://google.github.io/googletest/advanced.html#value-parameterized-tests). These tests allow you to test code sections over a range of variables. In our case, this is the go-to when testing element-functions for all available elements in a scheme. 
The tests should not test specific cases, but should also be applicable to code extensions.
Keep the code simple and understandable. For this you should follow our [Coding Guideline · DLR-AMR/t8code](https://github.com/DLR-AMR/t8code/wiki/Coding-Guideline).

## Output
The test should not print an output apart from error-messages of failing tests. Other messages should only print a message in debug configuration. Use the command `t8_debugf` to print such messages.

## Assertions and expectations 
Various checks can be applied to the code. There are assertions and expectations. The expectations are called by `EXPECT_*`. If an expectation fails, the current test (or function) will not be aborted. 
By using an assertion, which is called by `ASSERT_*`, a test (or a function) will abort directly. Note that if a function will be aborted, that the test will not be aborted. It is not possible to use assertions in functions, that have a return value. 

The rule of thumb should be: Use `EXPECT_*` whenever possible. Use `ASSERT_*` only if the test cannot continue if the asserted part fails.

The most common assertions and expectations are:
* EXPECT/ ASSERT_TRUE
* EXPECT/ ASSERT_FALSE
* EXPECT/ ASSERT_EQ		(equal)
* EXPECT/ ASSERT_NE		(not equal)
* EXPECT/ ASSERT_ LT		(less than)
* EXPECT/ ASSERT_LE		(less or equal)
* EXPECT/ ASSERT_GT		(greater than)
* EXPECT/ ASSERT_GE		(greater or equal)

More checks on: [Assertions Reference | GoogleTest](https://google.github.io/googletest/reference/assertions.html).

Choose the most fitting check for your test. Use `ASSERT_TRUE`, `ASSERT_FALSE`, `EXPECT_TRUE` or `EXPECT_FALSE` only if no other check can be applied.

## Customized Assertions for t8code
In many test we check the equality for two elements. We implemented a customized EXPECT/ASSERT for that case. It also handles the printing of the elements (coordinates, level, ...). If you want to check if two elements are equal use `EXPECT_ELEM_EQ`. 
To use this function in your test you have to include the header `test/t8_gtest_custom_assertion.hxx` in your testfile.
If you find another case, where a customized assertion can be benefitial, have a look at [Advanced Google Test](https://google.github.io/googletest/advanced.html) and [Pretty formatted Google Test Assertions](https://google.github.io/googletest/reference/assertions.html#EXPECT_PRED_FORMAT) to contribute your customized assertion. 

## Failure message
If a test fails we would like to know more about the reason for aborting. A failure message, which is connected with an assertion (or expectation) by using `<< “my_string”` throws an information message if the test fails at this point. 
```
ASSERT_TRUE(t8_cmesh_is_commited(cmesh)) << "Cmesh commit failed.";
ASSERT_EQ(dual_face, checkface) << “Wrong dual face. Expected “ << checkface << “ got ” << dual_face;
```

## MPI extensions of Google Test

Since t8code is designed as an MPI parallel library, the tests are executed in parallel as well.

However, the original GoogleTest is not designed for MPI parallel use. Hence, we use the [MPI extension](https://github.com/DLR-SC/googletest_mpi/tree/feature/mpi_nonblocking_expect-1.10.0/).

This extension communicates the test result automatically among all processes in `MPI_COMM_WORLD`.
Hence, if the test fails on one process it fails on all processes. This ensures that the failure is definitely reported.

Additionally, the extension introduces the new `EXPECT_*_MPI` and `ASSERT_*_MPI` macros. These are MPI collective variants of `EXPECT_*` and `ASSERT_*. Thus, if the specific check fails on one process, then it fails on all processes.

**Whenever you need to ASSERT something and carry out parallel communication afterwards, you must use one of the ASSERT_*_MPI macros.**
**Otherwise, the test could run in a deadlock.**

For more information, see https://github.com/DLR-SC/googletest_mpi/blob/feature/mpi_nonblocking_expect-1.10.0/googletest/docs/MPIGuide.md
