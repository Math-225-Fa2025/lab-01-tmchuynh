# Lab 01 – Hello R!

## Repository Structure

```         
.
├── lab-01-hello-r.Rmd # Main R Markdown lab file
├── lab-01-hello-r.html # Knitted HTML output (generated)
├── img/ # Images/screenshots used in instructions
├── README.md # This file
└── lab.css # Custom CSS for tufte HTML output
```

------------------------------------------------------------------------

## Exercises Overview

-   **Exercise 1**: Inspect the `datasaurus_dozen` dataset (rows, columns, variables).
-   **Exercise 2**: Plot and calculate correlation for the `dino` dataset.
-   **Exercise 3**: Plot and calculate correlation for the `star` dataset.
-   **Exercise 4**: Plot and calculate correlation for the `circle` dataset.
-   **Final Task**: Facet all datasets and compare correlations across them.

------------------------------------------------------------------------

## Correlation Log

In addition to the required exercises, this lab includes an **extended workflow** for tracking and visualizing correlations across datasets:

-   A global `correlation_log` tibble is initialized to **store correlation values incrementally**.
-   Each time you compute a correlation for a dataset, you can append it to `correlation_log` using the helper function `add_corr`:

``` r
add_corr <- function(df, dataset_label, method = "pearson") {
  r_val <- cor(df$x, df$y, method = method, use = "complete.obs")
  new_row <- tibble::tibble(
    step = if (nrow(corr_log) == 0) 1L else max(corr_log$step) + 1L,
    dataset = dataset_label,
    r = r_val,
    n = nrow(df),
    method = method,
    time = Sys.time()
  )
  corr_log <<- dplyr::bind_rows(corr_log, new_row)
  new_row
}
```

**Example Usage**:

``` r
dino_data  <- datasaurus_dozen %>% filter(dataset == "dino")
star_data  <- datasaurus_dozen %>% filter(dataset == "star")
circle_data <- datasaurus_dozen %>% filter(dataset == "circle")

add_corr(dino_data, "dino")
add_corr(star_data, "star")
add_corr(circle_data, "circle")
```

**To append all missing datasets at the end:**

``` r
all_corrs <- datasaurus_dozen %>%
  group_by(dataset) %>%
  summarize(r = cor(x, y), .groups = "drop") %>%
  mutate(
    step = if (nrow(corr_log) == 0) row_number() else max(corr_log$step) + row_number(),
    n = nrow(datasaurus_dozen) / n_distinct(datasaurus_dozen$dataset),
    method = "pearson",
    time = Sys.time()
  )

corr_log <- bind_rows(corr_log, all_corrs)
```

**Visualize the correlation log:**

``` r
ggplot(corr_log, aes(x = step, y = r, label = dataset)) +
  geom_line() +
  geom_point(size = 2) +
  geom_text(vjust = -0.8, size = 3) +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(
    title = "Correlation log across Datasaurus datasets",
    x = "Step / Dataset",
    y = "Pearson r"
  )
```

## What a correlation tells us

**Correlation (Pearson’s r)** measures the *strength* and *direction* of a linear relationship between two variables.

Values range from **−1 to +1**:

-   **+1** → perfect positive linear relationship (as x increases, y increases proportionally).
-   **0** → no linear relationship (x doesn’t predict y in a straight-line sense).
-   **−1** → perfect negative linear relationship (as x increases, y decreases proportionally).

**Example**: if you had test scores and study hours, a correlation near +0.8 would mean “more hours tends to mean higher scores.”

## What the difference in correlations tells us

Comparing correlations across datasets (like **dino**, **star**, **circle**) shows whether the *strength or direction* of linear association changes.

In the Datasaurus datasets, the correlations are **nearly identical (\~ −0.06)** even though the scatterplots are visually *completely different*.
That means the **correlation value alone does not capture the full story** — two very different data shapes can yield the same r.

The “difference” in correlation is meaningful only if the underlying patterns *really differ in linearity*.

**Example**: comparing r = 0.9 vs r = 0.2 tells you that one dataset has a strong linear relationship, while the other has a weak one.

But when all r values are similar (like here), it highlights that **summary stats can be misleading without visualization**.

------------------------------------------------------------------------

## Key Packages Used

-   [`tidyverse`](https://www.tidyverse.org/) – data manipulation & plotting
-   [`datasauRus`](https://cran.r-project.org/package=datasauRus) – datasets used in this lab
-   [`knitr`](https://yihui.org/knitr/) – knitting R Markdown to HTML/PDF

------------------------------------------------------------------------

## Learning Outcomes

-   Understand how to structure and knit R Markdown documents.
-   Know how to commit and push to GitHub using RStudio.
-   Appreciate why data visualization is critical, even when correlations are the same.

------------------------------------------------------------------------

## References

-   Matejka, J., & Fitzmaurice, G. (2017). *Same stats, different graphs: Generating datasets with varied appearance and identical statistics through simulated annealing.*
-   Alberto Cairo’s [original Datasaurus post](https://www.thefunctionalart.com/).

------------------------------------------------------------------------

💡 **Tip**: Always knit before you commit!
This ensures your R Markdown runs cleanly and produces the expected output.
