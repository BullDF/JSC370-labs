Lab 05 - Data Wrangling
================

# Learning goals

- Use the `merge()` function to join two datasets.
- Deal with missings and impute data.
- Identify relevant observations using `quantile()`.
- Practice your GitHub skills.

# Lab description

For this lab we will be dealing with the meteorological dataset `met`.
In this case, we will use `data.table` to answer some questions
regarding the `met` dataset, while at the same time practice your
Git+GitHub skills for this project.

This markdown document should be rendered using `github_document`
document.

# Part 1: Setup a Git project and the GitHub repository

1.  Go to wherever you are planning to store the data on your computer,
    and create a folder for this project.

2.  In that folder, save [this
    template](https://github.com/JSC370/JSC370-2024/blob/main/labs/lab05/lab05-wrangling-gam.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository of the same
    name that your local folder has, e.g., “JSC370-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

Most of the steps can be done using command line:

``` sh
# Step 1
cd ~/Documents
mkdir JSC370-labs
cd JSC370-labs

# Step 2
wget https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd
mv lab05-wrangling-gam.Rmd README.Rmd
# if wget is not available,
curl https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd --output README.Rmd

# Step 3
# Happens on github

# Step 4
git init
git add README.Rmd
git commit -m "First commit"

# Step 5
git remote add origin git@github.com:[username]/JSC370-labs
git push -u origin master
```

You can also complete the steps in R (replace with your paths/username
when needed)

``` r
# Step 1
setwd("~/Documents")
dir.create("JSC370-labs")
setwd("JSC370-labs")

# Step 2
download.file(
  "https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd",
  destfile = "README.Rmd"
  )

# Step 3: Happens on Github

# Step 4
system("git init && git add README.Rmd")
system('git commit -m "First commit"')

# Step 5
system("git remote add origin git@github.com:[username]/JSC370-labs")
system("git push -u origin master")
```

Once you are done setting up the project, you can now start working with
the MET data.

## Setup in R

1.  Load the `data.table` (and the `dtplyr` and `dplyr` packages),
    `mgcv`, `ggplot2`, `leaflet`, `kableExtra`.

``` r
library(data.table)
library(dtplyr)
library(dplyr)
library(mgcv)
library(ggplot2)
library(leaflet)
library(kableExtra)
```

``` r
fn <- "https://raw.githubusercontent.com/JSC370/JSC370-2024/main/data/met_all_2023.gz"
if (!file.exists("met_all_2023.gz"))
  download.file(fn, destfile = "met_all_2023.gz")
met <- data.table::fread("met_all_2023.gz")
```

2.  Load the met data from
    <https://github.com/JSC370/JSC370-2024/main/data/met_all_2023.gz> or
    (Use
    <https://raw.githubusercontent.com/JSC370/JSC370-2024/main/data/met_all_2023.gz>
    to download programmatically), and also the station data. For the
    latter, you can use the code we used during lecture to pre-process
    the stations data:

``` r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE, LAT, LON)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]

# Read in the met data and fix lat, lon, temp
```

3.  Merge the data as we did during the lecture. Use the `merge()` code
    and you can also try the tidy way with `left_join()`

``` r
met_dt <- merge(
  x = met,
  y = stations,
  by.x = "USAFID",
  by.y = "USAF",
  all.x = TRUE,
  all.y = FALSE
)
head(met_dt)
```

    ##    USAFID  WBAN year month day hour min   lat     lon elev wind.dir wind.dir.qc
    ## 1: 690150 93121 2023     6   1   NA  56 34294 -116147  696      240           5
    ## 2: 690150 93121 2023     6   1    1  56 34294 -116147  696       NA           9
    ## 3: 690150 93121 2023     6   1    2  56 34294 -116147  696      300           5
    ## 4: 690150 93121 2023     6   1    3  56 34294 -116147  696      300           5
    ## 5: 690150 93121 2023     6   1    4  56 34294 -116147  696      310           5
    ## 6: 690150 93121 2023     6   1    5  56 34294 -116147  696       20           5
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N      15          5      22000             5                 9
    ## 2:              C      NA          5      22000             5                 9
    ## 3:              N      93          5      22000             5                 9
    ## 4:              N     108          5      22000             5                 9
    ## 5:              N      72          5      22000             5                 9
    ## 6:              N      31          5      22000             5                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp temp.qc dew.point
    ## 1:        N    16093           5       N          5  289       5         6
    ## 2:        N    16093           5       N          5  289       5        22
    ## 3:        N    16093           5       N          5  261       5        50
    ## 4:        N    16093           5       N          5  239       5        61
    ## 5:        N    16093           5       N          5  233       5        67
    ## 6:        N    16093           5       N          5  222       5        67
    ##    dew.point.qc atm.press atm.press.qc CTRY STATE    LAT      LON
    ## 1:            5     10040            5   US    CA 34.294 -116.147
    ## 2:            5     10041            5   US    CA 34.294 -116.147
    ## 3:            5     10046            5   US    CA 34.294 -116.147
    ## 4:            5     10052            5   US    CA 34.294 -116.147
    ## 5:            5     10055            5   US    CA 34.294 -116.147
    ## 6:            5     10065            5   US    CA 34.294 -116.147

## Question 1: Identifying Representative Stations

Across all weather stations, which stations have the median values of
temperature, wind speed, and atmospheric pressure? Using the
`quantile()` function, identify these three stations. Do they coincide?

``` r
med_met_dt <- met_dt |>
  group_by(USAFID) |>
  summarize_at(vars(temp, wind.sp, atm.press), ~quantile(.x, 0.5, na.rm = TRUE))
head(med_met_dt)
```

    ## # A tibble: 6 × 4
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 690150  267       41     10094
    ## 2 720110  280       31        NA
    ## 3 720113  200       31        NA
    ## 4 720120  240       36        NA
    ## 5 720137  211       26        NA
    ## 6 720151  280.      36        NA

``` r
med_temp <- quantile(met_dt$temp, 0.5, na.rm = TRUE)
med_wind_sp <- quantile(met_dt$wind.sp, 0.5, na.rm = TRUE)
med_atm_press <- quantile(met_dt$atm.press, 0.5, na.rm = TRUE)
```

Next identify the stations have these median values.

``` r
med_temp_stat <- med_met_dt |>
  filter(temp == med_temp)
med_wind_sp_stat <- med_met_dt |>
  filter(wind.sp == med_wind_sp)
med_atm_press_stat <- med_met_dt |>
  filter(atm.press == med_atm_press)

head(med_temp_stat)
```

    ## # A tibble: 6 × 4
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 720263   217    26          NA
    ## 2 720312   217    26          NA
    ## 3 720327   217    46          NA
    ## 4 722076   217    36          NA
    ## 5 722180   217    31       10107
    ## 6 722196   217    28.5     10106

``` r
head(med_wind_sp_stat)
```

    ## # A tibble: 6 × 4
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 720110   280      31        NA
    ## 2 720113   200      31        NA
    ## 3 720258   180      31        NA
    ## 4 720261   277      31        NA
    ## 5 720266   185      31        NA
    ## 6 720267   200      31        NA

``` r
head(med_atm_press_stat)
```

    ## # A tibble: 6 × 4
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 720394   239      21     10117
    ## 2 722085   244      31     10117
    ## 3 722348   244      21     10117
    ## 4 723119   217      31     10117
    ## 5 723124   211      26     10117
    ## 6 723270   239      31     10117

``` r
merge(
  x = med_temp_stat,
  y = med_wind_sp_stat,
  by.x = "USAFID",
  by.y = "USAFID",
  all.x = FALSE,
  all.y = FALSE
) |> merge(
  y = med_atm_press_stat,
  by.x = "USAFID",
  by.y = "USAFID",
  all.x = FALSE,
  all.y = FALSE
)
```

    ##   USAFID temp.x wind.sp.x atm.press.x temp.y wind.sp.y atm.press.y temp wind.sp
    ## 1 723119    217        31       10117    217        31       10117  217      31
    ##   atm.press
    ## 1     10117

**Answer:** From the above merges, the station with USAFID 723119 has
median temperature, wind speed, and atmospheric pressure that match the
national medians.

Knit the document, commit your changes, and save it on GitHub. Don’t
forget to add `README.md` to the tree, the first time you render it.

## Question 2: Identifying Representative Stations per State

Now let’s find the weather stations by state with closest temperature
and wind speed based on the euclidean distance from these medians.

Knit the doc and save it on GitHub.

## Question 3: In the Geographic Center?

For each state, identify which station is closest to the geographic
mid-point (median) of the state. Combining these with the stations you
identified in the previous question, use `leaflet()` to visualize all
~100 points in the same figure, applying different colors for the
geographic median and the temperature and wind speed median.

Knit the doc and save it on GitHub.

## Question 4: Summary Table with `kableExtra`

Generate a summary table using `kable` where the rows are each state and
the columns represent average temperature broken down by low, median,
and high elevation stations.

Use the following breakdown for elevation:

- Low: elev \< 93
- Mid: elev \>= 93 and elev \< 401
- High: elev \>= 401

Knit the document, commit your changes, and push them to GitHub.

## Question 5: Advanced Regression

Let’s practice running regression models with smooth functions on X. We
need the `mgcv` package and `gam()` function to do this.

- using your data with the median values per station, first create a
  lazy table. Filter out values of atmospheric pressure outside of the
  range 1000 to 1020. Examine the association between temperature (y)
  and atmospheric pressure (x). Create a scatterplot of the two
  variables using ggplot2. Add both a linear regression line and a
  smooth line.

- fit both a linear model and a spline model (use `gam()` with a cubic
  regression spline on wind speed). Summarize and plot the results from
  the models and interpret which model is the best fit and why.

## Deliverables

- .Rmd file (this file)

- link to the .md file (with all outputs) in your GitHub repository
