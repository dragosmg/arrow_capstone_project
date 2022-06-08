
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Capstone project proposal

<!-- badges: start -->
<!-- badges: end -->

The goal of arrow_capstone_project is to …

## Summary

improve user and developer experience,

by: allowing them to use namespacing, supply their own function, access
documentation for existing bindings, improved error messaging.

## Scope

Jira tickets:

1.  [ARROW-14575](https://issues.apache.org/jira/browse/ARROW-14575):
    allow bindings to use the `pkg::` prefixes

-   might enable 3
-   **Steps**:
    -   Register each binding twice in the `nse_funcs` function registry
        (once as `fun()` and once as `pkg::fun()`).
    -   There are still some edge cases that need to be sorted, such as
        some of the unary functions (which are defined as a list which
        is used for several purposes)
-   **Estimated time**: 3-4 days
-   **Definition of done**:
    -   users being able to correctly use a namespaced version of *all*
        our bindings, making, for example, the following snippet of code
        work:

    ``` r
    nycflights13::flights %>% 
      arrow_table() %>% 
      mutate(year = lubridate::year(time_hour)) %>% 
      collect()
    ```

    -   or being able to use the {lubridate} version of `date()` in the
        snippet below:

``` r
  test_df <- tibble(
    posixct_date = as.POSIXct(c("2012-03-26 23:12:13", NA), tz = "America/New_York"),
    integer_var = c(32L, NA)
  )
  
  test_df %>% 
    arrow_table() %>% 
    mutate(a_date = lubridate::date(posixct_date)) %>%
    collect()
```

2.  [ARROW-14071](https://issues.apache.org/jira/browse/ARROW-14071):
    allow users to `arrow_eval` a function

-   there are several possible directions:
    -   
-   **Steps**:
    -   Translate the user-defined function with the help of bindings
        -   Give users access to existing bindings
        -   Translate user defined functions with the help of the
            existing bindings
    -   A second step might be accessing / registering these functions
        -   Where is the function? Global environment, other script that
            is being sourced or a package
        -   If registering is required, order of namespace loading /
            attaching might be important
-   **Estimated duration**: 2 weeks
-   **Definition of done**: users will be able to define and use their
    own bindings in {dplyr} - like pipelines, as long as they only
    contain bindings that have already been implemented, or direct calls
    to libarrow kernels. Users would be able to write their own version
    of `nchar` for example and that would work:

``` r
nchar2 <- function(x) {
  nchar(x)
}
tibble::tibble(my_string = "1234") %>%
  arrow_table() %>% 
  mutate(nchar(my_string), nchar2(my_string)) %>%
  collect()
```

3.  [ARROW-15011](https://issues.apache.org/jira/browse/ARROW-15011):
    (semi) automatically document a binding

-   Likely depends on ticket 1 (removing any ambiguity with regards to
    the function we’re binding to)
-   This will allow us to document differences in behaviours,
    unsupported arguments, etc.
-   I think this could be really useful for users. How would the access
    said documentation?
-   As a stretch we might include the version of the function we’re
    binding to. Functions / packages are not static and convergence
    between the original function and the arrow binding might drift over
    time.
-   **Steps**:
    -   create a (manual) prototype of what a documented binding would
        look like
    -   decide on how the user will access the documentation
    -   might involve writing custom roxygen2 tags and roclets
-   **Estimated duration**
    -   given the rough documentation on how to define custom roxygen2
        extensions, I anticipate this will take extra time to understand
        the roxygen2 source code.
    -   rough estimate: 3-4 weeks
-   **Definition of done**: bindings are documented based on the
    function they emulate.

4.  [ARROW-13370](https://issues.apache.org/jira/browse/ARROW-13370):
    more special handling for known errors in `arrow_eval`

-   This is related to, but not dependent on 3.
-   Maybe expand the scope to include more precise messaging, in cases
    where a part of a more complex call fails to pinpoint to the exact
    location / reason for the failure.
-   Components:
    -   we could opt for custom error messages when an argument is not
        supported, for example
    -   we could also warn when there is a difference between the
        binding and the original function (e.g. different default
        values, slight differences in implementation, etc.)
-   **Steps**:
-   **Estimated duration**:
-   **Definition of done**:

5.  [ARROW-15016](https://issues.apache.org/jira/browse/ARROW-15016):
    `show_query()` for `arrow_dplyr_query`

-   this would be useful for debugging / allowing users to inspect a
    query plan
-   **Steps**
    -   implement `$ToString` and `print()` methods for ExecPlan
    -   wire them up to a new function, `show_arrow_query()`
-   **Definition of done**: being able to print / inspect a query plan

6.  [ARROW-12322](https://issues.apache.org/jira/browse/ARROW-12322):
    work around masking of data type functions

-   **Steps**
-   **Definition of done**

## Extension

7.  Allow users to explore existing bindings and the differences between
    them and the functions they aim to replace

-   no Jira ticket yet
-   I’m thinking of an interface, maybe shiny that allows a use to see ,
    for example, what lubridate functionality is currently supported in
    Arrow and maybe explore the differences between the `fun` binding
    and `lubridate::fun`.

8.  [ARROW-14855](https://issues.apache.org/jira/browse/ARROW-14855):
    `build_expr()` should check that non-expression inputs have
    `vec_size() == 1L`

-   This is a bit of an extra ticket, not strictly related to the ones
    above.
