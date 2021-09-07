# Optimizing-Pandas-Data-Types

## Description

### What
Project on how to optimize the default pandas data types that are infered when reading a csv- file, for example. The result is dataframe that takes up much less RAM without the loss of any information in the data.

### How
By making use of the pd.read_csv parameters, downcasting the numerical values, and representing columns with low cardinality in a different way (categories for strings and sparse arrays for numbers). A detailed explanation of how is listed in my personal learnings below.

### Why
Pandas provides data structures for in-memory analytics, which makes using pandas to analyze datasets that are larger than memory datasets somewhat tricky. Even datasets that are a sizable fraction of memory become unwieldy, as some pandas operations need to make intermediate copies. With small data (under 100 megabytes), performance is rarely a problem. When we move to larger data (100 megabytes to multiple gigabytes), performance issues can make run times much longer, and cause code to fail entirely due to insufficient memory.

While tools like Spark can handle large data sets (100 gigabytes to multiple terabytes), taking full advantage of their capabilities usually requires more expensive hardware. And unlike pandas, they lack rich feature sets for high quality data cleaning, exploration, and analysis. For medium-sized data, we’re better off trying to get more out of pandas, rather than switching to a different tool.

That's what this Notebook is all about: How to shrink your pandas dataframe so it fit's your RAM better - without losing any information + making faster operations possible.


## Skills
Technical skills honed in this project are:
- Batch-Import of csv-files
- Feature Engineering
- Pandas Data Type Optimization
- Undertsanding of Pandas & NumPy internals

## Requirements
Installed modules:

| Library            | Version |
| :---               | --- |
| sys (Python)       | 3.9.1 | 
| pandas             | 1.2.5 | 
| numpy              | 1.21.0 | 
| matplotlib         | 3.4.1 |


## Personal Learnings:
This time the personal learnings can be summarized in a checklist for future projects:
1. **Use the pd.read_csv() parameters**:
  - **usecols**: read only the specified columns
  - **nrows**: read only the rows needed
  - **dtype**: if you already know the optimal data type for the columns, specify it with a dict
  - **chunksize**: load the data as an iterator and preprocess/aggregate it in a loop, concatenating the results again

2. **Downcast numeric values**: <br/>
When pandas reads a csv it looks at the first few rows for each column and then guesses the data type (int, float, string, etc.). Since pandas doesn't know if the last value in this column exceeds any size (big numbers), it automatically upcasts to the biggest format (int64, float64, etc.) which is often not needed. You can use the **pd.to_numeric(downcast=str)** function to take care of this.

3. **Use the categorical data type for low cardinality strings**: <br/>
Often Strings are represented as objects in a pandas DataFrame, which itself can be a memory safer when transformed to strings. The problem with strings though is that they occupy a lot of space and are often frequently repeated in a column. This means the column has low cardinality and a more efficient data storage option are categories. Much like OrdinalEncoder in Scikit-learn, pandas safes a mapping for all the different strings internally and the rows get the corresponding number (internally). That way strings won't be repeated and memory is safed --> **df.astype('category')**

4. **Use sparse arrays for low cardinality numbers**: <br/>
Use a sparse array if you have sparse data. Sparse data is data that primarily has the same value. 10 million float64 values uses 80MB of memory. If most of those values are the same, you can save a whole lot of memory converting to sparse. Making all but 10 of those values the same and converting to sparse drops the memory to .000248MB! 👍 Convert to sparse with pd.arrays.SparseArray().

## Outlook
XXXXXXXXXX


## References
I found the best information on this topic here:
- [Dataquest Blog](https://www.dataquest.io/blog/pandas-big-data/)
- [Pandas Documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/scale.html#)

The Dataset is from Ken Huang: 
- [Kaggle Profile](https://www.kaggle.com/kenhuang41/nba-basic-game-data-by-player)


## 📫 Let's connect and learn from each other:

[<img src="https://github.com/kevin-goetz/kevin-goetz/blob/main/LinkedIn Logo.png" height="40em" align="center" alt="Connect with Me on LinkedIn" title="Connect with Me on LinkedIn"/>](https://linkedin.com/in/kgötz) [<img src="https://github.com/kevin-goetz/kevin-goetz/blob/main/Codewars Logo.svg" height="40em" align="center" alt="Connect with Me on Codewars" title="Connect with Me on Codewars"/>](https://www.codewars.com/users/kevin-goetz)

