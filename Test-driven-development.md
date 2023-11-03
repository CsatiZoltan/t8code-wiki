# Test driven development

In general, when implementing new features or resolving bugs one should first implement unit tests that cover the feature and implement the feature afterwards.

This way, during development, the test can already be run to control progress.

See also: https://en.wikipedia.org/wiki/Test-driven_development

See also: https://github.com/DLR-AMR/t8code/wiki/Testing-with-GoogleTest

## How to add tests before implementing the functionality

It is good practice to implement and add unit tests for a bug or new feature even though you are currently not working towards resolving this feature.

However, until the feature is implemented the tests will fail, how do we handle this situation?

Our strategy is as follows:

1. Implement and add the new test cases.

2. Add the GoogleTest Macro `GTEST_SKIP()` to your tests, such that it will be skipped when t8code executes its tests.
 See: https://google.github.io/googletest/advanced.html#skipping-test-execution

Example:
```C++
TEST(SkipTest, DoesSkip) {
  GTEST_SKIP() << "Skipping single test";
  EXPECT_EQ(0, 1);  // Won't fail; it won't be executed
}

class SkipFixture : public ::testing::Test {
 protected:
  void SetUp() override {
    GTEST_SKIP() << "Skipping all tests for this fixture";
  }
};

// Tests for SkipFixture won't be executed.
TEST_F(SkipFixture, SkipsOneTest) {
  EXPECT_EQ(5, 7);  // Won't fail
}
```

3. If it does not exist yet, create a new issue covering the bug/feature.

4. Add the link of the issue as a comment in the new tests.

```C++
/* This test tests the functionality described in Issue: [LINK] 
 * Remove the GTEST_SKIP() macros when you start working on the issue.
 */
TEST(SkipTest, DoesSkip) {
  GTEST_SKIP() << "Skipping single test";
  EXPECT_EQ(0, 1);  // Won't fail; it won't be executed
}
```

5. In the issue add a new message/text that links to the new tests.
```
// Issue text here

For this issue, tests have already been implemented at [PATH/TO/TEST1] [PATH/TO/TEST2] ... .
Remember to activate the tests by removing any `GTEST_SKIP()` macros when you start working on this issue.
```