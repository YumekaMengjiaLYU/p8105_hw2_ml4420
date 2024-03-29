p8105\_hw2\_ml4420
================
Mengjia Lyu
2019-9-26

## Problem 1

``` r
trash_data = read_excel("./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", sheet = "Mr. Trash Wheel", range = cell_cols("A:N"), 
                        col_names = TRUE, skip = 1) %>%
                        janitor::clean_names() %>%
                        na.omit()
# omit rows that do not include dumpster-specific data


# round the number of sports balls to the nearest integer
trash_data$sports_balls = as.integer(round(trash_data$sports_balls))
```

``` r
# read and clean precipitation data for 2017 and 2018
precip_data_2017 = read_excel("./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", sheet = "2017 Precipitation", col_names = TRUE, skip = 1) %>%
                              janitor::clean_names() %>%
                              na.omit() %>%
                              mutate(year = 2017)
 
precip_data_2018 = read_excel("./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", sheet = "2018 Precipitation", col_names = TRUE, skip = 1) %>%
                              janitor::clean_names() %>%
                              na.omit() %>%
                              mutate(year = 2018)

# combine the two datasets
precip_data = full_join(precip_data_2017, precip_data_2018, by = NULL) %>%
                        mutate(month = month.name[month])
```

    ## Joining, by = c("month", "total", "year")

## Comments

The number of observations in the Mr. Trash Wheel dataset is 344. The
number of observations in the precipitation dataset is 24. Key variables
in the Mr. Trash Wheel dataset include **dumpster, month, year, date,
weight\_tons, volume\_cubic\_yards, plastic\_bottles, polystyrene,
cigarette\_butts, glass\_bottles, grocery\_bags, chip\_bags,
sports\_balls, homes\_powered**. Key variables in the precipitation
dataset include **month, total, year**. Total precipitation in 2018 is
70.33. The median number of sports balls in a dumpster in 2017 is
8.

## Problem 2

``` r
pols_month_data = read_csv("./data/fivethirtyeight_datasets/pols-month.csv") %>%
                  janitor::clean_names() %>%
                  separate(mon, into = c("year", "month", "day"), sep = "-", convert = TRUE) %>%    #break up mon into integer variables
                  mutate(month = month.name[month]) %>%                                             #replace month number with month name
                  mutate(president = ifelse(prez_dem == 0, "gop", "dem")) %>%                       #create new variable president
                  select(-prez_dem) %>%                                                             #remove prez_dem, prez_gop, day
                  select(-prez_gop) %>%
                  select(-day)
```

    ## Parsed with column specification:
    ## cols(
    ##   mon = col_date(format = ""),
    ##   prez_gop = col_double(),
    ##   gov_gop = col_double(),
    ##   sen_gop = col_double(),
    ##   rep_gop = col_double(),
    ##   prez_dem = col_double(),
    ##   gov_dem = col_double(),
    ##   sen_dem = col_double(),
    ##   rep_dem = col_double()
    ## )

``` r
snp_data = read_csv("./data/fivethirtyeight_datasets/snp.csv") %>%
           janitor::clean_names() %>%
           separate(date, into = c("month", "day", "year"), sep = "/", convert = TRUE) %>%          #arrange according to year and month
           mutate(month = month.name[month]) %>% 
           select(year, month, everything()) %>%
           select(-day)
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_character(),
    ##   close = col_double()
    ## )

``` r
unemployment_data = read_csv("./data/fivethirtyeight_datasets/unemployment.csv") %>%
                    janitor::clean_names() 
```

    ## Parsed with column specification:
    ## cols(
    ##   Year = col_double(),
    ##   Jan = col_double(),
    ##   Feb = col_double(),
    ##   Mar = col_double(),
    ##   Apr = col_double(),
    ##   May = col_double(),
    ##   Jun = col_double(),
    ##   Jul = col_double(),
    ##   Aug = col_double(),
    ##   Sep = col_double(),
    ##   Oct = col_double(),
    ##   Nov = col_double(),
    ##   Dec = col_double()
    ## )

``` r
colnames(unemployment_data) = c("year", "January", "February", "March", "April", "May", "June",
                                   "July", "August", "September", "October", "November", "December")

  
unemployment_tidy_data = 
  pivot_longer(
    unemployment_data,
    January:December,
    names_to = "month",
    values_to = "percentage_of_unemployment")

#merge snp into pols
snp_pols_data = 
  left_join(pols_month_data, snp_data, by = c("year", "month") )

#merge unemployment into the result
snp_pols_unemploy_data = 
  left_join(snp_pols_data, unemployment_tidy_data, by = c("year", "month"))
```

## Comments

Dataset *pols\_month\_data* contains observations of 7 variables related
to the number of senators, governors and representatives who are
democratic or republican at any given time. Dataset *snp\_data* contains
observations of the date and the closing values of the Standard & Poor’s
stock index on the associated date. Dataset *unemployment\_data*
contains observations of the percentage of unemployment in each month of
the associated year. Dataset *unemployment\_tidy\_data* is the
lengthened and tidied version of dataset *unemployment\_data*. Dataset
*snp\_pols\_data* contains the observations from both dataset
*pols\_month\_data* and dataset *snp\_data*.

The **resulting dataset** *snp\_pols\_unemploy\_data* contains the
observations from all of dataset *pol\_month\_data*, dataset *snp\_data*
and dataset *unemployment\_tidy\_data*. It has 822 rows and 11 columns.
Range of years is from 1947 to 2015. Names of key variables include
year, month, gov\_gop, sen\_gop, rep\_gop, gov\_dem, sen\_dem, rep\_dem,
president, close, percentage\_of\_unemployment.

## Problem 3

``` r
#library(tools)
baby_names_data = read_csv("./data/Popular_Baby_Names.csv") %>%
                  janitor::clean_names() %>%
                  mutate(gender = str_replace(gender, "FEMALE", "Female")) %>%
                  mutate(gender = str_replace(gender, "MALE", "Male")) %>%
                  mutate(childs_first_name = str_to_lower(childs_first_name)) %>%
                  mutate(ethnicity = str_to_lower(ethnicity))  
```

    ## Parsed with column specification:
    ## cols(
    ##   `Year of Birth` = col_double(),
    ##   Gender = col_character(),
    ##   Ethnicity = col_character(),
    ##   `Child's First Name` = col_character(),
    ##   Count = col_double(),
    ##   Rank = col_double()
    ## )

``` r
# change ethnicity to proper format
baby_names_data$ethnicity = str_replace_all(baby_names_data$ethnicity, "paci$", "pacific islander")
baby_names_data$ethnicity = str_replace_all(baby_names_data$ethnicity, "hisp$", "hispanic")


# remove duplicate rows
distinct(baby_names_data)                                                   
```

    ## # A tibble: 12,181 x 6
    ##    year_of_birth gender ethnicity              childs_first_na… count  rank
    ##            <dbl> <chr>  <chr>                  <chr>            <dbl> <dbl>
    ##  1          2016 Female asian and pacific isl… olivia             172     1
    ##  2          2016 Female asian and pacific isl… chloe              112     2
    ##  3          2016 Female asian and pacific isl… sophia             104     3
    ##  4          2016 Female asian and pacific isl… emily               99     4
    ##  5          2016 Female asian and pacific isl… emma                99     4
    ##  6          2016 Female asian and pacific isl… mia                 79     5
    ##  7          2016 Female asian and pacific isl… charlotte           59     6
    ##  8          2016 Female asian and pacific isl… sarah               57     7
    ##  9          2016 Female asian and pacific isl… isabella            56     8
    ## 10          2016 Female asian and pacific isl… hannah              56     8
    ## # … with 12,171 more rows

## Plotting

``` r
library(arsenal)

olivia_data = baby_names_data[which(baby_names_data$childs_first_name == "olivia"), ] %>%
              select(-gender) %>%
              select(-childs_first_name) %>%
              select(-count) %>%
              distinct()

olivia_tidy_data = pivot_wider(
  olivia_data,
  names_from = year_of_birth,
  values_from = rank            # rank in popularity of the name "Olivia" over time    
)

# create a reader-friendly table

knitr::kable(olivia_tidy_data)
```

| ethnicity                  | 2016 | 2015 | 2014 | 2013 | 2012 | 2011 |
| :------------------------- | ---: | ---: | ---: | ---: | ---: | ---: |
| asian and pacific islander |    1 |    1 |    1 |    3 |    3 |    4 |
| black non hispanic         |    8 |    4 |    8 |    6 |    8 |   10 |
| hispanic                   |   13 |   16 |   16 |   22 |   22 |   18 |
| white non hispanic         |    1 |    1 |    1 |    1 |    4 |    2 |

``` r
# most popular name among male children over time
most_ppl_male_name_data = baby_names_data[which(baby_names_data$gender == "Male" &
                                                baby_names_data$ethnicity == "white non hispanic" &
                                                baby_names_data$year_of_birth == 2016), ] %>%
                                          select(-year_of_birth) %>%
                                          select(-gender) %>%
                                          select(-ethnicity)
                                          
## scatterplot
ggplot(most_ppl_male_name_data, aes(x = rank, y = count)) + geom_point() + 
       labs(title = "Name Statistics of White Non-Hispanic Male Children Born in 2016",
            x = "rank in popularity of a name",
            y = "number of children with a name")
```

![](hw2_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->
