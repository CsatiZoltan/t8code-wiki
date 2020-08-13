# Coding Guidelines

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

### Doxygen

