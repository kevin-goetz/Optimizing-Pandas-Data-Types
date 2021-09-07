# Optimizing-Pandas-Data-Types
Project on how to optimize the default pandas data types for speed and space by feature engineering and downcasting.

## Personal Learnings:
This time the personal learnings can be summarized in a checklist for future projects:
1. Only read in the data you need. **pd.read_csv()** has a lot of useful **parameters** to narrow down the information:
  - **usecols**: read only the specified columns
  - **nrows**: read only the rows needed
  - **dtype**: if you already know the optimal data type for the columns, specify it with a dict
  - **chunksize**: load the data as an iterator and preprocess/aggregate it in a loop, concatenating the results again

2. **Downcast numeric values**. When pandas reads a csv it looks at the first few rows for each column and then guesses the data type (int, float, string, etc.). Since pandas doesn't know if the last value in this column exceeds any size (big numbers), it automatically upcasts to the biggest format (int64, float64, etc.) which is often not needed. You can use the **pd.to_numeric(downcast=str)** function to take care of this.

3. **Downcast String values**. Often Strings are represented as objects in a pandas DataFrame, which itself can be a memory safer when transformed to strings. The problem with strings though is that they occupy a lot of space and are often frequently repeated in a column. This means the column has low cardinality and a more efficient data storage option are categories. Much like OrdinalEncoder in Scikit-learn, pandas safes a mapping for all the different strings internally and the rows get the corresponding number (internally). That way strings won't be repeated and memory is safed --> **df.astype('category')**

4. Use a sparse array if you have sparse data. Sparse data is data that primarily has the same value. 10 million float64 values uses 80MB of memory. If most of those values are the same, you can save a whole lot of memory converting to sparse. Making all but 10 of those values the same and converting to sparse drops the memory to .000248MB! 👍 Convert to sparse with pd.arrays.SparseArray().

## References
I found the best information on this topic here:
- https://www.dataquest.io/blog/pandas-big-data/
- https://pandas.pydata.org/pandas-docs/stable/user_guide/scale.html#

The Dataset is from Ken Huang, posted on Kaggle: 
- https://www.kaggle.com/kenhuang41/nba-basic-game-data-by-player 
