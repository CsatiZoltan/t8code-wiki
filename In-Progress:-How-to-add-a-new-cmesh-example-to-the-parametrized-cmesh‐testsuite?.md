1. Create a new file in `test/t8_cmesh_generator/t8_cmesh_parametrized_examples/`
2. Create an `std::function` wrapper around the cmesh example that you want to use
3. (Optional) If not already there, create the vector of values of your paramter-type you want to test over in `t8_cmesh_params.hxx`
4. Create a function that prints the parameters of your example
5. Create a `std::function` wrapper around your print-function
6. Use the `cmesh_cartestion_product_params` to generate all combinations of your parameters
7. Add your new `cmesh_cartesian_product_params` object to the `cart_prod_vec` in `t8_cmesh_example_sets.hxx`