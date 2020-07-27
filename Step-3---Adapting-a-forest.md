## Step 3 - Adapting a forest

Adaptation is the process of refining and coarsening the elements in a forest according to
a given criterion. It is the core idea behind adaptive mesh refinement and thus the first major
mesh handling algorithm that we discuss.

In this example we describe how to define an adaptation criterion and how to adapt an existing forest.

You will find the code to this example in `example/tutorials/t8_step3_adapt_forest.cxx` and it creates the executable `example/tutorials/t8_step3_adapt_forest`.

As in the previous examples ([Step 1](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh) and [Step 2](https://github.com/holke/t8code/wiki/Step-2---Creating-a-uniform-forest)) we will use a cube geometry for our coarse mesh, but this time modelled as a hybrid
mesh with different element types (tet, prism and hex).

The refinement criterion will be a geometrical one. We will refine elements if they are within a radius of 0.2 of the point `(0.5, 0.5, 1)` and we will coarsen elements if they are outside a radius of 0.4.

We start with a uniform forest as in step 2 and adapt once, thus refining or coarsening by one level.


<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_uniformForest_lvlcolor.png" height="350">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_adaptForest_lvlcolor.png" height="350">
</p>

The uniform level 3 forest (left) will get adapted and then also have level 4 and level 2 elements (right).
The colors correspond to the refinement levels.

### Adapting a forest

When adapting a forest, we as the user are responsible for defining a suitable adaptation criterion in terms of a `t8_forest_adapt_t` callback function.

`t8code` will iterate through all elements of the forest and call the provided callback function once per element.
If the return value of the callback is `0` then the element will be kept in the forest. If the return value is `>0` then the element
will be refined, thus replaced with its children.

A family of elements is the set of all children of the same element (the parent element).
If the input element to the callback is the first element in a family - and the whole family is local to the same process -
then the whole family is passed to the callback function and a return value `<0` will specify that the family should be coarsened, thus replaced with its parent.
In this case, the callback will not be called again for any member of the family.


<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_square.png" height="250">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_squarerefined.png" height="250">
</p>
A square element (left) is refined into its 4 children (right).

<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_trianglefamily.png" height="250">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Step3_trianglecoarsened.png" height="250">
</p>
A triangle family (left) is coarsened into their parent (right).

Let us for now take the adaptation callback as a black box. We will describe it in detail at the end of
this page.

If we have an adaptation callback `t8_step3_adapt_callback` and a forest object `forest`, we can create a new forest from it via adapting with

```C
t8_forest_t forest_adapt = t8_forest_new_adapt (forest, t8_step3_adapt_callback, 0, 0, &adapt_data);
```

This will construct a new forest object `forest_adapt` whose elements arise from adapting the elements in `forest` according to the `t8_step3_adapt_callback` criterion.
The `adapt_data` argument is user defined data that we can pass onto `forest_adapt` and access during the callback with `t8_forest_get_user_data`.

The two `0` parameters are flags that specify
- Recursive adaptation. If this is true, the adapt callback will be called recursively for the elements
  until no element will further be refined or coarsened. 
- Create face ghosts. If this is true, `forest_adapt` will create a layer of (face) ghost elements.

### Reference counting

After calling `t8_forest_new_adapt` the old `forest` object will be destroyed.
This is due to `t8code`'s internal reference counting.
Each `forest` counts how often it is referenced, starting with 1 when it is constructed.
Using a `forest` to construct a new forest from it will decrease this reference count by 1, basically saying 'i do not need this forest anymore'.
If the reference count reaches 0 the `forest` is destroyed.

Often you will want to keep the `forest` after adapting. For example because you stored element
data (such as function values in a CFD solver) and you still need to properly interpolate this data onto the new adapted forest.
In this case, just call
```C
t8_forest_ref (forest);
```
to manually increase the reference counter.

Analoguously you can dereference a `forest` - and hence also destroy it, if its count reaches 0 - with
```C
t8_forest_unref (&forest);
```

The same applies to other structures such as `t8_cmesh_t` and `t8_scheme_cxx_t`.

### The adaptation callback

Let us now discuss the adaptation callback for our example in detail.

The callback declaration looks like this (see also `t8_forest_adapt_t` in `t8_forest.h`):
```C
int                 t8_step3_adapt_callback (t8_forest_t forest,
                                             t8_forest_t forest_from,
                                             t8_locidx_t which_tree,
                                             t8_locidx_t lelement_id,
                                             t8_eclass_scheme_c * ts,
                                             int num_elements,
                                             t8_element_t * elements[]);
```
The parameters are
| | |
|-|-|
| forest | The new forest that is currently under construction |
| forest_from | The old forest that is to be adapted |
| which_tree | Index of the current tree in `forest_from` |
| lelement_id | Index of the current element in the elements of the current tree |
| ts | The refinement scheme for this particular element shape |
| num_elements | How many elements are currently considered (either 1 or a family) |
| elements | The elements that are currently considered for adaptation |

To explain these a bit: A `forest` store its elements in different arrays. One for each (process local) tree. Thus, in order to find an element we need the number of its tree and the index of it
within this tree. These are `which_tree` and `lelement_id`.
We have already seen `t8_scheme_cxx_t`, which specifies the refinement scheme. This stores for each
element shape one member of type `t8_eclass_scheme_c` which provides the necessary functions
for this element shape (i.e. triangle, tetrahedron, square, etc.). The `ts` parameter provides us with the scheme for the currently active element.
We could for example use this to compute the element's level with
```C
int level = ts->t8_element_level (element);
```

We want to refine or coarsen an element depending on its distance from a given point.
In order to do so, we need to compute the coordinates of its midpoint via
```C
t8_forest_element_centroid (forest_from, which_tree, elements[0], tree_vertices, centroid);
```
which stores the coordinates into the variable `double centroid[3]`.
However, it also needs `tree_vertices` as input. These are the coordinates of the vertices
of the current tree. They are needed since we do not store the element's coordinates explicitly
but can compute them from the tree's vertex coordinates since the elements follow a structured
refinement scheme.

We compute these with
```C
const double *tree_vertices = t8_forest_get_tree_vertices (forest_from, which_tree);
```

All we need to do now is to compute the distance between `centroid` and our point `(0.5,0.5,1)`
and depending on certain thresholds refine the element, coarsen the family or do nothing.

How do we provide the midpoint and the thresholds to the callback function?
This is extra data that the adapt callback needs and in order to provide it, we have to use
a forest's user data that allows us to attach arbitrary data to a forest.

```C
struct t8_step3_adapt_data
{
  double  midpoint[3];               /* The midpoint of our sphere. */
  double  refine_if_inside_radius;   /* if an element's center is smaller than this value, we refine the element. */
  double  coarsen_if_outside_radius; /* if an element's center is larger this value, we coarsen its family. */
};
```

We can now build a variable
```C
struct t8_step3_adapt_data adapt_data = {
    {0.5, 0.5, 1},              
    0.2,                        
    0.4                         
  };
```

with the values we need and pass it onto `t8_forest_new_adapt`.
Inside the callback we can access this with

```C
const struct t8_step3_adapt_data *adapt_data = (const struct t8_step3_adapt_data *) t8_forest_get_user_data (forest);
```

Finally, we apply our criterion:
```C
/* Compute the distance to our sphere midpoint. */
  dist = t8_vec_dist (centroid, adapt_data->midpoint);
  if (dist < adapt_data->refine_if_inside_radius) {
    /* Refine this element. */
    return 1;
  }
  else if (num_elements > 1 && dist > adapt_data->coarsen_if_outside_radius) {
    /* Coarsen this family. Note that we check for num_elements > 1 before, since returning < 0
     * if we do not have a family as input is illegal. */
    return -1;
  }
  /* Do not change this element. */
  return 0;
```