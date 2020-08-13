# Coding Guidelines

## Comments

Every function should be well documented.
This includes a description of what the function does
and of its parameters.
```C++
/* Implements the integers identity function.
 * \param [in] x  The input value.
 * \return        The value \a x will be returned.
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

