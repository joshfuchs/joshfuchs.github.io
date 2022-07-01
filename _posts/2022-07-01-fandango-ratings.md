# Investigating Fandango Movie Ratings

You can find the full Jupter notebook on [GitHub](https://github.com/joshfuchs/DataScience_projects/blob/master/fandango_movie_ratings.ipynb).


In October 2015, a data journalist named Walt Hickey analyzed movie ratings data and found strong evidence to suggest that Fandango's rating system was biased and dishonest. He published his analysis in this [article](https://fivethirtyeight.com/features/fandango-movies-ratings/).

Fandango displays a 5-star rating system on their website, where the minimum rating is 0 stars and the maximum is 5 stars. Hickey found that there's a significant discrepancy between the number of stars displayed to users and the actual rating, which he was able to find in the HTML of the page. He was able to find that:

- The actual rating was almost always rounded up to the nearest half-star. For instance, a 4.1 movie would be rounded off to 4.5 stars, not to 4 stars, as you may expect.
- In the case of 8% of the ratings analyzed, the rounding up was done to the nearest whole star. For instance, a 4.5 rating would be rounded off to 5 stars.
- For one movie rating, the rounding off was completely bizarre: from a rating of 4 in the HTML of the page to a displayed rating of 5 stars.

Fandango officials replied that the biased rounding off was caused by a bug in their system rather than being intentional, and they promised to fix the bug as soon as possible

## Goal
We will analyze recent movie ratings to see if there has been a change in Fandango's movie ratings system after Hickey's analyis.

## Data
To accomplish our goal, we will use two data sets to compare from two different periods of time:

- Walt Hickey made the data he analyzed publicly available on [GitHub](https://github.com/fivethirtyeight/data/tree/master/fandango). We'll use the data he collected to analyze the characteristics of Fandango's rating system previous to his analysis.

- One of Dataquest's team members collected movie ratings data for movies released in 2016 and 2017. The data is publicly available on [GitHub](https://github.com/mircealex/Movie_ratings_2016_17) and we'll use it to analyze the rating system's characteristics after Hickey's analysis

## Understanding the Data

After importing and viewing the data, we isolate the columns that offer information about Fandango's ratings in separate variable.

```
fandango_before = early_ratings[['FILM', 'Fandango_Stars', 'Fandango_Ratingvalue', 'Fandango_votes', 'Fandango_Difference']]
```
```
fandango_after = late_ratings[['movie','year','fandango']]
```

When we review the README files for each dataset, we realize that the sampling process was different for each dataset, so the resulting samples are not likely to be representative of the population we were originally interested in. 

## New Goal
Because of the sampling differences, we need to change our goal. Let's determine whether there's any difference between Fandango's ratings for popular movies in 2015 and Fandango's ratings for popular movies in 2016. This can serve as a proxy for our original goal.

## Isolating the Samples We Need
We now want to compare
- All Fandango's ratings for popular movies released in 2015
- All Fandango's ratings for popular movies released in 2016. 

We will consider a movie popular if it has 30 or more fan ratings on Fandango. Let's check is all of the movies contain at least 30 fan ratings.

```
fandango_before[fandango_before['Fandango_votes'] < 30].count()
```

So the ```fandango_before``` dataset contains no movies that have fewer than 30 votes. The ```fandango_after``` dataset is a little more complicated because it does not include the number of fan ratings. To check this, we'll pick a random set of 10, then manually go check the number of ratings on Fandango.

```
fandango_after.sample(10,random_state=42)
```

All 10 of these movies have at least 30 votes on Fandango, so we can continue our analysis with decent confidence this sample contains what we want.

Both datasets contain movies that were released outside of 2015 or 2016. We want to focus our analysis on these years, so we need to select only the correct movies for both sets.

For the 2015 dataset, the year is included in the ```FILM``` column. We can use string methods to extract the year into a new column before isolating only 2015 movies.
```
fandango_before['Year'] = fandango_before['FILM'].str[-5:-1]
```
```
fandango_2015 = fandango_before[fandango_before['Year'] == '2015'].copy()
```

The 2016 dataset already identifies the year as a specific column, so we can easily extract the movies from 2016.
```
fandango_2016 = fandango_after[fandango_after['year'] == 2016].copy()
```

## Comparing Distribution Shapes

Now that we have isolated the movies we want to compare, let's begin the analysis with a high-level comparison of the distributions. For a first overview, we can create a Kernel Density Estimate plot for this comparison. 

![Fandango Ratings Distributions](/docs/assets/fandango_distributions.png)

Each distribution is left skewed. There are very few movies with ratings less than 3 stars. 2015 might be slightly more heavily balanced towards higher stars than 2016. There appears to be a little more of a tail with the 2016 data on the high side compared to 2015. 

## Comparing Relative Frequencies

We now want to analyze the data more granularly to investigate this potential difference between 2015 and 2016. Since the data sets contain different numbers of movies, let's create frequency tables for both years that show percentages. 

| 2015             |       |
|------------------|-------|
| Rating  | Percentage|
| 3.0              | 8.52  |
| 3.5              | 17.82 |
| 4.0              | 28.68 |
| 4.5              | 37.98 |
| 5.0              | 6.97  |

and

| 2016             |       |
|------------------|-------|
| Rating  | Percentage|
| 2.5              | 3.14  |
| 3.0              | 7.32  |
| 3.5              | 24.08 |
| 4.0              | 40.31 |
| 4.5              | 24.60 |
| 5.0              | 0.52  |

In 2016, fewer movies had high ratings of 4.5 or 5 stars. There are also more movies in 2016 that had the lowest rating of 2.5 stars. 

## Determining the Direction of the Change

We have confirmed that the two datasets have a difference in the distributions. However, the direction of the difference is not clear. Let's take a few summary statistics to get a better picture of the direction of the different. 


![Fandango Ratings Distributions](/docs/assets/fandango_summary_stats.png)

## Conclusions

There does appear to be a slight difference between Fandango ratings in 2015 and 2016. 2015 ratings were, on average, higher than 2016 ratings. We cannot state what caused the ratings to change, but the timing with Hikey's orginal article makes it possible that was the cause.

