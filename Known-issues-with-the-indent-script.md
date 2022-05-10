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

The `indent` program sometimes has problems to recognize custom data type (i.e. typedefs) resulting in misindentations.
To prevent these, add the custom data types to the file `scripts/t8indent_custom_datatypes.txt`.

The problems occurring with unrecognized data types are listed below.

#### modifiers

If `indent` does not recognize a data type, indentation of function return values fails when additional modifiers are used:

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

#### t8_element_t and sc_array_t

Even though we added `t8_element_t` and `sc_array_t` to the list of custom data types, functions returning a
pointer to one of these sometimes get misindented.

The indent script produces
```
t8_element_t       *
t8_foo ();
```

instead of the correct
```
t8_element_t *
t8_foo ();
```

#### pointers

Pointers to unknown data types get an extra space that we do not want.

Wrongly indented by `indent`:
```
t8_forest_t * pforest;
```

Correct indentation:
```
t8_forest_t *pforest;
```