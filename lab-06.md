Lab 06 - Ugly charts and Simpson’s paradox
================
Noah Booker
3/28/25

# Take a Sad Plot and Make It Better

## Instructional staff employment trends

### Load packages and data

``` r
library(tidyverse) 
library(scales)
#library(dsbox)
#library(mosaicData) 
```

``` r
staff <- read_csv("data/instructional-staff.csv")
```

    ## Rows: 5 Columns: 12
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (1): faculty_type
    ## dbl (11): 1975, 1989, 1993, 1995, 1999, 2001, 2003, 2005, 2007, 2009, 2011
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
staff_long <- staff %>%
  pivot_longer(cols = -faculty_type, names_to = "year") %>%
  mutate(value = as.numeric(value))

staff_long %>%
  ggplot(aes(x = year, y = value, color = faculty_type)) +
  geom_line()
```

    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?

![](lab-06_files/figure-gfm/codefromlabintro-1.png)<!-- -->

``` r
staff_long %>%
  ggplot(aes(
    x = year,
    y = value,
    group = faculty_type,
    color = faculty_type
  )) +
  geom_line()
```

![](lab-06_files/figure-gfm/codefromlabintro-2.png)<!-- -->

### Exercise 1

Include the line plot you made above in your report and make sure the
figure width is large enough to make it legible. Also fix the title,
axis labels, and legend label.

``` r
staff_long %>%
  ggplot(aes(
    x = year,
    y = value,
    group = faculty_type,
    color = faculty_type
  )) +
  geom_line() +
  labs(title = "Instructional Staff Employees, 1975-2011",
       x = "Year", y = "Percentage of Hires",
       color = "Faculty Type") +
  scale_y_continuous(labels = label_percent(scale = 1)) +
  theme_minimal()
```

![](lab-06_files/figure-gfm/plot1-1.png)<!-- -->

### Exercise 2

Suppose the objective of this plot was to show that the proportion of
part-time faculty have gone up over time compared to other instructional
staff types. What changes would you propose making to this plot to tell
this story?

The current plot seems to tell that story pretty well. The only line
that shows a consistent positive slope over the years is Part-Time
Faculty. But perhaps I’m overlooking some changes that could make that
fact more obvious…

Solution: I had the idea of trying to make the line for Part-Time
Faculty thicker than the other lines, and asked Claude.ai for a way to
do this. Claude helped me to create the code below which adds in a
separate geom_line layer for the dataset filtered to include just
Part-Time Faculty with the linewidth set to 1.5 (instead of the default
.5). I think this plot accomplishes the goal of emphasizing the growth
in Part-Time Faculty.

``` r
staff_long %>%
  ggplot(aes(
    x = year,
    y = value,
    group = faculty_type,
    color = faculty_type
  )) +
  geom_line() +
  geom_line(data = staff_long %>% filter(faculty_type == "Part-Time Faculty"),
          aes(x = year, y = value),
          linewidth = 1.5) + 
  labs(title = "Instructional Staff Employees, 1975-2011",
       x = "Year", y = "Percentage of Hires",
       color = "Faculty Type") +
  scale_y_continuous(labels = label_percent(scale = 1)) +
  theme_minimal()
```

![](lab-06_files/figure-gfm/plot1edited-1.png)<!-- -->

## Fisheries

### Exercise 3

Can you help the researcher improve the data visualizations he made from
the FOA data on the fisheries production of different countries? First,
brainstorm how you would improve it. Then create the improved
visualization and document your changes/decisions with bullet points.
It’s ok if some of your improvements are aspirational, i.e. you don’t
know how to implement it, but you think it’s a good idea. Implement what
you can and leave notes identifying the aspirational improvements that
could not be made. (You don’t need to recreate their plots in order to
improve them)

``` r
fisheries <- read_csv("data/fisheries.csv")
```

    ## Rows: 216 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): country
    ## dbl (3): capture, aquaculture, total
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

#### Brainstorm

I have no clue what I’m looking at with the first graph——the one with
countries on the x-axis and (what I only know from your description to
be) total harvest on the y-axis (there are no anxis labels). There’s a
blue curve and a red curve that both skyrocket on the left side (which
I’m assuming shows that China has a lot of harvest since they’re the
first contry on the x-axis). But then there’s little peaks as you move
righward along the curves, but the peaks don’t seem to correspond to
where the other countries are located on the x-axis. So, I have no clue.
The other two visualizations are 3D pie charts, one showing each
country’s proportion of the total capture tonnage, and the other showing
each country’s proportion of the total aquaculture tonnage. I learned
from Module 5 that bar charts are bad and 3D is bad. The description in
the lab says that the visualization excludes countries with less than
100,000 tons. This is probably a good call since there are 216 countries
in the dataset. It seems to me like a reasonable primary goal with
creating a visualization for this data is to be able to compare
countries’ total tonnage of fish. Second, in visualizing tonnage of fish
by country, we might also want to display how much of each country’s
total tonnage came from capture or aquaculture. Third, ignoring total
tonnage, we may be interested in comparing each country by the
proportion of their fish which came from capture or aquaculture.

#### Improved Visualization

• Let’s first get an idea of how many countries we can usefully compare.
The lab description says that the visualization excludes countries with
less than 100,000 tons. Let’s see how many countries that leaves us
with.

``` r
#fisheries %>% 
  #select(total)
```

• A barplot seems to me like the most appropriate way to compare
countries on their fishery production (much better than a pie chart or
whatever that other graph is).
