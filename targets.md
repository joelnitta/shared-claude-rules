---
description: Use when doing data analysis in R with the {targets} package
paths:
  - "**/_targets.R"
---

# R {targets} project instructions 

## General

- This project uses R and the `{targets}` package for workflow management.
- When generating code, prefer functions that are easy to use inside `{targets}` pipelines.
- Avoid unnecessary side effects in functions.
- Prefer clear, modular target definitions.
- Keep code concise and readable.

## Organization

- Put pipeline definitions in `_targets.R`.
- Put all functions in `R/functions.R`, even if the function is used somewhere else than the main pipeline
- Put all package load calls in `R/packages.R` then source this file (do not use the `packages` argument of `tar_option_set`)
- Targets and globals must have unique names. In other words, don't use the same name for a function and a target.

## Multi-workflow projects

- For complicated projects with multiple workflows, read the guidance at <https://books.ropensci.org/targets/projects.html> first
- For the main analysis, use _targets.R
- For other workflows (e.g., data preparation, supplementary or exploratory analyses), use an appropriately named R script, e.g. _targets_data.R 
- Use the _targets.yaml to configure the setup

## Documentation

- Add roxygen-style comments for every function in `R/functions.R`.
- Add comments in the `_targets.R` file to make it easy to understand the pipeline logic
- For section headers, format like this (four dashes after the header) `# header ----`
- Section headers in comments should match between `_targets.R` and `R/functions.R`

## Packages

- Use the {tarchetypes} package to help keep code readable
  - Use `target = command` syntax when possible as it is easier to read than `tar_target()`
  - For example use, tarchetypes::tar_file_read() to load data from files
    - In `tar_file_read()`, write the read step with unquoting, e.g.
      `read_record_raw(!!.x)`
  - use `tar_plan()` instead of `list()` to define the pipeline
  - For Quarto reports, use `tar_quarto()` to render documents
    - Example: `tar_quarto(report_name, "path/to/report.qmd")`
    - Do not create custom functions to wrap `quarto::quarto_render()`

## Code Patterns

- **Never use curly braces `{}` inside `tar_target()` calls**. Instead, create custom functions.
  - BAD: `tar_target(output_file, { ggsave("plot.png", plot = my_plot); "plot.png" }, format = "file")`
  - GOOD: Create `save_plot <- function(plot, path) { ggsave(path, plot = plot); path }` in R/functions.R, then use `tar_target(output_file, save_plot(my_plot, "plot.png"), format = "file")`
  - Rationale: Functions are testable, reusable, easier to debug, and keep pipeline code clean and declarative