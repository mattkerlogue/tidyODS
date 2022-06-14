
<!-- README.md is generated from README.Rmd. Please edit that file -->

# tidyods

<!-- badges: start -->

[![Project Status: Concept – Minimal or no implementation has been done
yet, or the repository is only intended to be a limited example, demo,
or
proof-of-concept.](https://www.repostatus.org/badges/latest/concept.svg)](https://www.repostatus.org/#concept)
<!-- badges: end -->

`{tidyods}` is an R package to import cells from an OpenDocument
Spreadsheet (ODS) file, and provide information about the cell’s
postion, value types and formulas, and provide methods to “rectify”
cells back to a 2-dimensional data.frame (specifically as a tidyverse
tibble).

## Installation

You can install the development version of `{tidyods}` like so:

``` r
remotes::install_github("mattkerlogue/tidyods")
```

## Usage

The `{tidyods}` package exports the following functions for users:

-   `read_ods_cells(path, sheet, quick = FALSE)`: extract the cells from
    a sheet in an ODS file
-   `read_ods_sheet(path, sheet, rectify = "simple", base_values = TRUE, quick = FALSE)`:
    extract cells from an ODS sheet and return as a rectangular dataset,
    like a spreadsheet
-   `ods_sheets(path)`: list the sheets in an ODS fie
-   `simple_rectify(path)`: “rectify” a set of cells into a rectangular
    dataset, like a spreadsheet

## Performance

An ODS file is a zipped collection of XML files and associated files.
`{tidyods}`, like the `{readODS}` package, uses the
[xml2](https://xml2.r-lib.org) package to process this file. There are
three likely sources of performance issues: downloading of remote files,
unzipping the ODS file, processing rows.

With default settings, performance of `{tidyods}` functions is slower
than `{readODS}`, however `{tidyods}` provides console messages and
progress bars to the user.

Performance can be improved by setting `quick = FALSE` in
`read_ods_cells()` and `read_ods_sheet()`, this will extract only a
simple text representation of the cell, i.e. it will not provide
information on cell or value types, formulas or the underlying base
value for numbers, dates or times. This method also ignores replicated
white space characters.

To test performance we will use an ODS file published by the UK
Government on the [number of civil servants by
postcode](https://www.gov.uk/government/statistics/number-of-civil-servants-by-postcode-department-responsibility-level-and-leaving-cause-2021)
(5,544 rows by 11 columns). Analysis of read times indicates that using
the `quick = TRUE` setting for `read_ods_cells()` results in a read that
is 10 times faster than the full cell extraction method and around 1.5
times faster than reading the sheet with `{readODS}`.

``` r
microbenchmark::microbenchmark(
  "tidyods_slow" = tidyods::read_ods_cells(test_ods, "Staff_in_post", 
    quick = FALSE),
  "tidyods_quick" = tidyods::read_ods_cells(test_ods, "Staff_in_post", 
    quick = TRUE),
  "readODS" = readODS::read_ods(test_ods, "Staff_in_post"),
  times = 3
)

#> Unit: seconds
#>          expr     min      lq    mean  median      uq     max neval
#>  tidyods_slow 35.4833 35.7546 35.8626 36.0259 36.0523 36.0786     3
#> tidyods_quick  3.2470  3.2584  3.2935  3.2697  3.3167  3.3638     3
#>       readods  5.1141  5.1606  5.2050  5.2072  5.2505  5.2939     3
```

A cursory test of a huge file that is known to be slow/non-performative
with {readODS} shows promise.

Performance issues are tracked and discussed in [issue
\#3](https://github.com/mattkerlogue/tidyods/issues/3) in the package’s
GitHub repository.

However, if performance is a major determinant of your workflow
(i.e. you have very large files), its likely more sensible to open the
file in a spreadsheet application, save the file as a `.xslx` format
file and read using `{tidyxl}`.

## Related projects

The `{tidyODS}` package is heavily inspired by two existing packages:

-   The [`{readODS}`](https://github.com/chainsawriot/readODS) package
    provides functionality to read and write ODS files with R, with the
    resulting data.frame reflecting the way the data is viewed in a
    spreadsheet editor application (i.e a two-dimension table
    structure).
-   The [`{tidyxl}`](https://nacnudus.github.io/tidyxl/) package reads
    cells from Excel files and provides a data.frame where each cell is
    its own row and columns provide information about the cell’s
    location, value and value type, formula and formatting.
-   The [`{unpivotr}`](https://nacnudus.github.io/unpivotr/) package
    works with datasets of cells (such as those created by tidyxl).

## Philosophy

The `{readODS}` package is the only package on CRAN that handles the
reading of ODS files, whereas there are more than 20 packages (as at 12
June 2022) on CRAN that work with Excel files (of which `{tidyxl}` is
one). In some respects this is due to the wider usage of Excel files by
businesses, governments, academia and other organisations to publish and
share data. Various governments and international organisations are
starting to mandate the use of OpenDocument Format files for publishing
of their information.

The initial purpose of `{tidyods}` was to provide a second package to
the R ecosystem for reading ODS files, in part prompted when
encountering an error with `{readODS}` and discovering no alternative
pacakge was available. For example if you run into an error when using
the `{readxl}` package you could use easily try the`{openxlsx}` or
`{xlsx}` instead.

The conceptual code development lead to the creation of an output
dataset similar to that produced by the `{tidyxl}` package.

Thus, `{tidyods}` extracts cells from an ODS document in the same manner
as `{tidyxl}`, but includes functions to “rectify” the cells so as to
also provide an output similar to that of `{readODS}` (and many of those
that read Excel files). This is similar to the
[rectify](https://nacnudus.github.io/unpivotr/reference/rectify.html)
function in the `{unpivotr}` package.
