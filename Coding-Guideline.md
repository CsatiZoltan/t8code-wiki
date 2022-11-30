# Coding Guidelines

## Indentation

`t8code` comes with its own indentation script `t8indent` that you should use to indent all code files.
You find it in the `scripts` folder of the main repository.

In order to indent a file `src/t8_foo.c` call

```bash
$ ./scripts/t8indent src/t8_foo.c
```

Sometimes the indentation script cannot indent a part of the code properly, or produces ugly results.
So you need to double check the file after indentation.
If you encounter a piece of code that is not indented properly, you can use the `/* *INDENT-OFF* */` and `/* *INDENT-ON* */`
comments to deactivate indentation for a part of the code:

```C++
/* *INDENT-OFF* */
/* Now the code here will be left unchanged by t8indent */

/* *INDENT-ON* */

/* The code here will be indented by t8indent */
```

You can `grep` the code base for these keyword in order to see some of the fails of `t8indent`.

Additional scripts to help with indentation, can be found in the `scripts` folder, see also the corresponding [README file.](https://github.com/holke/t8code/tree/develop/scripts#readme)

### Git indentation workflow

We provide a git hook that automatically prevents you from committing unindented files.
You should install this commit hook by copying it to the `.git/hooks` directory.

```bash
$ cp ./scripts/pre-commit .git/hooks
```

If you now try to commit an indented file you will get a message such as this one:
```bash
$ git add src/t8_foo.c
$ git commit -m 'A useful commit message'
Checking file src/t8_foo.c
File src/t8_foo.c is not indented.
```
So now we need to indent the file.

```bash
$ ./scripts/t8indent src/t8_foo.c
```

Check the changes and add them. We can use the `git add -p` feature for this:
```bash
$ git add -p ./src/t8_foo.c
```

It is important that you do not mix indentation of code that has nothing to do with your actual
commit into the commit.
So make sure to only include those indented parts of the code you are currently editing into the commit.

If you are finished with your commit and there are still indentation changes left, you can commit them all together
in a single indent commit.

Let us do a quick example. Suppose we added a function `foo` to `t8_foo.c`, but other parts of the file are unindented
(of course not you, but other developers, are to blame for this shameful error ;) ).

```bash
# Add your changes to the commit
$ git add src/t8_foo.c
# Try to commit your changes
$ git commit -m 'Added function foo to t8_foo.c' # Please use a more descriptive commit message
Checking file src/t8_foo.c
File src/t8_foo.c is not indented.
# Indent the file
./scripts/t8indent src/t8_foo.c
# Add the changes to the foo function part to our commit
git add -p # Now interactively accept only the changes to the foo part
# Commit again, this time it should work
$ git commit -m 'Added function foo to t8_foo.c'
# Now add the other changes of indent
$ git add src/t8_foo.c
# And commit them as an indentation commit
$ git commit -m 'Indented t8_foo.c'
```

Despite all effort, the indentation hook sometimes just wont work properly; see also the page on [Know issues with indent](https://github.com/holke/t8code/wiki/Known-issues-with-the-indent-script).
If you really need to commit code that does not pass the indent test, using `INDENT-OFF/ON` fails, and you know what you are doing, you can use `git commit --no-verify`.



## Comments

We use C-Style multiple line comments throughout.

```C++
/* Use this to comment a single line. */

// Not this.

/* Multiple line comments
 * should start each line with a '*'
 * And end with a new line.
 */

/* Each comment line should start and end with a space. */
/*Because this looks ugly.*/
```

All comments should be written as complete sentences with appropriate
English grammar, correct spelling and proper punctuation. Again,
add margins (whitespaces) between the text and the comment markers.

```C++
/* This is a comment written as a complete sentence, ending with a dot
 * and keeping proper margins (one whitespace) towards the '*' markers.
 */
```

Of course, short statements, resp. remarks, are also ok.

```C++
/* Checking for xyz. */
... some code ...
```

Every function should be well documented.
This includes a description of the function and of its parameters.

We stick to the doxygen style, using `\param` followed by `[in]`, `[out]` or `[in,out]` 
to describe function parameters and `\return` to describe the return value.
Do this even in `.c` and `.cxx` files that are not compiled with doxygen.

```C++
/** Implements the integers identity function.
 *  \param [in] x  The input value.
 *  \return        The value \a x will be returned.
 */
int identity (int x) 
{
  return x;
}
```


## Memory handling

`t8code` implements its own memory handling routines.
To allocate and free memory, we use the `T8_ALLOC*` and `T8_FREE` macros.

All our code must be free of memory leaks. Check your code with `valgrind` and `valgrind --leak-check=full` for any memory leaks.


## Naming conventions

### Prefix

We use the prefix `t8` to indicate `t8code`'s namespace. Thus, every function, struct and class should have a `t8` prefix.

```
t8_foo (int x)
{
   /* Do something useful */
}

struct t8_struct
{
  int foo;
};
```

Functions that belong to a particular struct interface should all have the same prefix which should be related to the struct.

For example all functions in the forest API have `t8_forest` as a prefix.

### Naming scheme

We use the lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case) naming convention for functions, structs, classes and variable names:

```
t8_prefix_explanatory_name
```

For compiler macros, we use SCREAMING_SNAKE_CASE with all uppercase letters:

```
T8_MACRO_NAME
```

### Loop variables

As all variables loop variables should get an explanatory name.
Especially integer type loop variables should start with a small `i`, `j`, or `k`.

Good examples: `iface_number, jelement`

Bad example: `i`

## Debugging mode

The debugging mode (configure option `--enable-debug`) can and should be used to perform runtime checks.

You can recognize the debugging mode because the macro `T8_ENABLE_DEBUG` is set.

```C
#ifdef T8_ENABLE_DEBUG
/* Code that is only executed in debugging mode */
#endif
```

Debugging mode is not performance critical and you can use it for expensive checks.

Note, however, that these will not be executed in release mode (without `--enable-debug`), so it should only be used for code that is not required for the successful operation of t8code.

## Assertions

You can use assertions via the macro `T8_ASSERT (expr)`. This macro is only active in debugging mode.
If `expr` is true, nothing happens. If `expr` is false the code will abort and provide you with information where the abort occured.

A typical usage of assertions is to check for implicit assumptions.
Consider a function `void t8_foo (t8_forest_t forest);` that operates on a forest, but requires the forest to be committed.
We can use `T8_ASSERT` to check the incoming argument in debugging mode.

```C
void t8_foo (t8_forest_t forest)
{
  T8_ASSERT (t8_forest_is_committed (forest));
  /* Remaining code */
}
```
This will catch a missusage of the function in debugging mode (but not in release mode).

We recommend to use assertions frequently.

## Scope of variables

The scope of variables should be as small as possible.

Use
```C++
for (int level = 0; level < ...
```
instead of
```C++
int level;
/* Some Code */
for (level = 0; level < ...
```

## Constness and declaration of variable

Follow the rule "if a variable can be const then it should be const". Also, declare variables at best when they are used.

Use 
```C++
const t8_locidx_t num_elements = t8_forest_get_num_local_elements (forest);
```

instead of

```C++
t8_locidx_t num_elements;
/* Some Code */
num_elements = t8_forest_get_num_local_elements (forest);
```

It is also better to use multiple (const) variables than to reuse the same variable for multiple purposes.

Note, that this change is introduced quite recently to t8code and a lot of legacy code does not follow this convention yet.

## General rules

* Output parameters in function declarations should come after input parameters.

* Use a new declaration line for each variable, even if they are of the same type:
```
int foo;
int bar;
```
instead of
```
int foo, bar;
```

 * Avoid "magic numbers". If you use a number as input that has a proper meaning that is not immediately clear from the context, give this variable a name. Either by using a constant variable or a preprocessor macro.
```
for (iface = 0; iface < T8_DTRI_NUM_FACES; ++iface) ...
```
instead of
```
for (iface = 0; iface < 3; ++iface)
```

## Application Programming Interface (API)

In `t8code` we have internal APIs and public APIs. In general, most
rules apply for both kinds of API.

### Minimal API Surface
Every API design MUST aim for a minimal API surface without sacrificing on product requirements. API design SHOULD NOT include unnecessary resources, relations, actions or data. API design SHOULD NOT add functionality until deemed necessary ([YAGNI principle](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)).

Or in other words, add as few routines as possible in the header files and declare helper routines as `static`.
If in the future, a helper routine is needed elsewhere, move the declaration
into a header file.

### Robustness
Every API implementation and API consumer MUST follow Postel's law:

> Be conservative in what you send, be liberal in what you accept.
– [John Postel](https://en.wikipedia.org/wiki/Robustness_principle)

### To be extended ...
