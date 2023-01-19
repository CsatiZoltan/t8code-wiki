## Search

In this tutorial we discuss t8code's search algorithm.
You will find the code to this example in `tutorials/general/t8_tutorial_search.cxx` and it creates the executable `tutorials/general/t8_tutorial_search` (Currently, not on the main, but on the develop branch).

We assume that you know how to generate a coarse mesh and an adapted forest on it.
If not, please read [Step 1](https://github.com/holke/t8code/wiki/Step-1---Creating-a-coarse-mesh), [Step 2](https://github.com/holke/t8code/wiki/Step-2---Creating-a-uniform-forest) and [Step 3](https://github.com/holke/t8code/wiki/Step-3---Adapting-a-forest/_edit).

The purpose of search is to efficiently identify a subset of elements that match given conditions and to execute
callback functions for these elements.

Examples of this feature include:
 - Identifying the elements at the domain boundary.
 - Finding elements that contain particles.
 - In a weather simulation identify all elements that contain a hurricane.

In this tutorial we will create random particles inside our mesh, then use search to find those elements that contain particles and
count for each element how many particles it contains.

Due to the tree-based structure of t8codes meshes we can search the mesh hierarchically from the coarsest level down to the elements,
checking on each level whether or not to continue the search.
This way, we can exclude whole portions of the mesh from the search early on and thus have a significant performance advantage compared to linearly searching through all elements as would be necessary with unstructured meshes.
You can see in the output of the executable that the number of searched elements is
significantly smaller than the actual number of elements:
To search for 2000 particles in a forest with 141,052 elements, the search looked at 16,448 elements.


<p align="center">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Search_forest.png" height="250">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Search_forest_clip.png" height="250">
<img src="https://github.com/holke/t8code/wiki/pictures/tutorials/Search_forest_particles.png" height="250">
</p>

Left to right: The forest that we use in this example. A cut through the forest to show the inside (colors represent refinement levels.)
The elements that we searched for that contain particles (colors represent number of particles).

## Callback functions and queries

  All we have to do to use search is to define what a query is (if we use any) and 
  implement two callback functions (one if we do not have queries), the element callback and the query callback.
  
  In each (process local) tree of the forest, search will create the level 0 element that
  coincides with the tree and call the element callback function on it.
  In the element callback we decide whether to continue the search or not.
  If we continue the search, the children of this level 0 element are created and the
  element callback will be called for them -- again deciding whether to continue or not.
  This process repeats recursively and stops at those fine elements that are actually contained
  in the forest (leaf elements).
  
  Additionally, the search algorithm can be given an array of 'queries' and a query callback. 
  These queries can be arbitrarily defined data. In our case the queries will be our particles.
  If queries and a query callback are provided, then for each element first the element callback
  is called to decide whether or not to continue searching. If it returns true, the query-callback
  will be called once for each active query object. 
  If the query object returns 0, this query object will get deactivated for this element and its
  recursive children. The recursion stops when no queries are active anymore (independently of the return value of the per element callback).

  Both callbacks have the same signature.
  When they are called information about the currently searched element, whether or not it is a leaf
  and its descendants that are leafs is provided.
  In query mode a pointer to the current query and its index in the query array are also provided.

```C++
int         search_callback (t8_forest_t forest,          // The forest
                             t8_locidx_t ltreeid,         // The current tree
                             const t8_element_t *element, // The element to be searched
                             const int is_leaf,           // True if the element is a leaf (an actual element in our forest)
                             t8_element_array_t *leaf_elements, // The leafs in the forest that are descendants (arise from recursive refining) of element
                             t8_locidx_t tree_leaf_index, // The tree local index of the first leaf in leaf_elements
                             void *query,                 // The current query (NULL in element callback mode)
                             size_t query_index)          // The index of the current query in the query array (invalid in element callback mode)
```

## Particles

We want to create random particles in the mesh and search for the elements containing them.
A particle is just a point in 3D.
```C++
typedef struct
{
  double              coordinates[3];   /* The coordinates of our particle. */
} t8_tutorial_search_particle_t;
```

In `t8_tutorial_search_build_particles` we build an `sc_array` of `num_particles` many particles with random coordinates in a certain subregion of our mesh.

This array will be passed to search as our `queries` and for each element the active queries are those particles that are contained inside the element.

We also want to count for each element how many particles it contains and how many elements we searched in total.
Thus, we build a user data struct
```C++
typedef struct
{
  sc_array           *particles_per_element;    /* For each element the number of particles inside it. */
  t8_locidx_t         num_elements_searched;    /* The total number of elements created. */
} t8_tutorial_search_user_data_t;
```
that we will pass onto our search callback.
The `particles_per_element` array will be allocated to hold as many integers as we have (process local) elements.

## The callbacks

Our two callbacks are now as follows:

### The element callback

The per element callback that is called once for every searched element and returns true if the recursion should
continue with the element's children.
In our case, we want to continue as long as we have particles left that may be contained in the element.
Since the recursion will stop automatically when we have no active queries, we can always return true in the element callback.

We use our user data to count the searched elements.

```C++
static int
t8_tutorial_search_callback (t8_forest_t forest,
                             t8_locidx_t ltreeid,         // The current tree
                             const t8_element_t *element, // The element to be searched
                             const int is_leaf,           // True if the element is a leaf (an actual element in our forest)
                             t8_element_array_t *leaf_elements, // The leafs in the forest that are descendants (arise from recursive refining) of element
                             t8_locidx_t tree_leaf_index, // The tree local index of the first leaf in leaf_elements
                             void *query,                 // The current query (NULL in element callback mode)
                             size_t query_index)          // The index of the current query in the query array (invalid in element callback mode)
{
  T8_ASSERT (query == NULL);

  /* Get a pointer to our user data and increase the counter of searched elements. */
  t8_tutorial_search_user_data_t *user_data =
    (t8_tutorial_search_user_data_t *) t8_forest_get_user_data (forest);
  T8_ASSERT (user_data != NULL);
  /* Count this element */
  user_data->num_elements_searched++;
  /* Continue the search */
  return 1;
}
```

### The query callback

In our example this is the callback that does the actual computation.
It is called once for each particle that may be contained in the element (these are the active particles).

We check whether the current particle is actually contained in the element using the `t8_forest_element_point_inside` function. If yes, the we return true,
and the particle will remain active for this element. Thus we will check in which children it is contained.
If no, then we return false, deactivating the particle and it will not be checked again for any of the element's
children.
If we return false for every active particle of an element, the search will stop for the element and all of its descendants.

We also get the information whether or not the currently searched element is a leaf element in the forest.
If it is and we find that the particle is contained in the element, we have found an element that contains a particle.
What we do now is to add the particle to the particle counter of this element.

The important part of the callback is:
```C++
  /* Cast the query pointer to a particle pointer. */
  t8_tutorial_search_particle_t *particle =
    (t8_tutorial_search_particle_t *) query;
  /* Test whether this particle is inside this element. */
  particle_is_inside_element =
    t8_forest_element_point_inside (forest, ltreeid, element, tree_vertices,
                                    particle->coordinates, tolerance);
  if (particle_is_inside_element) {
    if (is_leaf) {
      /* The particle is inside and this element is a leaf element.
       * We mark the particle for being inside the partition and we increase
       * the particles_per_element counter of this element. */
      /* In order to find the index of the element inside the array, we compute the
       * index of the first element of this tree plus the index of the element whithin
       * the tree. */
      t8_locidx_t         element_index =
        t8_forest_get_tree_element_offset (forest, ltreeid) + tree_leaf_index;
      particle->is_inside_partition = 1;
      *(double *) t8_sc_array_index_locidx (particles_per_element,
                                            element_index) += 1;
    }
    /* The particles is inside the element. This query should remain active.
     * If this element is not a leaf the search will continue with its children. */
    user_data->count++;
    return 1;
  }
  /* The particle is not inside the element. Deactivate this query.
   * If no active queries are left, the search will stop for this element and its children. */
  return 0;
```

## The actual search

All that is left to do for us now is to initialize our user data and start the search:

```C++
  
  t8_tutorial_search_user_data_t user_data;

  /* Initialize the particles per element as an array of one double per local element. */
  sc_array_init_count (user_data.particles_per_element, sizeof (double),
                       num_local_elements);
  /* Set each entry to 0 */
  for (ielement = 0; ielement < num_local_elements; ++ielement) {
    *(double *) t8_sc_array_index_locidx (user_data.particles_per_element, ielement) = 0;
  }
  /* Intialize the number of searched elments to 0. */
  user_data.num_elements_searched = 0;
  /* Set the forest's user data pointer. */
  t8_forest_set_user_data (forest, &user_data);
```

To perform the search we call `t8_forest_search`:
```C++
  /* Perform the search of the forest. The second argument is the search callback function,
   * then the query callback function and the last argument is the array of queries. */
  t8_forest_search (forest, t8_tutorial_search_callback,
                    t8_tutorial_search_query_callback, particles);
```