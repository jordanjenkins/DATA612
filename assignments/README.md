## Assignment One
Load and explore the stock dataset provided at: https://raw.githubusercontent.com/frankData612/data_612/master/stock_data/stocks_yahoo.csv.

Dataset is provided by the author of the "Easy And Fun With BeautifulSoup". Data are extracted from the Yahoo Finance website and include 90 days of data from November 21, 2019 until April 28, 2020. Data are row, current and good for learning purposes.

### Load Data
#### Required Packages
`import pandas as pd`
#### Data loading
The data was imported using the pandas function `read_csv(filepath, low_memory = 'False')`. The low_memory parameter was added to supress a warning of mixed dtypes in in csv.

### Initial Exploratory Analysis
Several steps were taken to gain an understanding of the stocks dataset.
* `pd.head()` and `pd.tail()` were used to look at the beginning and end of the dataset and provided an opportunity to
 see what data was contained within the csv.
 
 * The attribute of the dataframe `pd.column` printed the columns names of the dataset.
 
 * It is important to know the data types of the dataset as a whole and the Series within. 
  `type(dataframe)` verified that the data was stored in a Pandas DataFrame. `pd.info()` 
  printed the type of each column and well as the non-null counts. Several columns have 
  null cells which will need to be evaluated and addressed prior to any analysis.
  
 ### Summarize a few columns
 I was interested in grouping the data by characteristics of the date (e.g., by week). 
 In order to support this, I converted the date column from type date to datetime using
 `pd.to_date_time`.  
 
 I subset out the columns I was interested in using:  
 `df_sub = df[["date", "company_name", "average volume", "1Y target est"]]`
 
 The final step was to group the average volume and 1Y estimate by company and week, giving a weekly average of these values for each company.
 
 `df_sub.groupby(['company_name', df_sub['date'].dt.isocalendar().week]).mean()`
 
 `groupby()` groups a DataFrame by the `'company_name'` column and by the ISO week.
 `df_sub['date'].dt.isocalendar().week`: `dt` is the accessor object for datetime properties of a series. 
 `isocalendar()` is a pandas function that returns a tuple containing the year, week number, and weekday of a date. I was 
 interested in the week which was accessed by `.week`. The mean of these groups was computed using `mean()`.
 
 ## Assignment Two
 I continued with the same data set as assignment one. The primary objective of this assigment was to create a date difference in months from the most recent date in the dataset.
 
 ### Steps Taken
 * I had already converted the date field to datetime, otherwise this would have been the first step.
 * I found the max date using `max_date = df.date.max()`
 * Create a new column `'days_from_max_date'` using: `max_date - df.date`
   * Each record was populated with a positive number of days from the max date.
 * Convert days to months
   * A new column `months from max_date` defined as `'days_from_max_date' - np.timedelta(1,'M')`
   * `np.timedelta()` takes a number representing the number of units (here 1) and a date/time unit (here 'M' for month), and returns a datetime object.
 * Finally, `df.to_csv()` was used to save the reulting modified dataframe to a text file.
 
 ### Assignment Three
 #### Additional Feature Engineering
Some columns are not useful for analysis as they currently stand. I converted the `52 week range` into two columns representing the min and max 52 week value, the daily price range `price range` into daily min and max values, and the `price_change` into a percent change and absolute change value.

The general method I used was `df.column.str.split(delimiter, n = 1 ,expand = True)`.
##### Example Split and Create New Columns
```
new = df['52 week range'].str.split(' - ', n=1, expand=True)
df['min_52'] = new[0]
df['max_52'] = new[1]
df.drop(columns='52 week range', inplace=True)  
```
The price change needed additional processing and was done as follows:
```
new = df['price_change'].str.split(' ', n=1, expand=True)
df['price_chng'] = new[0]
df['price_chng_pct'] = new[1].str[1:-2]
df.drop(columns='price_change', inplace=True)
```
`new[1].str[1:2]` converted a value such as `(+13.45%)` to `+13.45`.

Next I converted the most of the Series with type object to type numeric using:
```
df[cols_to_numeric] = df[cols_to_numeric].apply(pd.to_numeric, errors='coerce', axis=1)
```

Where `cols_to_numeric` was a list of colums that I wanted to change to numeric. This line applies `pd.to_numeric` to all of the columns listed, coercing any non-numbers to Nan.

#### Subset number of companies
I subsetted the data to include only four companies, GOOG, MSFT, AAPL, and AMZN, these are large tech companies and I waould like to compare them.

#### Plotting
I looked at the distribution of trade volume two ways with a histogram and box plots. I also looked at the distribution of two variables `volume` and `price_chng_pct` using a hex desity plot with histograms. Finally, I used a scatter plot to compare `price_chng_pct` as a function of `volume` for each of the four companies.

### Assignment Four
I created a csv with current data using the code at https://github.com/tonysla/Easy-And-Fun-With-BeautifulSoup which is what was used to generate the original stock data set.

Assignment four occurs at the beginning of the notebook. I loaded the new data using `read_csv()`, modified a couple column names so they match the names from the existing data, and appended the dataframe to the existing data using `df.append(df_new, ignore_index=True)`.

#### Addressing missingness
`df.isnull().sum(axis = 0)` provided me with a snapshot of missing data by column. Most of the data in `earnings date` was missing, I opted to drop the column. Altenratively, I could have transformed this into a boolean `is_earnings_date`.

Since a simple fillna would not work because there are inherent groupings to the data. I created a helper function that gorups the data (here by `company name`), sets the index by date and tries three different fill methods.

After applying this function to the dataset, there were still a decent number of records with missing data. These were only for three stocks and all of the data for each record was missing. I dropped these records from the dataset. After this processing, I had a dataset with no missingness.

## Assignment Five

### Change Column from a Non-Categorical Type to a Categorical Type
It may be useful to look at volume as it relates to the day's average volume. If the volume is much higher than average, there could be a lot of volatility in the price. I will split the column into the following:
* Higher than Average (greater than the 75th percentile for the individual stock)
* Average (between the 25th and 75th percentile)
* Below Average (below the 25th percentile)

I created a new column `vol_cat` based on `volume` that gave a quartile rank of the volume. I intended to use `pandas.qcut` to do this, but I had issues with duplicate edge values for a few companies. If I would have considered volume market-wide I would have likely been able to use `qcut` without issue. To work around this, I utilized a custom function that defines percentile edges and applies a rank to the values in a series.

```
def pct_rank_qcut(series, n):
    edges = pd.Series([float(i) / n for i in range(n + 1)])
    f = lambda x: (edges >= x).argmax()
    return series.rank(pct=1).apply(f)
```

The actual ranking was done with the following:
```
# Group the data by company name and apply the custom qcut function
df['vol_cat'] = df.groupby('company_name')['volume'].transform(pct_rank_qcut,4)
```

I then combined the two middle quartiles and defined `vol_cat` as type categorical.
```
df['vol_cat'] = np.where(
     df['vol_cat'].between(0, 1), 
    'Below_Avg', 
     np.where(
        df['vol_cat'].between(1, 3), 'Avg', 'Above_Avg'
     )
)
```
### Column to String
I created a column of weekday from the date and changed the type to string. `df['day_of_week'] = df.date.dt.dayofweek.astype(str)`

## Assignment Six
### Regex Exercise
The goal was to convert `market cap` from a string to a float in the same unit. `market cap` looks like "26B" where there is a number ending with a letter in [B, M, T].

#### Regex Pattern Number
The pattern I used to extract the number is as follows: `r'(\d+\.?\d*)'`.
* \d+ digits before optional decimal
* .? optional decimal(optional due to the ? quantifier)
* \d* optional digits after decimal

#### Regex Pattern Number
The pattern used to extract the unit is as follows: `r'([a-zA-Z]+)'`. It just returns an letter matched.

#### Create New Column
I used to following function to put each market cap value into the same unit:
```
def unit_conv(row):
    val = row['mk_val']
    unit = row['mk_multiplier']
    
    if unit == 'T':
        return val * 1000 # if unit is Trillion, multiply by 1000 to get billions
    elif unit == 'M':
        return val / 1000 # if unit is Million, divide by 1000 to get billions
    else:
        return val
```

This value was placed in `mk_cap_cleaned_billions` using `df['mk_cap_cleaned_billions'] = df.apply(unit_conv, axis=1)`
