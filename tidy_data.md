Tidy Data
================

This document will show how to tidy data.

# Pivot longer

``` r
pulse_df=
  read_sas("data/public_pulse_data.sas7bdat") |>
  janitor::clean_names()
```

``` r
pulse_tidy_df =
  pulse_df |>
  pivot_longer(
    cols= bdi_score_bl:bdi_score_12m,
    names_to= "visit",
    values_to = "bdi_score",
    names_prefix = "bdi_score_"
  ) |>
  mutate(
    visit = replace(visit, visit == "bl", "00m") 
  )|>
  relocate(id, visit)
```

He would do all that like this in one block:

``` r
pulse_df=
  read_sas("data/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  # new part:
  pivot_longer(
    cols= bdi_score_bl:bdi_score_12m,
    names_to= "visit",
    values_to = "bdi_score",
    names_prefix = "bdi_score_"
  ) |>
  # new part end
  mutate(
    visit = replace(visit, visit == "bl", "00m") 
  )|>
  relocate(id, visit)
```

Do one more example:

``` r
litters_df=
  read_csv("data/FAS_litters.csv", na= c("NA", ".", "")) |>
  janitor::clean_names()
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Gestational 0 and gestational 18, we want it to be better. we want a var
gestation with 0 and 18 and then weight that says the number.

``` r
litters_df=
  read_csv("data/FAS_litters.csv", na= c("NA", ".", "")) |>
  janitor::clean_names() |>
  pivot_longer(
    cols= gd0_weight:gd18_weight,
    names_to = "gd_time", 
    values_to = "weight"
  ) |> 
  mutate(
    gd_time = case_match(
      gd_time,
      "gd0_weight"  ~ 0,
      "gd18_weight" ~ 18
    ))
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#case match requires you to list all options and what you want it wo 
# case win also exists.
```

# pivot wider

Let’s make up an analysis result table

``` r
analysis_df=
  tibble(
    group = c("treatment", "treatment", "control", "control"),
    time = c("pre", "post", "pre", "post"),
    mean = c(4, 10, 4.2, 5)
  )
```

If you want to take a table you made to show someone else Pivot wider
for human readability

``` r
analysis_df |>
  pivot_wider(
    names_from = time,
    values_from = mean
  ) |>
  knitr::kable()
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |   10 |
| control   | 4.2 |    5 |

``` r
# use knitr:kable() if you want to show a table to anyone
```

## bind tables

``` r
fellowship_ring=
  read_excel("data/LotR_Words.xlsx", range="B3:D6") |>
  mutate(movie = "fellowship_ring")

two_towers=
  read_excel("data/LotR_Words.xlsx", range="F3:H6") |>
  mutate(movie = "two_towers")

return_king=
  read_excel("data/LotR_Words.xlsx", range="J3:L6") |>
  mutate(movie = "return_king")

lotr_df = 
  bind_rows(fellowship_ring, two_towers, return_king) |>
#still not tidy
  janitor::clean_names() |>
  pivot_longer(
    cols= female:male,
    names_to = "sex",
    values_to = "words"
  ) |>
  relocate(movie) |>
  mutate(race = str_to_lower(race))
```

\#join FAS datasets

Import `litters` dataset

``` r
litters_df=
  read_csv("data/FAS_litters.csv", na=c("NA", ".", "")) |>
  janitor::clean_names() |>
  mutate(
    wt_gain = gd18_weight- gd0_weight
  ) |>
  separate(
    group, into=c("dose", "day_of_treatment"), sep = 3
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Import pups next!

``` r
pups_df=
  read_csv("data/FAS_pups.csv", na=c("NA", ".", "")) |>
  janitor::clean_names() |>
  mutate(
    sex = 
      case_match(
      sex,
      1 ~ "male",
      2 ~ "female"
    )
  )
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

join the data sets!

``` r
fas_df=
  left_join(pups_df, litters_df, by = "litter_number") |>
  relocate(litter_number, dose)
```
