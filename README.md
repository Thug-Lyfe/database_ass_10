# database_ass_10

I wrote the logical data model with as little redudancy as possible, with 8 tables:
1. location (id (PK),location)
2. subject (id (PK),subject)
3. frequency (id (PK),frequency)
4. measure (id (PK),measure)
5. flak (id (PK),flak)
6. time (id (PK),time)
7. indicator (id (PK),indicator)
8. value (id (PK),value,locationID,subjectID,frequencyID,measureID,flakID,timeID,indicatorID)

It can be seen visually as: 

![data_sketch](https://github.com/Thug-Lyfe/database_ass_10/blob/master/data2.jpg "logical data model sketch")

Or more aesthetically pleasing xD  note: sry that it says indicator on the middle one, it should be value.

![data_final](https://github.com/Thug-Lyfe/database_ass_10/blob/master/data1.jpg "logical data model")

The tables were created as such:

```python
%sql create table subject(id serial primary key,subject varchar(100) unique);
%sql create table frequency(id serial primary key,frequency varchar(100) unique);
%sql create table measure(id serial primary key,measure varchar(100) unique);
%sql create table location(id serial primary key,location varchar(100) unique);
%sql create table flak(id serial primary key,flak varchar(100) unique);
%sql create table time(id serial primary key,time varchar(100) unique);
%sql create table indicator(id serial primary key,indicator varchar(100) unique);
```
```python
%%sql create table value(id serial primary key,value numeric, 
                         measureID int references measure(id), 
                         timeID int references time(id), 
                         subjectID int references subject(id), 
                         frequencyID int references frequency(id), 
                         flakID int references flak(id), 
                         locationID int references location(id), 
                         indicatorID int references indicator(id),
                         constraint uni_tup unique (measureID, timeID, subjectID,flakID,locationID,indicatorID));
```

The data was added with this method:

Note: at line 36 one can just change the filename to one of the other three to add them to the database as well.

Note_2: to do as this you need to have your csv files in the same dir as the notebook file like this:
![dir_showcase](https://github.com/Thug-Lyfe/database_ass_10/blob/master/dir.png "dir")
```python
import csv
import psycopg2

conn = psycopg2.connect("host=data dbname=appdev user=appdev")
cur = conn.cursor()
list_loc = %sql select location from location;
list_subject = %sql select subject from subject;
list_frequency = %sql select frequency from frequency;
list_measure = %sql select measure from measure;
list_flak = %sql select flak from flak;
list_time = %sql select time from time;
list_indicator = %sql select indicator from indicator;
list_value = %sql select * from value;
with open('lifeexpectancy.csv', 'r') as f:
    reader = csv.reader(f)
    next(reader)  # Skip the header row.
    for row in reader:
        if ((row[0],) not in list_loc):
            list_loc.append((row[0],))
            cur.execute(
            "INSERT INTO location (location) VALUES ('%s')" %
            row[0])
            conn.commit()
            
        if ((row[1],) not in list_indicator):
            list_indicator.append((row[1],))
            cur.execute(
            "INSERT INTO indicator (indicator) VALUES ('%s')" %
            row[1])
            conn.commit()
            
        if ((row[2],) not in list_subject):
            list_subject.append((row[2],))
            cur.execute(
            "INSERT INTO subject (subject) VALUES ('%s')" %
            row[2])
            conn.commit()
            
        if ((row[3],) not in list_measure):
            list_measure.append((row[3],))
            cur.execute(
            "INSERT INTO measure (measure) VALUES ('%s')" %
            row[3])
            conn.commit()
            
        if ((row[4],) not in list_frequency):
            list_frequency.append((row[4],))
            cur.execute(
            "INSERT INTO frequency (frequency) VALUES ('%s')" %
            row[4])
            conn.commit()
            
        if ((row[5],) not in list_time):
            list_time.append((row[5],))
            cur.execute(
            "INSERT INTO time (time) VALUES ('%s')" %
            row[5])
            conn.commit()
            
        if ((row[7],) not in list_flak):
            list_flak.append((row[7],))
            cur.execute(
            "INSERT INTO flak (flak) VALUES ('%s')" %
            row[7])
            conn.commit()
        
        list_flak.append(row[6])
        cur.execute(
        "INSERT INTO value (value,measureID,timeID,subjectID,frequencyID,flakID,locationID,indicatorID) "
        "VALUES "
            "(%s"
            ",(select id from measure where measure.measure like '%s')"
            ",(select id from time where time.time like '%s')"
            ",(select id from subject where subject.subject like '%s')"
            ",(select id from frequency where frequency.frequency like '%s')"
            ",(select id from flak where flak.flak like '%s')"
            ",(select id from location where location.location like '%s')"
            ",(select id from indicator where indicator.indicator like '%s'))" % 
            (row[6],row[3],row[5],row[2],row[4],row[7],row[0],row[1])
            )
        conn.commit()
```

The growth rate was found with this method: 

Note: The worst growth rate can be found by changing the order to asc instead of desc

Note_2: We exclude locationID 43,44,42,49 in our case as this is, OECD, OECDE, EU28, EA19 as these are not countries

Note_3: The best growth is China and the lowest is Iceland (low capita sucks)
```python
%%sql select location.location,sum(value) as value_sum,count(value) as value_co, ((max(value) - min(value)) / count(value))  as "Growth rate" from value
join location on (location.id = locationID)
where indicatorID = 1
AND measureID = 1
AND locationID not in (43,44,42,49)
group by location.location
order by "Growth rate" desc
limit 2;
```

For plotting I first needed the tertiary, below secondary and above secondary education percentage numbers, with coresponding lifeexpectancy numbers. All of which i define to be used later like this:

Note: I used usa instead of china, even though it had less growth, this was done as china only had one year where they published their education records, which would make for a boring plot...
```python
usa_try = %sql select value as "try",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'USA' AND subject like 'TRY'
usa_below = %sql select value as "below",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'USA' AND subject like 'BUPPSRY'
usa_upper = %sql select value as "upper",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'USA' AND subject like 'UPPSRY'
isl_try = %sql select value as "try",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'ISL' AND subject like 'TRY'
isl_below = %sql select value as "below",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'ISL' AND subject like 'BUPPSRY'
isl_upper = %sql select value as "upper",lifeexp from value join location on (location.id = locationID) join subject on (subject.id = subjectID) join (select timeID,value as lifeexp from value where indicatorID = 3 AND subjectID = 1 AND locationID = 30) as lifeexp on (lifeexp.timeID = value.timeID) join time on (time.id = value.timeid) where location like 'ISL' AND subject like 'UPPSRY'
```

The plots can then be made like this:
```python
us_try_x = []
us_try_y = []
for ele in usa_try:
    us_try_x.append(ele[0])
    us_try_y.append(ele[1])
    
us_below_x = []
us_below_y = []
for ele in usa_below:
    us_below_x.append(ele[0])
    us_below_y.append(ele[1])
    
us_upper_x = []
us_upper_y = []
for ele in usa_upper:
    us_upper_x.append(ele[0])
    us_upper_y.append(ele[1])

is_try_x = []
is_try_y = []
for ele in isl_try:
    is_try_x.append(ele[0])
    is_try_y.append(ele[1])
    
is_below_x = []
is_below_y = []
for ele in isl_below:
    is_below_x.append(ele[0])
    is_below_y.append(ele[1])
    
is_upper_x = []
is_upper_y = []
for ele in isl_upper:
    is_upper_x.append(ele[0])
    is_upper_y.append(ele[1])

plt.xlabel('% of population educated')
plt.ylabel('lifeexp')
plt.title('stay in school = infinite life for Iceland')
plt.text(40,78,'cyan=upper\ngreen=below\nred=try',fontsize=12)
plt.plot(is_try_x,is_try_y,'ro',is_below_x,is_below_y,'go',is_upper_x,is_upper_y,'co')
plt.show()

plt.xlabel('% of population educated')
plt.ylabel('lifeexp')
plt.title('stay in school = infinite life for usa')
plt.text(64,77,'cyan=upper\ngreen=below\nred=try',fontsize=12)
plt.plot(us_try_x,us_try_y,'rp',us_below_x,us_below_y,'gp',us_upper_x,us_upper_y,'cp',)
plt.show()
```
And they look like this: where green is below secondary level, cyan is above secondary level and red is tertiary level education.

The x axis is the percentage of adults at that level of education

The y axis is the life expectancy.

![plot_iceland](https://github.com/Thug-Lyfe/database_ass_10/blob/master/iceland.png "iceland plot")
![plot_usa](https://github.com/Thug-Lyfe/database_ass_10/blob/master/usa.png "usa plot")

From this it can be hypothesized that when ones living standard is generally higher, then one tend to stay longer in school.

This is not to be confused with the hypothesi that people who have higher education live longer. As this would need a correlation between the age of people and their education level at death, instead of a nation wide life expectancy and general education level at that a particular year. Though it has been shown in other studies that it is true, you cannot use this data to do the same.


