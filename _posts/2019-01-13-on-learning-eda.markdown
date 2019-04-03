---
layout: "post"
title: "On Learning EDA"
date: "2019-01-13 21:48"
---

### Introduction

This post is going to document the process for my learning and practicing EDA (Exploratory Data Analysis) and completion of the first project in the Metis bootcamp. This was a group project where I worked with two other members of the cohort to complete the project. The purpose of this project was to analyze data from turnstile counters at MTA stations using data downloaded from the MTA website to help a potential client discover ways to properly allocate marketing resources, and to help us gain a firmer understanding of the principles and methods of data wrangling and EDA in the process. 

The data came in basic CSV files structured like this:

```
CSV File:
- "C/A"
- "UNIT"
- "SCP"
- "STATION"
- "LINENAME"
- "DIVISION"
- "DATE"
- "TIME"
- "DESC"
- "ENTRIES"
- "EXITS
```

Individual turnstiles could be viewed by grouping the file by `C/A`, `UNIT`,  `SCP`, and `STATION`, with a concatenation of the four indicating a single turnstile. After grouping for turnstiles, the columns we were most interested in were `DATE`, `TIME`, `ENTRIES`, and `EXITS`. Entries and exits were collected as cumulative sums, meaning they were counting up from a predetermined starting point for every event that occurred since. Dealing with this was a major portion of the cleaning process in this project. 

### Tools

The tools used to complete this project were mostly Python libraries that we had learned about last week in the bootcamp. These primarily consisted of the Pandas library for data cleaning, manipulation, and transformation, and Matplotlib and Seaborn for creating plots and visuals. Other modules were used as well, such as datetime for creating datetime objects from dates and times, and pickle for creating pickle files which could be shared between team members for use in various Jupyter Notebooks. Speaking of which, all of the cleaning and analysis for this project was done in Jupyter Notebooks.

### Collecting And Cleaning

As mentioned earlier, the data for this project came from the MTA website in a CSV format. We ended up using the Python url module for collecting the data into a Pandas data frame. We made a decision to focus primarily on data from May of 2018, thinking that this sample of the data would be most likely to represent a similar pattern of activity to incoming data from May of 2019. Adding and analyzing more data from various time frames was discussed and likely would have been pursued with more time. 

The first step in cleaning the data was to deal with this issue of the cumulative sum. We initially pursued a strategy of using the Pandas `diff` function to subtract the next row from each preceding row to get totals for each time period by turnstile. This ended up not working, so instead we used the Pandas `shift` function to create new columns in the data frame shifted by one row from the row prior, yielding columns titled `PREV_ENTRIES` and `PREV_EXITS` containing the cumulative sum from the previous row. The next step was to simply subtract each entry and exit from the cumulative sum of the previous entries and exits yielding the total activity for that time period. 

```
df = df.sort_values(turnstile + ['DATE', 'WEEKDAY', 'TIME']) 
df[['PREV_TIME','PREV_ENTRIES','PREV_EXITS']] = \
                            (df.groupby(turnstile)['TIME','ENTRIES','EXITS']\
                            .transform(lambda x: x.shift(1)))
```

The subtraction process would sometimes yield negative values, which was obviously illogical. We corrected this by taking the absolute value of each subtraction and filtering out values which were so extreme as to be beyond the realm of possibility. We determined this value to be about 4000 entries or exits. The function for calculating the absolute value and filtering was fairly simple. 

```
def get_entry/exit_counts(row, max_counter):
    counter = row["ENTRIES"] - row["PREV_ENTRIES"]
    if counter < 0:
        counter = -counter
    if counter > max_counter: 
        return 0 
    return counter
```

Applying this function to the `ENTRIES` and `EXITS` columns gave us our total values for each time period. 

We also decided to add a column for extracting weekdays from the `DATE` column to allow us to do analysis by day of the week. This was fairly straightforward using Python’s datetime module. 

```
df['DATETIME'] = pd.to_datetime(df.DATE +" " + df.TIME, format='%m/%d/%Y %H:%M:%S')
df['WEEKDAY_NUM'] = [dt.weekday() for dt in df['DATETIME']]

weekdays = {
    0: 'Monday',
    1: 'Tuesday',
    2: 'Wednesday',
    3: 'Thursday',
    4: 'Friday',
    5: 'Saturday',
    6: 'Sunday'
}

df['WEEKDAY'] = [weekdays[day] for day in df['WEEKDAY_NUM']]
```

In addition, we added a column containing total activity by adding the entries and exits columns together. Some thought went into this decision, but we determined that total activity would yield a greater chance of success in marketing efforts than simply focusing on entries or exits individually.

The remainder of the cleaning process consisted of dropping duplicate data and `Nan` values from the data frame and eliminating now unnecessary columns, such as `PREV_ENTRIES` and `LINENAME`. Pickling was used to save this data frame externally so various team members could use it to conduct further analysis in their own notebooks.

### Analysis

The key to our analysis was determining the busiest stations and busiest days of the week and cross-referencing the two. The first step was determining the busiest days of the week, which resulted in our basically eliminating weekends from our analysis due to their relative absence of activity. 

We wanted to determine what was the busiest weekday for each station, which required some manipulation. We grouped the data by station and by weekday, sorted it according to activity, yielding a table with seven rows for each station according to each day of the week, sorted in descending order. Extracting the first row from each seven row group gave us the busiest day for each station. 

```
df.sort_values(['STATION','WEEKDAY_NUM'], inplace=True)
stations_by_weekday = df.groupby(['STATION','WEEKDAY_NUM','WEEKDAY']).sum()
stations_by_weekday.reset_index(inplace=True)

stations_by_weekday.sort_values(['STATION','TOTAL_ACTIVITY'], ascending=False, inplace=True)
stations_by_weekday = stations_by_weekday.groupby('STATION').first()
stations_by_weekday.sort_values('TOTAL_ACTIVITY', ascending=False, inplace = True)
```

We decided to focus exclusively on the busiest 100 stations, as different analysis had suggested to us that the activity at the busiest stations far outweighed that at the average station. 

Further analysis was done using to visualize total entries and exits for all stations and to dive into only the top five percent of stations to view their total activity across the whole month. The analysis done here was useful for determining which stations to focus our efforts on.

The next phase of the analysis was determining the frequency of stations for each of the busiest days. We discovered that Wednesday and Thursday showed the most traffic for the busiest stations, followed by Friday and Tuesday. We plotted the traffic at the ten busiest stations for these days and also explored weekday versus weekend traffic. 

### Recommendations

In coming up with the presentation, we delivered recommendations for which stations to target and on which days. We specifically mentioned seven stations that showed over three million entries and exits on the busiest days across the month of May. 

### Future Work

With more time, we would have liked to do more analysis regarding different times of the day and delivering more recommendations regarding not only day of the week and station, but also the optimal time to properly allocate resources. Though this data would have likely confirmed our intuitions, it would still have been useful for the presentation. 

We also would have eventually liked to give more granular recommendations by merging our data frame with economic and demographic data for the different neighborhoods and zip codes of New York. This would have been a much more complicated project, but would probably well worth the time.

### Learning

This project helped me to really hone my data wrangling abilities and develop a much more comprehensive understanding of the principles of EDA and the techniques involved. I also gained a significantly greater understanding of the primary Python tools for data analysis and feel much better prepared to use these tools for more complicated analysis in the future. 