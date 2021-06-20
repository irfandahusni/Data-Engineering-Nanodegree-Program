# Project : Data Modeling with Postgres 

## Goals and Purpose

This Udacity Data Engineering Nanodegree project aims to create a `sparkifydb` database schema and its ETL pipeline for a startup called <i>Sparkify</i>. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app. Which risen the need to do the data modeling that produces the database. This database then will be used by the analytics team to further analyze what song their users are listening to. 

## Datasets

This database consisted from two JSON datasets `song dataset` and `log dataset`.
<ol>
 <li><strong>Song Dataset</strong></li>
 
The first dataset is song dataset that originated from this [source](https://labrosa.ee.columbia.edu/millionsong/), where the files directory looks like these

 `song_data/A/B/C/TRABCEI128F424C983.json`
<br>
 `song_data/A/A/B/TRAABJL12903CDCF1A.json`
 
 and the sample of one of its dataset is like this 
 
`{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}`

 <li><strong>Log Dataset</strong></li>
The second dataset is a log files which comes in JSON format generated from this [source](https://github.com/Interana/eventsim), it is an
event simulator based on the songs in the dataset above. These simulate activity logs from a music streaming app based on specified configurations. The dataset is partitioned 
partitioned by year and month. This is a sample path to two a file in this dataset

 `log_data/2018/11/2018-11-12-events.json`
 
 and below is an example of what the data in a log file, 2018-11-12-events.json, looks like.
`![data preview](https://github.com/irfandahusni/Data-Engineering-Nanodegree-Program/blob/main/1.%20Songplay%20Analysis/picture/log-data.png?raw=true)
</ol>

## Database Schema 

The database schema forms a star schema that has 1 <i>fact</i> table (songplays) and 4 <i>dimension</i> tables (users, songs, artist. time). 

`![database schema](https://github.com/irfandahusni/Data-Engineering-Nanodegree-Program/blob/main/1.%20Songplay%20Analysis/picture/songstartupERD.png?raw=true)

## ETL Pipeline 
### 1. Song Dataset ETL

 The song and artist tables are created from the song dataset file. These two tables come from this raw file in JSON format : 
 ```json
{
  "num_songs": 1,
  "artist_id": "ARD7TVE1187B99BFB1",
  "artist_latitude": null,
  "artist_longitude": null,
  "artist_location": "California - LA",
  "artist_name": "Casual",
  "song_id": "SOMZWCG12A8C13C480",
  "title": "I Didn't Mean To	",
  "duration": 218.93179	,
  "year": 0
}
```
below is the sample data of the songs table 

| song_id            | title                          | artist_id          | year | duration  |
|--------------------|--------------------------------|--------------------|------|-----------|
| SOMZWCG12A8C13C480 | I Didn't Mean To               | ARD7TVE1187B99BFB1 | -    | 218.93179 |

and below is the sample data of the artists table 

| artist_id          | name      | location        | lattitude | longitude |
|--------------------|-----------|-----------------|-----------|-----------|
| ARD7TVE1187B99BFB1 | Casual    | California - LA | -         | -         |

### 2. Log Dataset ETL 

The log dataset is used to produce users and time table. The raw dataset looks like this 

```json
{
 "artist":null,
 "auth":"Logged In",
 "firstName":"Walter",
 "gender":"M",
 "itemInSession":0,
 "lastName":"Frye",
 "length":null,
 "level":"free","location":"San Francisco-Oakland-Hayward,  CA",
 "method":"GET",
 "page":"Home",
 "registration":1540919166796.0,
 "sessionId":38,
 "song":null,
 "status":200,
 "ts":1541105830796,
 "userAgent":"\"Mozilla\/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/36.0.1985.143 Safari\/537.36\"","userId":"39"
 }
```
The name index then extracted to the users and time table where they look like these

<strong>Time Table</strong>
| start_time                 | hour | day | week | month | year | weekday |
|----------------------------|------|-----|------|-------|------|---------|
| 2018-11-30 00:22:07.796000 | 0    | 30  | 48   | 11    | 2018 | 4       |

<strong>Users Table</strong>
| user_id | first_name | last_name | gender | level |
|---------|------------|-----------|--------|-------|
| 91      | Jayden     | Bell      | M      | free  |

### 3. Create Songplays Table 
Songplays table is generated using `INNER JOIN` of `artist` table and `users` table where the end result looks like this 


| songplay_id | start_time                 | user_id | level | song_id | artist_id | session_id | location                            | user_agent                                                                                                              |
|-------------|----------------------------|---------|-------|---------|-----------|------------|-------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| 1           | 2018-11-30 01:08:41.796000	 | 73      | paid  | -       | -         | 1049       | Tampa-St. Petersburg-Clearwater, FL | "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.78.2 (KHTML, like Gecko) Version/7.0.6 Safari/537.78.2" |

