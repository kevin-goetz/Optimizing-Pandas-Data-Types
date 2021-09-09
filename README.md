# Optimizing-Pandas-Data-Types

## Description

### What
Project on how to optimize the default pandas data types that are inferred when reading a csv-file. The result is a dataframe that takes up much less RAM without the loss of any information in the data.

### How
By making use of the pd.read_csv parameters, downcasting the numerical values, and representing columns with low cardinality in a different way (categories for strings and sparse arrays for numbers). A detailed explanation of how is listed in my personal learnings below.

### Why
Pandas provides data structures for in-memory analytics, which makes using pandas to analyze datasets that are larger than memory somewhat tricky. Even datasets that are a sizable fraction of memory become unwieldy, as some pandas operations need to make intermediate copies. With small data (under 100 megabytes), performance is rarely a problem. When we move to larger data (100 megabytes to multiple gigabytes), performance issues can make run times much longer, and cause code to fail entirely due to insufficient memory.

That's what this Notebook is all about: How to shrink your pandas dataframe so it fit's your RAM better - without losing any information.

## Impact
I tested the function below with the 17 Seaborn test datasets. The code for the test is in the folder ["Seaborn Test Datasets"](https://github.com/kevin-goetz/Optimizing-Pandas-Data-Types/tree/main/Seaborn%20Test%20Datasets):

[<img src="https://github.com/kevin-goetz/Optimizing-Pandas-Data-Types/blob/main/Seaborn%20Test%20Datasets/Seaborn%20Test%20Data%20Sets%20Downsized%20Ranking.PNG" height="400em" align="center"/>

Another great plus of this project is when you safe your downsized DataFrame to **df.to_feather(path)**. All the data types are safed so you only have to do it once + it costs way less disc space + the read/write speed is much faster. 

## Copy & Try Yourself!

Just copy the function definition and use it with one of your bigger dataframes: df_small = downcast(df_big). That easy.

```python

def downcast(df: pd.DataFrame) -> pd.DataFrame:
    
    ''' Compression of the common dtypes "float64", "int64", "object" or "string" '''

    # memory before downcasting
    mem_before = df.memory_usage(deep=True).sum()
    mem_before_mb = round(mem_before / (1024**2), 2)

    # convert the dataframe columns to appropriate dtypes (e.g. object to string, or 1.0 float to 1 integer, etc.)
    df = df.convert_dtypes()

    # string categorization (only the ones with low cardinality)
    for column in df.select_dtypes(['string', 'object']):
        if (len(df[column].unique()) / len(df[column])) < 0.5:
            df[column] = df[column].astype('category')

    # float64 downcasting
    for column in df.select_dtypes(['float']):
        df[column] = pd.to_numeric(df[column], downcast='float')

    # int64 downcasting (depending if negative values are apparent (='signed') or only >=0 (='unsigned'))
    for column in df.select_dtypes(['integer']):
        if df[column].min() >= 0:
            df[column] = pd.to_numeric(df[column], downcast='unsigned')
        else:
            df[column] = pd.to_numeric(df[column], downcast='signed')

    # memory after downcasting & compression
    mem_after = df.memory_usage(deep=True).sum()
    mem_after_mb = round(mem_after / (1024**2), 2)
    compression = round(((mem_before - mem_after) / mem_before) * 100)

    # downcasting summary
    print(f'DataFrame compressed by {compression}% from {mem_before_mb} MB down to {mem_after_mb} MB.')

    return df
    
```

## Skills
Technical skills honed in this project are:
- Batch-Wrangling of csv-files
- Feature Engineering
- Pandas Data Type Optimization
- Understanding of Pandas & NumPy internals
- Using different flat file types with faster I/O

## Requirements
Installed modules:

| Library            | Version |
| :---               | --- |
| sys (Python)       | 3.9.7 | 
| pandas             | 1.3.2 |
| seaborn            | 0.11.1 |

## Personal Learnings:
This time the personal learnings can be summarized in a checklist for future projects:
1. **Use the pd.read_csv() parameters**:
  - **usecols**: read only the specified columns
  - **nrows**: read only the rows needed
  - **dtype**: if you already know the optimal data type for the columns, specify it with a dict
  - **chunksize**: load the data as an iterator and preprocess/aggregate it in a loop, concatenating the results again

2. **Choose correct data types**: <br/>
**df.convert_dtypes()** is a really powerful method that converts the df into a df with correct data types. For example: There is a column that has a float64 data type, but it only holds numbers with .0, so it's basically all integers. This method corrects the column automatically to int64. Or Object data types that are actually strings. All this safes memory and it also brings another advantage: the new data types are pandas ExtensionDtype. This data type allows integer columns to have NA-values and not convert to float, like it used to be. With the [Pandas Version Update 1.3.0](https://pandas.pydata.org/docs/whatsnew/v1.3.0.html) from July 2021 those data types can be downcasted.

3. **Downcast numeric values**: <br/>
When pandas reads a csv it looks at the first few rows for each column and then guesses the data type (int, float, string, etc.). Since pandas doesn't know if the last value in this column exceeds any size (big numbers), it automatically upcasts to the biggest format (int64, float64, etc.) which is often not needed. You can use the **pd.to_numeric(downcast=str)** function to take care of this.

4. **Use the categorical data type for low cardinality strings**: <br/>
Often Strings are represented as objects in a pandas DataFrame, which itself can be a memory safer when transformed to strings. The problem with strings though is that they occupy a lot of space and are often frequently repeated in a column. This means the column has low cardinality and a more efficient data storage option are categories. Much like OrdinalEncoder in Scikit-learn, pandas safes a mapping for all the different strings internally and the rows get the corresponding number (internally). That way strings won't be repeated and memory is safed --> **df.astype('category')**

5. **Use sparse arrays for low cardinality numbers**: <br/>
Sparse data is data which contains mostly NaN / missing value, though any value can be chosen, including 0. On the contrary, A column in which the majority of elements are non zero is called dense. Convert to sparse with **df[column] = df[column].astype(pd.SparseDtype(df['FG'].dtype, pd.NA))** or directly use the parameter of pandas dummy function: **pd.get_dummies(data, sparse=True)**. Though this technique was not needed in this project, it could be an advantage in future projects with sparse data.

6. **Using optimized I/O file formats**: <br/>
A csv-file doesn't save any data types so it would be a pitty to loose all the work of optimizing a dataframe when saved to csv. There are more suitable file formats with faster I/O as well, like: pickle, hdf5, feather, parquet, etc. Just use pandas **df.to_feather(filepath, compression='lz4')**.

## Outlook
There is still one big disadvantage: The function is only applicable when the DataFrame is already loaded into RAM. So what if we need to optimize the data types before loading it into RAM because it doesn't fit in there yet? That's where chunking and saving the minimum and maximum of the column into a DataFrame comes in handy. One could then downcast this intermediate DataFrame and safe the optimized Data Types as a dictionary for the read_csv parameter "dtype: dict". Another opportunity could be a out-of-core library like [Vaex](https://vaex.io/docs/index.html) that makes use of memory-mapping and doesn't load all the data into RAM.

Also sparse arrays weren't used in this project and could be an advantage in certain datasets with sparse data, e.g. tables for recommendation engines that have a lot of 1/0 data. This feature would exceed an acceptable function length and could be added in a future project for a custom module.


## References
**Downcasting and Categoricals**:
- [Dataquest Blog](https://www.dataquest.io/blog/pandas-big-data/)
- [Pandas Documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/scale.html#)

**Sparse Data**:
- [TDS Blog](https://towardsdatascience.com/working-with-sparse-data-sets-in-pandas-and-sklearn-d26c1cfbe067)

**Optimal I/O Files**:
- [TDS Blog](https://towardsdatascience.com/the-best-format-to-save-pandas-data-414dca023e0d)

The **Dataset** is from Ken Huang: 
- [Kaggle Profile](https://www.kaggle.com/kenhuang41/nba-basic-game-data-by-player)


## 📫 Let's connect and learn from each other:

[<img src="https://github.com/kevin-goetz/kevin-goetz/blob/main/LinkedIn Logo.png" height="40em" align="center" alt="Connect with Me on LinkedIn" title="Connect with Me on LinkedIn"/>](https://linkedin.com/in/kgötz) [<img src="https://github.com/kevin-goetz/kevin-goetz/blob/main/Codewars Logo.svg" height="40em" align="center" alt="Connect with Me on Codewars" title="Connect with Me on Codewars"/>](https://www.codewars.com/users/kevin-goetz)

