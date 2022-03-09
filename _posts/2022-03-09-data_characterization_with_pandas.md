# Data Characterization with Pandas

When exploring a new dataset with Pandas, I always run the same first six lines of code to get a basic understanding of what is in the dataset and how it is structured. I have learned that these commands most quickly get me to a place where I can understand the dataset to get started on analysis. Let's go ahead and load the dataset (this one doesn't count!):

```
df = pd.read_csv('file.csv')
```
Now, we have read in the file and loaded it as df.


```
df.shape
```
The first and most basic command. I want to know the how many rows and columns (in that order in Python) so I have an idea of what I am working with.

```
df.head()
```
The head() command displays the first 5 rows of each column, though you might not see every column depending on how many your dataset includes. This gives me an initial look into the column names and data types.

```
df.dtypes
```
The utility of this immediately follows the previous command. Here I know I see every column name as well at the data type for each. Now is when I start to get a sense of the amount of categorial and numerical data I have. 

```
df.describe()
```
describe() gives a top level statistical summary of the numerical columns. 


```
cols_with_missing = [col for col in df.columns
                    if df[col].isnull().any()]

print('Columns with missing values: ', cols_with_missing)

for x in cols_with_missing:
    print(x, ':', df[x].isna().sum())
```
Now that I have a general sense of what my data includes, I want to know how much data is missing for each column. This command first tells me the name of each column with missing data, then tells me how much data is missing for each column. This is going to be key for whatever data exploration and analysis I have. 

```
sns.relplot(x = 'Variable1', y = 'Variable2', hue = 'Variable3', data = df)
```
Finally, I love using Seaborn to visualize the dataset. I like sns.relplot because it quickly gives me a 3 dimensional view of the data. I pick different variables to start looking for and clustering or correlations that might be useful in my analysis. 