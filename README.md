# NumPy and Pandas code snippets

* [Iteration and access]()
* [Generation of time ranges]()
* [Grouping and aggregation over time]()
* [Modifying values]()
* [Appending rows]()
* [Appending columns]()
* [Finding values]()
* [Python language]()
* [Python environment, packaging, deployment]()

## Iteration and access

### Iteration

Iterating DataFrame:

* `for ir in t.itertuples()` the fastest approach (100 times faster than iterrows). `ir` is a tuple (of type `pandas.core.frame.PandasV`. `ir[0]` ór `ir.Index` is index of the row, `ir[1]` or `ir.my_column` first column etc.
* `for index, row in df.iterrows()` slow because it converts each row into a series
* `for date, row in df.T.iteritems()` transpose and iterate. great solution to iterate over a specific column
* `for index in df.index`
* `df.values`, `df.column_name.values`, `df.index.values` access underlying numpy array
* `apply`

Iterating Series:

* `for date, row in ser.iteritems()`

Iterating dictionary sorted:

* `for key, value in sorted(my_dict.iteritems())` in Python 2
* `for key, value in sorted(my_dict.items())` in Python 3 (and 2)
* `sorted(foo.keys())`

Links:

* https://stackoverflow.com/questions/7837722/what-is-the-most-efficient-way-to-loop-through-dataframes-with-pandas

### Selection

Three main components of any DataFrame:

* `index = df.index` list of row labels of type `pandas.core.indexes.base.Index`
* `columns = df.columns` list of column labels of type `pandas.core.indexes.base.Index`
* `values = df.values` array of cells of type `numpy.ndarray`

There are three main indexers:

* `df[ ]` select column(s)
  * `df['food']` select single column (returns Series)
  * `df[['color', 'food', 'score']]` select multiple columns (returns DataFrame even if only one column in the list)

* `df.loc[ ]` uses *labels* for selection of rows and/or columns
  * `df.loc['Niko']` select single row (Series of field valus) using its label (index value)
  * `df.loc[['Niko', 'Penelope']]` select multiple rows
  * `df.loc['Niko':'Dean']` select a slice of rows. Importantly, the last label is included
  * `df.loc[:'Aaron']` slice from the beginning
  * `df.loc['Dean':]` slicde to the end
  * `df.loc['Niko':'Christina':2]` slice with step 2
  * `df.loc[['Dean', 'Cornelia'], ['age', 'state', 'score']]` select two rows and three columns
  * `df.loc[:, ['food', 'color']]` all rows and two columns (but it is easier to use simple index)

* `df.iloc[ ]` uses *integer locations* for selecting rows and/or columns
  * `df.iloc[3]` select one row
  * `df.iloc[[5, 2, 4]]` multiple rows (NOT df.iloc[5, 2, 4])
  * `df.iloc[3:5]` a range or rows
  * `df.iloc[3:]` from 3rd until end
  * `df.iloc[3::2]` every second starting from 3rd element
  * `df.iloc[[2,3], [0, 4]]` two rows and two columns
  * `df.iloc[0, 2]` single row and single column

* `df.ix` DEPRECATED

Links:

* Part 1: https://medium.com/dunder-data/selecting-subsets-of-data-in-pandas-6fcd0170be9c
* Part 2: https://medium.com/dunder-data/selecting-subsets-of-data-in-pandas-39e811c81a0c
* Part 3: https://medium.com/dunder-data/selecting-subsets-of-data-in-pandas-part-3-d5704b4b9116
* Part 4: https://medium.com/dunder-data/selecting-subsets-of-data-in-pandas-part-4-c4216f84d388


## Generation of time ranges

### Problem

For many analysis problems, it is necessary to generate a range of intervals on a time axis. In particular, it is important for time series analysis. Having this functionality is as important as having the standard `range` function. For time axes, the task is more difficult than for integers because there are quite different scales from milliseconds to years, intervals of different lengths like weeks and different conventions how to label the intervals.

For any range, including date/time ranges, the following parameters are needed:

* `origin` or `base` of the raster which is a reference point (or phase). It could be start of some current interval like current hour, day or year or some absolute point like Unix Epoch. Note that this value is not an interval and it does not have a duration or length.
* `start`, `end`, `periods` (two of them have to be specified) allow us to determine the first and last elements in the range. Additional options could be useful which specify whether these values are inclusive or exclusive, that is, whether the range has to cover these points or not. Note that the length (periods) is a duration which can be specified either in original (date/time) units or as the (integer) number of intervals.
* `label` is intended for representing the interval. One approach is to represent intervals by either left border or right border. In addition, we need to specify whether we want to label intervals by the border value (for example, some date) or by the interval number (starting from origin).
* `closed` is intended for specifying which border - left or right - is inclusive (the other border is then exclusive).
* `freq` or `size` of one interval is a duration in some units like milliseconds or years.

Not all of these parameters are (well) represented and we described them in order to introduce the concepts behind time ranges.

### Classes and constants

Classes:

* `DatetimeIndex` (index of `Timestamp` elements)
  * A list of `Timestamp` is automatically coerced to `DatetimeIndex`
  * https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DatetimeIndex.html

* `PeriodIndex` (index of `Period` elements)
  * A list of `Period` is automatically coerced to `PeriodIndex`.
  * https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.PeriodIndex.html

* `TimedeltaIndex` (index of `Timedelta` elements)
  * https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.TimedeltaIndex.html
  * https://pandas.pydata.org/pandas-docs/stable/timedeltas.html

* `DateOffset` (dateutil.relativedelta)
  * https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.tseries.offsets.DateOffset.html
  * https://pandas.pydata.org/pandas-docs/stable/timeseries.html#timeseries-offsets
  * Offset aliases: http://pandas.pydata.org/pandas-docs/stable/timeseries.html#offset-aliases
  * https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#offset-aliases

Links:
* http://pandas.pydata.org/pandas-docs/stable/timeseries.html
* https://pandas.pydata.org/pandas-docs/stable/indexing.html

### `pd.date_range` function

The main function for generating time ranges is `pd.date_range`: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.date_range.html

`pd.date_range` takes range specification in its arguments and returns DatetimeIndex. The elements of the index are labels of range intervals.

Here are some notes about this function:
* It is not possible to specify length in absolute units as duration - the only way is to specify it as number of intervals in the `periods` parameter
* The `normalize` parameter is a limited way to specify origin. If it is true then it will stick to midnight. For example, there are numerous ways to generate a range with frequency 15 minutes. However, if `normalize` is true then it will start counting from midnight which is unambigous.

### Generating interval integer numbers

The standard approach returns labels which are interval borders and hence have a type of the main axis, that is, time stamps. In many cases, we actually do not need these time stamps but rather want to get interval numbers. For example, instead of producing all hour starts for a week as time stamps, we could a sequence of hour integer identifiers like Hour 23, Hour 24, Hour 25 and so on. This approach has the following advantages:
* Integer interval identifiers are shorter
* Integer interval identifiers are consecutive
* If properly defined, they provide a unique and unambigous way to identify time intervals

Unfortunately, this cannot be done using a standard function. However, this function can be implemented if we define the necessary time raster like hourly raster or week raster and then introduce integer identifiers by specifying the origin and other necessary parameters.

## Grouping and aggregation over time

### Problem of grouping

Let us assume that we have asynchronous events with arbitrary time stamps and our task is to aggregate data in these events by grouping them in intervals like 1 hour or 1 minute. The first task before we can apply aggregation is grouping, that is, each event has to be assigned some group.

### `resample` function

The main purpose of the `resample` function is to change the time range or raster of a time series: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.resample.html

Essentially, this function creates a time range and then groups all input events by using this raster. Note however that the time range (raster) is created internally and is not available in explicit form. Rather, the function returns a special `Resampler` object which includes information about grouping and can be used for aggregation.

It takes as input a time series and a specification of the time range used for grouping. The parameters of this time range are very similar to those used in the `date_range` function.

Notes:
* It works only with a date index in the input time series
* Example of finding average price per month: `df.set_index('date').resample('M')["price"].mean()`
* In addition, group by name: `df.set_index('date').groupby('name')["price"].resample("M").sum()`

### `Grouper` object

`Grouper` allows for encoding grouping conditions. It takes a column as a parameter and (in the case of time series) a frequency specification (along with other options). It is then passed to the groupby function which returns an object used later for aggregation:
```python
gb = df.groupby(Grouper(key='date', freq='60s'))
gb = df.groupby(['name', pd.Grouper(key='date', freq='A-DEC')])
gb = df.groupby(Grouper(level='date', freq='60s', axis=1))
prices = gb.['price'].sum()
```

### Rolling grouping and aggregation

Here the number of groups is equal to the number of records, that is, each record defines a group. This group consists of neigbour records. The main question and parameter is the size of the group. There are two main options:
* The maximum number of records in the group
* The distance of records from the record expressed in terms of time duration. Here we need to specify a date-time column 

Windows with maximum number of rows:
```python
r = s.rolling(window=60)
r.mean()
```

With of length 2 seconds:
```python
s.rolling('2s')
```

### Cumulative grouping and aggregation

This type of grouping is performed in the `expanding` function: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.expanding.html

#### Custom resampling with interpolation

Assume that we have asynchronous measurements and the task is to estimate aggregated values for each interval. The measurements can be at any moments within these interval, for example, very close to the interval borders or somewhere in the middle of the interval. Also, the distribution of measurements among intervals can vary: there are intervals with no measurements or with many measurements.

Links:
* `bfill()` backward filling
* `interpolate()`

## Modifying individual cells

```python
df.loc[x, c] = 1
df.at[4, 'B'] = 10  # Use instead of set_value which is deprecated
df.iat[]
```

### pandas `update` function

It has the semantics of *imposing* one object on another and in this sense is similar to dict.update.

We can control which target (original data frame) values to modify as follows:
* by choosing the desired index values
* by setting NA cells in another data frame which will not influence the original data frame (equivalent to removing this index and row)
* by setting NA cells in the original data frame which needs to be updated along with `overwrite` parameter `False`

Notes:
* Index is used for matching when imposing values
* The values are overwritten in-place
* The length never changes, that is, no new row are added
* For Series, the series name is treated as column name

Links: 
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.update.html

### pandas `combine_first` function

Notes:
* It can be viewed as an extended version of `fillna` but instead of using a constant value we use matching (by index) values from another (other) df
* Only NaN values from the original (target) data frame will be overwritten (by using matching values from the second (other) data frame. NaN is used as an indicator of what needs to be modified
* The number of rows and columns can increase. The row and column indexes of the resulting DataFrame will be the union of the two

Links: 
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.combine_first.html

### pandas `assign` function

Actually, it does not modify values in-place. Rather, it overwrites whole columns (in case they have the same name). 

It very simmilar to `apply` method and can be used instead of it if lambda function is used to compute new column values:

```python
df.assign(C=lambda x: x.A + x.B)  # New column C will be computed and appended to existing columns
```

Links:
* Link: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.assign.html

### Masked operations

Previous operations were assignments. Let us now assume that we want to modify existing values by applying some operation to them. This can be done by using `apply` or `assign`. Yet, they are applied to the whole column(s), that is, modify all values. What if we want to modify only a subset of values?

One approach is to a apply an operation to all values but the operation will return the same value for cell which do not have to be modifed. Another approach is to apply an operation to all values but this operation is known to not modify certain values.

This approach has one problem: sometimes the necessity to apply the operation and modify the cell does not depend on the cell value itself - it can depend on some external factors.

`numpy.putmask` modifies elements of an array which satisfy the specified condition.
For example, this code will add 2 to all positive elements.
```python
np.putmask(x, x>0, x+2)
```

A similar function exists in pandas. For example, this will assign 1.0 to all positive cells (`x` is either data frame or series):
```python
x.mask(df > 0, 1.0) 
```

See also `where`.

There are operations which take an explicit mask as a parameter so that any operation is applied to only a subset of elements. In this case, a mask is boolean array computed separately. For example, such a mask could represent only elements with odd or even indexes.

Below, only two first elements are used to compute the sum:
```python
>>> mx = ma.masked_array([1, 2, 3], mask=[0, 0, 1])
>>> mx.sum()
3.0
```

Notes:
* Link: https://docs.scipy.org/doc/numpy/reference/maskedarray.generic.html

## Appending rows and columns

Understanding the logic behind pandas functions can be quite difficult because pandas tries to combines concepts from several modeling paradigms (matrix, relatoinal, multidimensional).

When thinking about operations with data it is easier and conceptually more consistent to make the following assumptions:
* An operation is always *directed* where the first element is being updated by applying the second argument. In particular, it is different how join operation is treated.
* An operation does not change the data in the first element but rather appends new elements to it (rows or columns).
* An operation can remove some data from the first element but in this case no new elements are added.

### pandas `join` function

General solution consists in using join which will attach columns by matching the rows using index.

```python
>>> dat1.join(dat2)
```

If you want to includes rows from the both frames (useful only in special cases) then use outer join:
```python
>>> dat1.join(dat2, how='outer')
```
For example, this might be useful if the first (initial) data frame is empty and we want to attach new columns as they are genreated in a loop.

### pandas `concat` function

Concat is based on the same code (and logic) as join but has more specific purpose and semantics, that is, it is easier to understand what it is doing.

Concat columns (because `axis` is 1):
```python
>>> pd.concat([dat1, dat2], axis=1)
>>> dat1 = pd.concat([dat1, dat2], axis=1)
```

### pandas `append` function

Append is a shortcut for `concat` and it will always append rows to the end, that is, the result length will always be the sum of the lengths. Also, the original cells/values will never be changed. Indexes for appended rows depend on the argument: either retain (duplicates possible) or create new by continuing the existing index. Non-existing columns will also be added with their values. In the original rows, they will have None values.

Notes:
* Rows will be appended to the data frame
* Non-existing columns will be added to the data frame
* `ignore_index` influences only how the index for new appended rows is generated

Link: 
* http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.append.html

#### pandas `merge_asof` function (nearest merge)

TBD

## Finding values

The goal here is to find "coordinates" or "location" of some value(s). As usual, there are at least two possible "coordinates": labels (index values) or integer offsets.

### pandas `searchsorted` function

This function finds an insertion point for a value. If it is applied to series then the insertion point is a label (index value) which can be then used for access. When it is applied to an index or array then an integer offset is returned:

```python
df.index.searchsorted(row_label)
```
Note that the value might not really exist.

Links: 
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.searchsorted.html
* https://docs.scipy.org/doc/numpy/reference/generated/numpy.searchsorted.html#numpy.searchsorted

### pandas `get_loc` function

Find integer id (offset) for the specified label or raise a `KeyError` if it does not exist:

```python
df.index.get_loc(row_label)
```

## Python language

### Comparison and None

Comparison:

* 'is' is identity testing (reference comparison) equivalent to id(a) == id(b)
* '==' is equality testing (content comparison)

Check for non null:

```python
if val is not None:
if val: # Same as above
if val is None:
```

The following is not recommended (because controversies if val is a complex object like pandas data frame):
```python
val != None
```

### Pass arguments

Passing arguments: https://stackoverflow.com/questions/334655/passing-a-dictionary-to-a-function-in-python-as-keyword-parameters

```python
mydict = {'b': 100, 'c': 200}
test(a=3, **mydict)
```

Important is that the dict cannot overwrite the explicit arguments, and the dict cannot contain unspecified (in the function) arguments.	
	
One generic approach is to merge all possible sources of arguments into one dict and then use only this one dict. Also, we need to remove unspecified fields from this dict, for example, if this dict originates from some configuration.

How to merge two dicts: https://stackoverflow.com/questions/38987/how-to-merge-two-dictionaries-in-a-single-expression

Python 2:

```python
z = x.copy()
z.update(y) # which returns None since it mutates z
```

Python 3.5:

```python
z = {**x, **y}  # values from y will replace those from x
```

Delete entry from dict: https://stackoverflow.com/questions/5844672/delete-an-element-from-a-dictionary	

```python
d.pop("key")  # It mutates dict
del d['key']  # Mutates dict
```
python
Copy dictionary (shallow copy):

```python
r = dict(d)
```

### Internal structure of Python objects

Find a function in a module:

```python
import foo
method_to_call = getattr(foo, 'bar')
result = method_to_call()
```

or shorter

```python
import foo
result = getattr(foo, 'bar')()
```

Here you have to know already the module name. 'hasattr' can be used to determine if a function is defined. This version might be better:

```python
getattr(foo, 'bar', lambda: None)
```

Find a function by name:

* 'locals()["myfunction"]()' - use a local symbol table https://docs.python.org/3/library/functions.html#locals
* 'globals()["myfunction"]()' - use a global symbol table http://docs.python.org/library/functions.html#globals

If it is necessary to also import a module:

```python
module = __import__('foo')  # Python 2
module = importlib.import_module  # Python 3
func = getattr(module, 'bar')
func()
```

### Date and time

Definitions:	

* Unix time = POSIX time = UNIX Epoch time = number of seconds elapsed since 01.01.1970 00:00:00 UTC minus leap seconds.
* UTC (Coordinated Universal Time) - no daylight saving time - normally same as GMT but GMT is not precisely defined.

Links:

* Date transformations: https://stackoverflow.com/questions/8777753/converting-datetime-date-to-utc-timestamp-in-python/8778548#8778548

## Python environment, packaging, deployment

### User space

User space is where all packages are stored for only this user. It is analogous to the global dir but only for this user.

Install into user space: 

```python
pip install --user package-name
```

User-base binary directory is needed and must be in PATH. how to get user-base dir:

* Linux: "python -m site --user-base" (typically ~/.local) - and then add /bin to the end
* Windows: "python -m site --user-site" (typically AppData\Roaming\Python\Python36\site-packages) - and replace site-packages with Scripts

### virtualenv
	
virtualenv is a tool to create isolated Python environments. virtualenv creates a folder which contains all the necessary executables to use the packages that a Python project would need.

```
pip install virtualenv - install the tool
```

Create a virtualenv for a project:

```
cd my_project_folder 
virtualenv my_project
```

It will be in the current directory (where the project is) with Python executables and copy of pip (to install other packages). Exclude this folder from source control (or use standard name for all projects like 'env').

Use any python interpreter of your choice (or we can use envvar VIRTUALENVWRAPPER_PYTHON as a global parameter):

```
virtualenv -p /usr/bin/python2.7 my_project - 
```

Activate the current virtualenv (otherwise it will not be used):

```
my_project/bin/activate
```

From now on, all packages installed using pip will be placed in this virtualenv folder (not global).

Deactivate the current virtualenv (switch to system default):

```
deactivate
```

Delete the folder to delete the virtualenv

Freeze the current state of packages etc. in a file:

```
pip freeze > requirements.txt
```

Recreate the environment:

```
pip install -r requirements.txt"
```

### pipenv

Install pipenv in user space:
```
pip install --user pipenv 
```
Install globally:
```
pip install pipenv
```

Pipenv manages dependencies on a per-project basis. So we need to change into the project dir and do installation from it:

```
pipenv install some-package
```

It will install this package and update Pipfile (which tracks your dependencies).

For each project it will create locally (for the user): a separate virtualenv (in \.virtualenvs folder) and a separate Pipfile (in project folder)

```
pipenv install # No package. it will use requirements.txt and install the packages.
```

Links:
* https://docs.pipenv.org/
* http://docs.python-guide.org/en/latest/dev/virtualenvs/
