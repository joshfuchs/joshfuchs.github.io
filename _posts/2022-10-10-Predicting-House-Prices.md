# Predicting House Sale Prices

See the full Jupyter Notebook on [GitHub](https://github.com/joshfuchs/DataScience_projects/blob/master/Predicting_House_Sale_Prices.ipynb). 


## Data
We'll work with the Ames Housing data set, from 2006 to 2010. The origianal paper with the data can be [found here](https://www.tandfonline.com/doi/abs/10.1080/10691898.2011.11889627) and the [data dictionary is here](https://s3.amazonaws.com/dq-content/307/data_description.txt).

## Goal
We will build a simple machine learning pipeline to clean the data, select features, engineer features, and make predictions. We'll write three different functions that will serve these purposes.

## Transform Features
Our ```transform_features``` function will remove features that are missing more than 5% of their values. We then fill in the numerical columns with the mode of that column. It will create two columns that are more descriptive and drop columns that are not useful or that leak information about the target. 


```
def transform_features(df):
    '''
    This functions first removes features we don't want to use 
    in the model, because of missing values or data leakage. 
    
    It then transforms the features into the proper format, 
    such as numerical to categorial, scaling numerical, and 
    filling in missing values. 
    
    Finally, it creates new features by combining other features
    '''
    
    # remove any column with more than 5% missing values
    
    df_null_counts = df.isnull().sum()
    
    df_length = df.shape[0]
    
    # Filter Series to columns containing >5% missing values
    drop_missing_cols = df_null_counts[(df_null_counts > df_length/20)].sort_values()

    # Drop those columns from the data frame. 
    df_fewer_missing = df.drop(drop_missing_cols.index, axis=1)

    # for numerical columsn missing fewer than 5% of values
    # fill in with the mode
    
    numerical_cols = df_fewer_missing.select_dtypes(include=['int','float']).columns
    #print(numerical_cols)
    for col in numerical_cols:
        if df_null_counts[col] < (0.05 * df_length):
            df_fewer_missing[col].fillna(df_fewer_missing[col].mode(),inplace=True) 
    
    # create two new columns that are more descriptive
    years_sold = df_fewer_missing['Yr Sold'] - df_fewer_missing['Year Built']
    years_since_remod = df_fewer_missing['Yr Sold'] - df_fewer_missing['Year Remod/Add']
    df_fewer_missing['Years Before Sale'] = years_sold
    df_fewer_missing['Years Since Remod'] = years_since_remod
    
    # Drop rows with negative values for both of these new features
    df_fewer_missing = df_fewer_missing.drop([1702, 2180, 2181], axis=0)

    # drop columns that aren't useful 
    # and those that leak info about the sale
    df_fewer_missing = df.drop(["PID", "Order", "Mo Sold", 
                                "Sale Condition", "Sale Type", "Year Built", "Year Remod/Add",
                               'Mas Vnr Area','BsmtFin SF 1','Total Bsmt SF','Garage Yr Blt'], axis=1)
    
    return df_fewer_missing

```

## Select Features
Next, our ```select_features``` function selects which features we want to include in our model. For the numerical columns, we set a correlation threshold and only keep features that meet that threshold. Similarly for nominal features, we set a uniqueness threshold. Finally, we drop the text columns.

```
def select_features(df, coeff_threshold=0.4, uniq_threshold=10):
    
    # drop the following columns if they are in the df
    drop_columns = ['Garage Cars', 'TotRms AbvGrd', 'Garage Area']
    df_columns = df.columns.tolist()
    for col in drop_columns:
        if col in df_columns:
            df.drop(col,axis=1,inplace=True)

    # select only the numerical dtypes
    # if the correlation coefficient is less than the threshhold
    # drop those columns
    numerical_df = df.select_dtypes(include=['int', 'float'])
    abs_corr_coeffs = numerical_df.corr()['SalePrice'].abs().sort_values()
    df = df.drop(abs_corr_coeffs[abs_corr_coeffs < coeff_threshold].index, axis=1)
    
    
    # deal with the nominal features
    # only keep those with more than the threshhold
    nominal_features = ["PID", "MS SubClass", "MS Zoning", "Street", "Alley", "Land Contour", "Lot Config", "Neighborhood", 
                    "Condition 1", "Condition 2", "Bldg Type", "House Style", "Roof Style", "Roof Matl", "Exterior 1st", 
                    "Exterior 2nd", "Mas Vnr Type", "Foundation", "Heating", "Central Air", "Garage Type", 
                    "Misc Feature", "Sale Type", "Sale Condition"]
    
    transform_cat_cols = []
    for col in nominal_features:
        if col in df.columns:
            transform_cat_cols.append(col)

    uniqueness_counts = df[transform_cat_cols].apply(lambda col: len(col.value_counts())).sort_values()
    drop_nonuniq_cols = uniqueness_counts[uniqueness_counts > 10].index
    df = df.drop(drop_nonuniq_cols, axis=1)
    
    text_cols = df.select_dtypes(include=['object'])
    for col in text_cols:
        df[col] = df[col].astype('category')
    df = pd.concat([df, pd.get_dummies(df.select_dtypes(include=['category']))], axis=1).drop(text_cols,axis=1)
    
    return df
```


## Train and Test
Our final function, ```train_and_test```, fits a linear regression model to the dataset. It accepts an option *k-value* that determines the level of cross-validation. k=0 is holdout validation. Higher values of k set the number of folds. It returns the root mean square error as the metric to evaluate the model. 

```
def train_and_test(df, k=0):
    '''
    df = pandas dataframe
    k = integer for cross validation
    '''
    
    if k == 0:
        # holdout validation
        # split into train and test dfs
        train = df[:1460]
        test = df[1460:]
    
        # train linear regression model using all numerical columns 
        # except SalePrice
    
        numeric_train = train.select_dtypes(include=['float','int'])
    
        features = numeric_train.columns.drop("SalePrice")


        lr = LinearRegression()
        print(features)

        lr.fit(train[features],train['SalePrice'])
    
        predictions = lr.predict(test[features])
    
        mse = mean_squared_error(test['SalePrice'],predictions)
        rmse = np.sqrt(mse)
    
        return rmse
    
    if k > 0:
        # cross validation 
        lr = LinearRegression()
        kf = KFold(n_splits = k, shuffle=True,random_state=1)

        numeric_train = df.select_dtypes(include=['float','int'])
        features = numeric_train.columns.drop("SalePrice")

        
        mses = cross_val_score(lr,
                              df[features],
                              df['SalePrice'],
                              scoring = 'neg_mean_squared_error',
                              cv=kf)
        
        rmse = np.sqrt(np.absolute(np.mean(mses)))
        return(rmse)
```


