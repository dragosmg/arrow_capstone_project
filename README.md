
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

-   I have done some work on this, but I thought I’d add it to the list
    since it adds important functionality and will also enable other
    tickets (such as 3).
-   The design I opted for, so far, was to register bindings twice, once
    with the name and once with the namespace (`pkg::`) prefix.
-   There are still some edge cases that need to be sorted, such as some
    of the unary functions (which are defined as a list which is used
    for several purposes)
-   **Definition of done**: users being able to use a namespaced version
    of all our bindings, making, for example, the following snippet of
    code work:

``` r
nycflights13::flights %>% 
  arrow_table() %>% 
  mutate(year = lubridate::year(time_hour)) %>% 
  collect()
```

2.  [ARROW-14071](https://issues.apache.org/jira/browse/ARROW-14071):
    allow users to `arrow_eval` a function

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

-   Likely depends on ticket 1
-   This will allow us to document differences in behaviours,
    unsupported arguments, etc.
-   I think this could be really useful for users (?). How would the
    access said documentation?
-   **Steps**:
-   **Definition of done**

4.  [ARROW-13370](https://issues.apache.org/jira/browse/ARROW-13370):
    more special handling for known errors in `arrow_eval`

-   This could be related to 3 since we could opt for custom error
    messages when an argument is not supported, for example
-   We could also warn when there is a difference between the binding
    and the original function (e.g. different default values, slight
    differences in implementation, etc.)
-   **Steps**
-   **Definition of done**

5.  [ARROW-15016](https://issues.apache.org/jira/browse/ARROW-15016):
    `show_query()` for `arrow_dplyr_query`

-   **Steps**
-   **Definition of done**

6.  [ARROW-12322](https://issues.apache.org/jira/browse/ARROW-12322):
    work around masking of data type functions

-   **Steps**
-   **Definition of done**

7.  [ARROW-14855](https://issues.apache.org/jira/browse/ARROW-14855):
    `build_expr()` should check that non-expression inputs have
    `vec_size() == 1L`

-   This is a bit of an extra ticket, not strictly related to the ones
    above.
