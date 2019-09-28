p8105\_hw2\_ml4420
================
Mengjia Lyu
2019-9-26

## Problem 1

This is an R Markdown document. Markdown is a simple formatting syntax
for authoring HTML, PDF, and MS Word documents. For more details on
using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that
includes both content as well as the output of any embedded R code
chunks within the document. You can embed an R code chunk like
this:

``` r
trash_data = read_excel("./data/HealthyHarborWaterWheelTotals2018-7-28.xlsx", sheet = "Mr. Trash Wheel", range = cell_cols("A:N"), 
                        col_names = TRUE, skip = 1) %>%
                        janitor::clean_names() %>%
                        na.omit()
# omit rows that do not include dumpster-specific data


# round the number of sports balls to the nearest integer
trash_data$sports_balls = as.integer(round(trash_data$sports_balls))
```

``` r
# read and clean precipitation data for 2017 and 2018
precip_data_2017 = read_excel("./data/HealthyHarborWaterWheelTotals2018-7-28.xlsx", sheet = "2017 Precipitation", col_names = TRUE, skip = 1) %>%
                              janitor::clean_names() %>%
                              na.omit() %>%
                              mutate(year = 2017)
 
precip_data_2018 = read_excel("./data/HealthyHarborWaterWheelTotals2018-7-28.xlsx", sheet = "2018 Precipitation", col_names = TRUE, skip = 1) %>%
                              janitor::clean_names() %>%
                              na.omit() %>%
                              mutate(year = 2018)

# combine the two datasets
precip_data = full_join(precip_data_2017, precip_data_2018, by = NULL) %>%
                        mutate(month = month.name[month])
```

    ## Joining, by = c("month", "total", "year")

\#\#Comments The number of observations in the Mr. Trash Wheel dataset
is 285. The number of observations in the precipitation dataset is 19.
Key variables in the Mr. Trash Wheel dataset include dumpster, month,
year, date, weight\_tons, volume\_cubic\_yards, plastic\_bottles,
polystyrene, cigarette\_butts, glass\_bottles, grocery\_bags,
chip\_bags, sports\_balls, homes\_powered. Key variables in the
precipitation dataset include month, total, year. Total precipitation in
2018 is 23.5. The median number of sports balls in a dumpster in 2017 is
8.
