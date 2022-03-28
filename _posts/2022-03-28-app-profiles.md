# Profitable App Profiles

This project analyzes app data (for both Android and iOS mobile apps) to identify which apps are most popular and why. This will help us understand what types of apps are likely to attract more users. We'll be using basic functions, lists, and dictionaries in Python to accomplish this task.

This post summarizes the project and conclusions. You can find my full [Jupyter notebook here](https://github.com/joshfuchs/DataScience_projects/blob/master/App%20Profiles%20Project.ipynb).

## Import and Explore Data

Data can be downloaded directly from:

[Android apps](https://dq-content.s3.amazonaws.com/350/googleplaystore.csv)

[iOS apps](https://dq-content.s3.amazonaws.com/350/AppleStore.csv)


After importing the data, we'll define a function we can use to easily view the data

```
def explore_data(dataset, start, end, rows_and_columns=False):
    dataset_slice = dataset[start:end]    
    for row in dataset_slice:
        print(row)
        print('\n') # adds a new (empty) line after each row

    if rows_and_columns:
        print('Number of rows:', len(dataset))
        print('Number of columns:', len(dataset[0]))
```

We use `explore_data` to quickly look at both the Android and iOS datasets and get a sense of what is included. Next, we are ready to clean the data to prepare it for analysis.

## Clean Data
Android data has an bad entry on row 10472 according to [this Kaggle discussion](https://www.kaggle.com/lava18/google-play-store-apps/discussion/66015). We delete that row using the `del` command. 

Next, we should count the number of duplicate rows in the Android dataset.

```
unique_app_names = []
duplicate_app_names = []

for x in android:
    app_name = x[0]
    if app_name in unique_app_names:
        duplicate_app_names.append(app_name)
    else:
        unique_app_names.append(app_name)
        
print('Number of unique apps: ', len(unique_app_names))
print('\n')
print('Number of duplicate apps: ', len(duplicate_app_names))
```

We see that there are 9659 unique apps and 1181 duplicate apps in this dataset. We run the same lines of code for the iOS dataset and find that there are no duplicates.

We don't want to remove the duplicate entries randomly. Instead, let's keep the entry with the largest number of reviews. This will give us the most up-to-date and insightful information.

To remove the duplicates, we will create a dictionary with only the highest number of reviews saved. The dictionary key will be the app name and the value will be the highest number of reviews. In the next step, we will use this to identify with row of android data corresponds to this.

```
reviews_max = {}
for app in android:
    name = app[0]
    n_reviews = float(app[3])
    if name in reviews_max and reviews_max[name] < n_reviews:
        reviews_max[name] = n_reviews
    elif name not in reviews_max:
        reviews_max[name] = n_reviews
```

Now, we will use the `reviews_max` dictionary to check if each app matches the number of max reviews. If it does, we will save that row. We'll confirm each app is not already in `android_clean` before adding it to account for the edge case where the total number of reviews matches on multiple entries.

```
android_clean = [] # new, clean dataset
already_added = [] # only store app names

for app in android:
    name = app[0]
    n_reviews = float(app[3])
    if n_reviews == reviews_max[name] and name not in already_added:
        android_clean.append(app)
        already_added.append(name)
```

## Remove Non-English Apps
We are only interested in English language apps for this exercise, so we will remove, using `ord()`, any non-English apps.

First, let's write a function that takes a string and returns `False` if there is a character in the string that does not belong to the set of common English characters. We'll set the limit at 3 non-english characters to account for emoji, etc.

```
def check_characters(string):
    non_english = 0
    for x in string:
        if ord(x) > 127:
            non_english += 1
    if non_english > 3:
        return False
    else:
        return True
```

## Isolating Free Apps

Now, we want to isolate only the free apps. Our business model for this project is in-app ads. 

```
ios_free = []
android_free = []

for row in ios_english:
    price = row[4]
    if price == '0.0':
        ios_free.append(row)
        
        
for row in android_english:
    price = row[7]
    if price == '0':
        android_free.append(row)
```


Now, the data are clean and ready for analysis

## Most Common Apps By Genre
Our goal is to identify apps that are successful (a.k.a. attract more users) on both Android and iOS operating systems. The validation strategy for an app has three steps:

1. Build a minimal Android version of the app, and add it to Google Play.
2. If the app has a good response from users, we develop it further.
3. If the app is profitable after six months, we build an iOS version of the app and add it to the App Store.

We now want to determine the most common genres for each market. We will build a frequency table that shows up the percentage of apps by genre

```
def freq_table(dataset, index):
    new_dict = {}
    total = 0
    
    for row in dataset:
        total += 1
        value = row[index]
        if value in new_dict:
            new_dict[value] += 1
        else:
            new_dict[value] = 1
            
    dict_percentages = {}
    for key in new_dict:
        percentage = (new_dict[key] / total) * 100
        dict_percentages[key] = percentage
    return dict_percentages
```

We generate this table on both sets of data

#### Notes and comments on iOS apps

The most common genre is Games at 58% followed by Entertainment at 7.8%. 

Games and entertainments apps are the most popular general types of apps. There is a spattering of productivity apps (Education, Utilities, Health & Fitness), but these form a significant minority of the general app types.     

#### Notes and comments on Android apps

There is not as much of a clear preference amongst Android app users. 

It is not immediately clear the exact difference between the `Category` and `Genres` columns, except that `Category` is a coarser grouping. 

In both categories, there is a mix of entertainment and productivity apps. 

## Apps with Most Users
Now, let's determine the kind of apps with the most users. For the Android dataset, we can use the `Installs` columns. For iOS, we will use the `rating_count_tot` column. We will write a nested loop that calculates the average number of users by genre for each dataset. 

```
ios_genres = freq_table(ios_free, 11)

for genre in ios_genres:
    total = 0
    len_genre = 0
    for row in ios_free:
        genre_app = row[11]
        if genre_app == genre:
            total += float(row[5])
            len_genre += 1
    avg_num_ratings = total / len_genre
    print(genre, avg_num_ratings)
```

#### iOS conclusions
Navigation apps have the most average reviews. Social Networking, Reference, and Music are the next highest. These might be driven by a few particularly popular apps in each case, but we do get a variety here.



#### Android conclusions
Next, we can look at the average number of installs for the Google Play store. This dataset only gives a minimum number of installs based on ranges such as '100,000+', so we will use this base number to get an estimate.

Maps and Navigation, News and Magazines, Productivity are the categories of apps that contain the highest approximate average installs. 



