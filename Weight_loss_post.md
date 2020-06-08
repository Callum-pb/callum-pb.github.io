# Weight Tracking Visualisation

![](Weight_lost_so_far.png)

I have been logging my weight every morning in a Google Sheets worksheet, along with occasional body measurements. I've been tracking for over a year now and I thought it about time I look at what's changed.

Off the back of some programming courses, this would be my first stand alone Python projects exploring data. It was simple enough, just one set of time-series data that needed plotting.

My current setup is a pretty slow Chromebook, something I've never programmed on before. So I decided the easiest route was to use Google Colab, which uses Jupyter notebooks, and let them do all of the processing work.

I started by hooking up the notebook to my Google Account with the following:

```
from google.colab import auth
auth.authenitcate_user()
import gspread
from oauth2client.client import GoogleCredentials
gc = gspread.authorize(GoogleCredentials.get_application_default())
```

Running this allows use to log into your Google account and securely access any Google Sheets on the account.

After importing the usual modules (pandas, matplotlib and numpy) I loaded in my selected workbook, the weight log I've been keeping.

Then, it was time to load it into a Pandas DataFrame and start to clean it. The first step was to drop the first few rows as the spreadsheet has some info in the first for rows relating to starting weights, and some other information that I used when the spreadsheet was my primary analysis tool.


Top of CSV
                0            1          2  ...            4      5      6
0      04/06/2020           kg        lbs  ...                  kg    lbs
1    Start Weight        130.9      288.0  ...  Goal Weight    100    220
2  Current Weight       112.10      246.6  ...    Remaining  12.10  26.62
3     Weight Lost        18.80       41.4  ...                           
4                                          ...                           
5                                          ...                           
6            Date  Weight (kg)  7-day Avg  ...                           
7      12/05/2019        130.9          -  ...                           
8      13/05/2019        127.8          -  ...                           
9      14/05/2019        127.8          -  ...


After some cleaning, I had a DataFrame of just the relevant cells, now it was time to ensure the quality of the data was there, deal with the data types and formatting, and deal with the NaNs.

Top of data table
             0            1
6         Date  Weight (kg)
7   12/05/2019        130.9
8   13/05/2019        127.8
9   14/05/2019        127.8
10  15/05/2019        128.7
11  16/05/2019        128.5
12  17/05/2019        128.6
13  18/05/2019        128.6
14  19/05/2019        128.2
15  20/05/2019        128.1

After  setting the column headers, re-indexing and changing the data types I was good to start plotting.

Because there were two major periods of time missing from the log, I had to decide what to do with them visually. I wanted to show an interpolation, but make sure it was visually separate from the data that was actually recorded.

To do this I decided the quick and dirty route was to copy the weight data into a new column, and apply a linear interpolation to fill the gaps. I then made a boolean mask from the original data to use on the new interpolated column. The mask allowed me to select all the rows containing the original data and rewrite them as NaNs. This now allows more control of the interpolated data on the final plot.

I also know that daily logs come with a reasonable amount of variability, so a moving average would be useful in smoothing the curve. I applied a 7-day moving average to the original data and gave this it's own column, again for ease of plotting. I chose the 7-day window based on some previous analysis I had done in the spreadsheet, looking at minimising K-values and giving the closest trend line whilst also allowing it to react to new trends quickly.

The code for calculating these new columns, and their plots, are below:

```
#Add 7-day moving average column to df and then add to plot
weights['MovAvg'] = weights.WeightKG.rolling(min_periods=1,window=7,center=True).mean()
#seperate dotted plot for interpolated missing data - could add to either series but I'd like to show that it is interpolacted and not primary data
weights['Interp'] = weights['WeightKG'].interpolate(method='linear',limit_direction='forward') #New col, linear interpolate from begingin all missing data
weights['Interp'][weights.WeightKG.notna()] = np.nan #Create boolean showing where the not-null values are in original weights col, then set those rows in interp col to NAN so they don't show on plot

#Plots
weights.plot(x='Date',y='WeightKG',ax=ax, grid=True,style='b:',label='Recorded Weight') #Recored weights
weights.plot(x='Date',y='MovAvg',ax=ax,label = '7-day Avg') #7-day moving average - centered
weights.plot(x='Date',y='Interp',ax=ax,style='r:',label = 'Missing Data') #Interpolated data

```

And below is the final output:

![](Weight_lost_so_far.png)

The Colab notebook and csv can be found [here](https://github.com/Callum-pb/python_projects/tree/master/Weight_tracker)
