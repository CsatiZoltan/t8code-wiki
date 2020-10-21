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

Despite all effort, the indentation hook sometimes just wont work properly.
If you really need to commit code that does not pass the indent test. and you know what you are doing, you can use `git commit --no-verify`.



## Comments

Every function should be well documented.
This includes a description of what the function does
and of its parameters.

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

We use C-Style multiple line comments throughout.

```C++
/* Use this to comment a single line */

// Not this

/* Multiple line comments
 * should start each line with a '*'
 * And end with a new line.
 */

/* Each comment line should start and end with a space. */
/*Because this looks ugly.*/
```

### Memory handling

`t8code` implements its own memory handling routines.
To allocate and free memory, we use the `T8_ALLOC*` and `T8_FREE` macros.

All our code must be free of memory leaks. Check your code with `valgrind` and `valgrind --leak-check=full` for any memory leaks.


### Naming guidelines

#### Prefix

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

#### Naming scheme

We use the lowercase [snake_case](https://en.wikipedia.org/wiki/Snake_case) naming convention for functions, structs, classes and variable names:

```
t8_prefix_explanatory_name
```

For compiler macros, we use SCREAMING_SNAKE_CASE with all uppercase letters:

```
T8_MACRO_NAME
```