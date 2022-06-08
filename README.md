
-   [1 Apache Arrow capstone project](#1-apache-arrow-capstone-project)
-   [2 Core](#2-core)
    -   [2.1 Allow bindings to use the `pkg::`
        prefixes](#21-allow-bindings-to-use-the-pkg-prefixes)
    -   [2.2 Allow users to `arrow_eval` a
        function](#22-allow-users-to-arrow_eval-a-function)
    -   [2.3 Document a binding (semi)
        automatically](#23-document-a-binding-semi-automatically)
    -   [2.4 More specific handling for known errors in
        `arrow_eval`](#24-more-specific-handling-for-known-errors-in-arrow_eval)
    -   [2.5 Allow users to inspect ExecPlans (`show_query()` for
        `arrow_dplyr_query`)](#25-allow-users-to-inspect-execplans-show_query-for-arrow_dplyr_query)
    -   [2.6 Work around masking of data type
        functions](#26-work-around-masking-of-data-type-functions)
    -   [2.7 Document all of the above in user / developer facing
        documentation](#27-document-all-of-the-above-in-user--developer-facing-documentation)
    -   [2.8 Write a blog post](#28-write-a-blog-post)
-   [3 Stretch goals](#3-stretch-goals)
    -   [3.1 Allow users to explore existing
        bindings](#31-allow-users-to-explore-existing-bindings)
    -   [3.2 Guard `build_expr` against non-expression inputs longer
        than
        1](#32-guard-build_expr-against-non-expression-inputs-longer-than-1)
    -   [3.3 Throw helpful errors on bad object types in dplyr
        expressions](#33-throw-helpful-errors-on-bad-object-types-in-dplyr-expressions)

<!-- README.md is generated from README.Rmd. Please edit that file -->

# 1 Apache Arrow capstone project

<!-- badges: start -->
<!-- badges: end -->

Implementing the changes below would considerably improve user and
developer experience. They will remove ambiguity when calling a binding
(by allowing users to use the `pkg::` prefix), they will make it easier
to figure out what is going on when things go wrong (by adding a way to
inspect the Arrow query being generated in a dplyr pipeline, and by
allowing more granular condition handling/ messaging). Moreover, user
will be able to include their own functions (using regular R syntax),
operating on Arrow data, in a dplyr pipeline. Users will also have
access to a minimal documentation for each binding, point to the
function the binding is emulating and highlighting possible differences.

# 2 Core

## 2.1 Allow bindings to use the `pkg::` prefixes

-   Jira:
    [ARROW-14575](https://issues.apache.org/jira/browse/ARROW-14575)
-   users will be able to use either the function name (`fun`) or the
    namespace-qualified (`pkg::fun`) version when calling Arrow bindings
-   when defining a binding we should be using the `pkg::fun` notation
-   **Linked to**: [Document a binding (semi)
    automatically](#23-document-a-binding-semi-automatically)
-   **Steps**:
    -   Register each binding twice in the `nse_funcs` function registry
        (once as `fun()` and once as `pkg::fun()`).
    -   Sort out edge, e.g for some of the unary functions (which are
        defined as a list which is used for several purposes)
    -   document (where?) that going forward we should be using
        `pkg::fun` when defining a binding, which will register 2 copies
        of the same binding.
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

    -   and being able to use the {lubridate} version of `date()` in the
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

## 2.2 Allow users to `arrow_eval` a function

-   Jira:
    [ARROW-14071](https://issues.apache.org/jira/browse/ARROW-14071)
-   users would be able to define their functions using regular R
    syntax, and, as long as a direct translation of the internals is
    possible with existing bindings, they would be able to use them in
    an {arrow} dplyr pipeline.
-   there are several possible directions:
    -   will users need to register their own functions or would this be
        done automatically?
    -   when does the registration take place? before or after loading
        arrow - maybe use `setHook()` to register function before
        attaching {arrow}
    -   could we make use of {rlang}’s top and bottom of data mask?
    -   investigate if `rlang::inject()` could be used here
-   **Steps**:
    -   Translate the user-defined function with the help of bindings
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

## 2.3 Document a binding (semi) automatically

-   Jira:
    [ARROW-15011](https://issues.apache.org/jira/browse/ARROW-15011)
-   Likely depends on [Allow bindings to use the `pkg::`
    prefixes](#21-allow-bindings-to-use-the-pkg-prefixes) (removing any
    ambiguity with regards to the function we’re emulating)
-   This will allow us to document differences in behaviours,
    unsupported arguments, etc.
-   I think this could be really useful for users. How would they access
    said documentation?
-   As a *stretch* we might include the package version of the function
    we’re linking to. Functions / packages are not static and
    convergence between the original function and the arrow binding
    might drift over time.
-   **Steps**:
    -   create a (manual) prototype of what a documented binding would
        look like
    -   generate a skeleton / minimal documentation automatically
    -   allow adjustment of the minimal documentation where needed
    -   decide on how the user will access the documentation
    -   might involve writing custom {roxygen2} tags and roclets
    -   **follow-up ticket / step**: document all bindings
-   **Estimated duration**
    -   given the rough documentation on how to define custom roxygen2
        extensions, I anticipate this will take extra time due to the
        need to understand the {roxygen2} source code.
    -   rough estimate: 3-4 weeks (without documenting all the bindings)
-   **Definition of done**: bindings can be documented based on the
    function they emulate

## 2.4 More specific handling for known errors in `arrow_eval`

-   Jira:
    [ARROW-13370](https://issues.apache.org/jira/browse/ARROW-13370)
-   This is related to, but not dependent on [Document a binding (semi)
    automatically](#23-document-a-binding-semi-automatically).
-   At present we rely on the `"not supported.*Arrow"` incantation to
    identify an `arrow-try-error`, which implies that error messages
    that depart from this are not surfaced and the users never see them.
-   Maybe expand the scope of this issue / ticket to include more
    precise messaging, in cases where a part of a more complex call
    fails to pinpoint to the exact location / reason for the failure.
-   Components:
    -   we could opt for custom error messages when an argument is not
        supported, for example
    -   we could also warn when there is a difference between the
        binding and the original function (e.g. different default
        values, slight differences in implementation, etc.)
-   **Steps**:
    -   expand the classification of Arrow errors
    -   update the behaviour: allow the printing of error messages when
        they do not contain `"not supported.*Arrow"`
-   **Estimated duration**: 1-2 weeks
-   **Definition of done**: allow more detailed error messages
    (i.e. messages not containing than `"not supported.*Arrow"` can be
    classified as `arrow-try-error` (or a different, related class) and
    surfaced to inform the user).

``` r
# this error messages surfaces
register_binding("difftime", function(time1,
                                      time2,
                                      tz,
                                      units = "secs") {
    if (units != "secs") {
      abort("`difftime()` with units other than `secs` not supported in Arrow")
    }
    ...
}

# while this one would get lost in the condition handling
register_binding("difftime", function(time1,
                                      time2,
                                      tz,
                                      units = "secs") {
    if (units != "secs") {
      abort("This function errors for units other than `secs`")
    }
    ...
}
```

## 2.5 Allow users to inspect ExecPlans (`show_query()` for `arrow_dplyr_query`)

-   Jira:
    [ARROW-15016](https://issues.apache.org/jira/browse/ARROW-15016):
-   this would be useful for debugging / allowing users to inspect a
    query plan
-   **Steps**
    -   implement `$ToString` / `print()` methods for ExecPlan
    -   wire them up to a new function, `show_arrow_query()`
-   **Estimated duration**: 3-4 days
-   **Definition of done**: being able to print / inspect a query plan.
    The following snippet of code should work:

``` r
nycflights13::flights %>% 
  arrow_table() %>% 
  filter(year == 2013, month == 5, day == 27) %>% 
  show_arrow_query()
```

## 2.6 Work around masking of data type functions

-   Jira:
    [ARROW-12322](https://issues.apache.org/jira/browse/ARROW-12322)
-   several data type functions in the arrow package are named very
    generically, so they represent a large area for potential masking
    problems.
-   should be tackled before [Allow bindings to use the `pkg::`
    prefixes](#21-allow-bindings-to-use-the-pkg-prefixes)
-   this would need to handle the following scenarios:
    -   `type` is a variable in the calling environment whose value is a
        `DataType` object
    -   `type` is a call to a function defined by the user that returns
        a `DataType` object
    -   `type` is any arbitrary R expression that the user has wired up
        to return a `DataType` object
    -   `type` is a call to a function in another package or in the
        user’s environment that masks the Arrow type function of the
        same name
-   **Steps**:
-   **Definition of done**:
    -   the following code should run:

``` r
  # Works when type function is masked
  string <- rlang::string
  expect_equal(
    Array$create("abc", type = string()),
    arrow::string()
  )

  # Works when with non-Arrow function that returns an Arrow type
  # when the non-Arrow function has the same name as a base R function...
  str <- arrow::string
  expect_equal(
    Array$create("abc", type = str()),
    arrow::string()
  )

  # ... and when it has the same name as an Arrow function
  type <- arrow::string
  expect_equal(
    Array$create("abc", type = type()),
    arrow::string()
  )

  # Works with local variable whose value is an Arrow type
  type <- arrow::string()
  expect_equal(
    Array$create("abc", type = type),
    arrow::string()
  )
```

## 2.7 Document all of the above in user / developer facing documentation

## 2.8 Write a blog post

# 3 Stretch goals

I haven’t detailed these, as that could be done depending on the
progress of the work on the *core* tickets.

## 3.1 Allow users to explore existing bindings

-   and the differences between them and the functions they aim to
    replace
-   no Jira ticket yet
-   I’m thinking of an interface, maybe shiny that allows a use to see ,
    for example, what lubridate functionality is currently supported in
    Arrow and maybe explore the differences between the `fun` binding
    and `lubridate::fun`.

## 3.2 Guard `build_expr` against non-expression inputs longer than 1

-   Jira:
    [ARROW-14855](https://issues.apache.org/jira/browse/ARROW-14855)
-   `build_expr()` should check that non-expression inputs have
    `vec_size() == 1L`
-   This is a bit of an extra ticket, not strictly related to the ones
    above.

## 3.3 Throw helpful errors on bad object types in dplyr expressions

-   Jira:
    [ARROW-12093](https://issues.apache.org/jira/browse/ARROW-12093)
-   related to [More specific handling for known errors in
    `arrow_eval`](#24-more-specific-handling-for-known-errors-in-arrow_eval)
