Title: Pandas Snippets
Date: 2019-04-22
Category: Python-Package
# Pandas Snippets
## Recommended Practices
- `loc` command is the most recommended way to set values for a column for specific indices.
- 

## plot
### subplot
```python
fig = plt.figure(figsize = (16, 6), dpi = 80)
ax1 = fig.add_subplot(131)  # 1*3 的图像， 在第一个位置
sns.violinplot(x = tmp_series, ax = ax1, label = 'ax1', color = 'y')
ax1.set_xlabel('ax1')
ax1.set_title('ax1', fontsize = 22, loc = 'center')
ax1.legend()
ax1.set_xlim(0, 200)

ax2 = fig.add_subplot(132)
ax3 = fig.add_subplot(133)
```
### hist
```python
values_counts_series = df.groupby['col1']['col2].nunique()
values_counts_series.hist(bins = 50, figsize = (16, 9))

```
### box
```python
values_counts_series = df.groupby['col1']['col2].nunique()
values_counts_series.plot.box()

ax = sns.boxplot(x = df['cols1'])
ax.set_xlim(0, 100)
```
### bar
```python
def autolabel(rects):
    '''
    attach some text label
    '''
    for rect in rects:
        height = rect.get_height()
        ax.text(rect.get_x() + rect.get_width() / 2., 1.02 * height,
                '%.2f' % round(height, 2),
                ha = 'center', va = 'bottom')

fig, ax = plt.subplots(figsize=(8, 6))
ind = np.arange(3)
width = 0.35
rects1 = ax.bar(ind, x, width, color = 'b')
```
### line
```pyton
df[df['col1'] == m].plot(x = 'col2', y = ['col3', 'col4'], figsize = (16,9))
```

## read 
```python
# from  list of dict 
df = pd.DataFrame.from_records(list_dict)
```
## datetime func 

```python
# read and convert to datatime64 format
df= pd.read_csv(filepath, parse_dates=['col1','col2'], infer_datetime_formate = True)

# to_datetime
df['col'] = pd.to_datetime(df['col'], format = '%Y/%m/%d %H:%M')


import datetime
pd[pd['date_col'].dt.date == datetime.date(2018, 12, 9)]
df['date_col'] = df['time_col'].dt.date
df['month_col'] = df['time_col'].dt.month
df['week_col'] = df['time_col'].dt.weekofyear
df['wd_col'] = df['time_col'].dt.weekday
df['hour_col'] = df['time_col'].dt.hour
df['day_of_year_col'] = df['time_col].dt.dayofyear
df.loc[:, 'minutes_cols'] = (df.loc[:, 'off_time'] - df.loc[:, 'on_time']).apply(lambda x: x.seconds / 60)

# label on holiday and weekday
def labelOnDate(date):
    '''
    描述：把日期转换为对应的工作日或者节假日
    返回：0：工作日 1： 节假日 2：周末
    '''
    holiday_list = []
    workday_list = [0, 1, 2, 3, 4]
    weekend_list = [5, 6]
    date_str = date.strftime('%Y-%m-%d')
    whatday = date.weekday()
    if date_str in holiday_list:
        return 2
    elif date_str in workday_list:
        return 0
    elif date_str in weekend_list:
        return 1

df['label_on_date'] = df['date_cols'].apply(labelOnDate)

# type convertion
## from object to datetime64[ns]
import datetime as dt
df['dt64_date'] = df['oj_date'].apply(lambda x: dt.datetime.strptime(x, '%Y-%m-%d'))

# resample
## 按周统计
tmp = arr.resample('W').sum()
## 按天统计
tmp = arr.resample('D').sum()

```
## unchanged shape operations for a dataframe

### operation on rows or columns
```python
# block sample
def block_sample(df=pd.DataFrame(), block_size = 6, num_samples = 3):
    samples = [df.iloc[x: x + block_size] for x in np.random.randint(len(df), size = num_samples)]


# find specific rows or columns
trip_df[(trip_df['odo_dif'] == 0) & (trip_df['did_time'] < pd.Timedelta('00:03:00')) & (trip_df['dis_soc'] < 0.3))]

# 查看缺失行
df[df['loss_cols'].isnull()]
## 查看缺失行所在索引
df[df['loss_cols'].isnull()].index.tolist()
all_df.iloc[pd.isnull(all_df).any(1).nonzero()[0], :]
## 查看缺失情况
df.isnull().sum().sort_values(ascending = False)
# 查看零值情况
all_df['on_odo'].astype(bool).sum(axis = 0)

## change sequence
df = df.sort_values(by = ['col1', 'col2'], axis = 0, ascending = [True, True])

## str operations
df = df[df['cols'].str.contains('specfic_words | other_words1 | other_words2')]
# remove single quotes
df['str_cols'] = df['str_cols'].str.replace("'", "")
# concat columns
df['str_concat_col'] = df['str1'].str.cat(df['str2'], sep = '-')
```

## change shape operations for a dataframe
```python
# add columns based on other columns values or diff values bet rows
df['last_row_diff'] = df['cols_2'] - df.groupby['col_1']['col_2'].shift(1) 
def add_label(row):
    if row['col_1'] >= pd.Timedelta('00:10:00'):
        return row['col_3']
    if pd.isnull(row['last_row_diff']):
        return row['col_3']
    return np.nan;
df['label'] = df.apply(lambda row: add_label(row), axis = 1)

# delete
all_df.drop_duplicate(subset=['vin', 'on_time'], keep = 'first', inplace = True)

# filter
tm_vin = charge_days_series[charge_days_series > 12].index.values
all_df = all_df[all_df['vin].isin(tm_vin)] 
# all_df = all_df[~all_df['vin].isin(tm_vin)]
```

### shift within group
- shift down values by rows within a group
```python
gb = df.groupby('vin)
df['last_odo] = gb['odo'].shift(1)
```
or use `transform`
```python
df['B_shifted'] = df.groupby(['A'])['B'].transform(lambda x:x.shift())
```
In pratice, a time-saving method as follows:
```python
df = df.sort_values(by=['name','timestamp'])
df['time_diff'] = df['timestamp'].diff()
df.loc[df.name != df.name.shift(), 'time_diff'] = None
```
### sort
- sort by multiply columns
    ```python
    charge_df.sort_values(['vin', 'on_time'], ascending = ['True', 'True'], inplace = True)
    ```
### add/delete columns
```python
# based on  other columns of other rows
def diffFunc(arr):
    return (arr - arr.shift(1)).apply(lambda x:x.days)
df['diff_days'] = df.groupby('bycol').transform(diffFunc).fillna(0)

# based on other columns within same rows
df['new_col'] = df.apply(lambda x: x['col1'] / x['col2'], axis = 1)

# delete duplicate
df.drop_duplicates()

# add columns based on 'windows-like' rows
df['windows_key_list'] = pd.Series(df['key'].str.cat([df.groupby(['bycol']).shift(-i)['key'] for i in range(1, windows_size)], sep = ' ')

# add columns based 'str-value' column 
def lenOfStr(x):
    return len(str(x).split())
df['new_col'] = df['str_col'].apply(lenOfStr)

# add columns  counting depulicated times across rows
def countRepeat(x, duplicate_times):
    if pd.isnull(x):
        return np.nan
    else:
        L = str(x).split()
        filtered_L = [item for item, count in collections.Counter(L).items() if count >= duplicate_times]
        return filtered_L
df['filter_duplicate_item'] = df['duplicate_str_col'].apply(countRepeat, args = (duplicate_times, ))
```
### changes columns values
```python
# fill zero value with other columns values
charge_df['on_odo'] = np.where(charge_df['on_odo'] == 0, charge_df['off_odo'], charge_df['on_odo'])

# replace
df['cols'] = df['cols'].replace({0 : np.nan})

# fillna with mean
df['cols'] = df['cols'].fillna(df.groupby(['vin', 'on_odo'])['cols'].transform('mean'))

# fillna 
df['cols'].fillna(method='ffill', inplace = True)

# fillna with 0
df.update(df['cols'].fillna(0))

# fillna with mean within group
def replace(g):
    mask = g == 0
    g[mask] = g[~mask].mean()
    return g

df['gnss'] = df.groupby(['vin', 'on_odo'])['gnss'].transform(replace)
# or
df['gnss'] = df.groupby(['vin', 'on_odo'])['gnss'].transform(lambda x: x.where(x != 0).fillna(x[x != 0].mean()))

# fill with other columns values of last row 
cond = ((df['on_odo'] != 0) & (df['on_odo'] == df['off_odo'].shift(1)))
df['on_gnss'] = df['on_gnss'].mask(cond).fillna(method = 'ffill')

```

## create a dataframe based on a dataframe
### copy vs slice
```python
df1 = df.copy()
```
### apply
```python
def func(x):
    v1 = len(x)
    # v2 = 
    return pd.Series([v1, v2], index = [v1_name, v2_name])
df_gby = df.groupby('bycol')
df_temp = df_gby.apply(func)
```
### agg 
```python
# groupby + agg
df_gby_agg = df.groupby('col1').sum().reset_index()

```
- basic agg on many columns groupby one column
```python
agg_df = df.groupby('vin).agg({
    'on_odo': 'min', 'off_odo': 'max', 'loss_odo': np.sum})
```

- agg on many columns using built-in func 
```python
new_df = df.groupby(['bycol1', 'bycol2']).agg({'col1': 'first', 'col2': 'last'}).reset_index()
```

- agg use self-defined func
```python
def col_avg(arr):
    tmp = arr.value_counts()
    tmp = tmp.reindex(range(tmp.index.min(), tmp.index.max() + 1), fill_value = 0)
    return tmp.mean()
col_vag_df = df.groupby('bycol')['col'].agg(col_avg)

```

- agg on many columns  and on same column with different func
```python
def head_ind(x):
    return x.iloc[0]
df = df.groupby(['bycol1', 'bycol2']).agg({'col1': 'min', 'cols2': 'max',
                                          'col2': [('total', np.sum), ('head', head_ind)]}).reset_index()
df.columns = df.columns.map('_'.join)
# or as follows
df_temp = df[['col1', 'col2']].groupby(['bycol']).agg({'col1':[func1, func2],
'col2':[('func1', func1), ('func2', func2)]})
df_temp.columns = df_temp.columns.get_level_values(1)
```

### concat
```python
df = pd.concat(df1.iloc[m1:m2, :],df2.iloc[m1:m2, :], ignore_index = True)

# concat by adding columns
merge_df = pd.concat([df0, df1, df2], axis = 1)
```
### merge
```python
merge_df = df.merge(df0, on = 'key', how = 'left').merge(df1, on = 'key', how = 'left').merge(df2, on = 'key', how = 'left')

```

## operations on index
```python
charge_df.reset_index(drop=True, inplace = True)

# change columns sequence
col_names = ['col1', 'col2', 'col3', 'col4']
df = df.reindex(columns = col_names)

# 
df.columns = df.columns.map('_'.join)

# change name
df.rename(columns = {'old_col1': 'new_col1'}, axis = 1, inplace = True)


```

## count
```python
df['col1'].groupby(df['col1']).count().sort_values()
```


