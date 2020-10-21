The indentation sometimes messes up.
We currently rely on `GNU indent` to perform the indentation and this script is optimized for `C` code
and has some problems with indenting `C++`.
We are working on a solution.

### Duplicate const

`const` modifiers at the end of class member functions get duplicated.
```C++
void 
foo() const
{
}
```
is indented to
```C++
void 
foo() const const
{
}
```
which is not valid code.

### Custom data types

If `indent` does not recognize a data type, indentation of function return values failes when additional modifiers are used:
```C
static my_own_type 
foo()
{
}
```
is indented to
```C
static             my_own_type
foo()
{
}
```
Correct would be:
```C
static my_own_type
foo()
{
}
```