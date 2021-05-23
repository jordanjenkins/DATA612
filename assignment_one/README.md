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
