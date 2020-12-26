# Data Modeling with Postgres

## Context
The startup **Sparkify** wants to analyze the data they've collectied on songs and users. The analytics team is particularly interested in knowing which songs their users are listening to. The data is stored in JSON format in flat files, which makes it difficult to access and analyze. The startup wants to store the data in tabular format in a relational database in a way that's easily accessible. I've been hired as a data engineer to design the data model and to build the ETL pipelines.

## Approach
I create a database in Postgres and store the data using the Entity-Relation model. Then I use python to create the ETL pipelines, where I transform the data from JSON format into tabular format and load it into the Postgres database. Below I provide more description on the data model, the files included, the overall workflow and the ETL process.

## Data Modeling
As mentioned previously, Sparkify's data is stored in JSON format. There are two main datasets: songs and logs.  
Sample from songs dataset:
```{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}```. 

I've used the Star Schema for this project. There is one main fact table tha contains all the measures associated to each event (each song played). There are four dimension tables, each with a primary key referenced in the fact table. Below are the details of each of those tables.  

* ###### Songplays (fact table)
* ###### Users (dimension table)
* ###### Songs (dimension table)
* ###### Artists (dimension table)
* ###### Time (dimension table)
  
## Project Files
1. **data** folder contains all the logs and songs JSON files.
2. **sql_queries.py** contains all SQL queries needed to drop, create tables, and insert rows. Queries are stored in python variables.
3. **create_tables.py** actually drops and creates tables. This python script makes the connection with the database and defines functions that will take in the SQL queries in *sql_queries.py* as parameters. 
4. **test.ipynb** is a python notebook that allows us to check our work and explore the tables created.
5. **etl.ipynb** is a python notebook that serves as a draft for the main ETL file.  
6. **etl.py** reads and processes files from song_data and log_data and loads them into the fact and dimensions tables. 
7. **README.md** provides overview of the project.

## Workflow
1. Create DROP, CREATE and INSERT statements in *sql_queries.py*. Then in *create_tables.py* we make a connection to the database and actually run the DROP and CREATE statements.
2. Verify that all tables were created successfully by running *create_tables.py* in test.ipynb ```%run create_tables.py ```.
4. In *etl.ipynb* we access the JSON data, parse through it, and create pandas data frames that will eventually be used to create the fact and dimension tables.
5. After applying filters and conversions, we lock the the conceptual ETL logic in *etl.ipynb*. Then we generalize the process in *etl.py* (details below).
6. Finally ran *etl.py* in test.ipynb ```%run etl.py``` and verify the tables were succesfully filled with appropriate data.

## ETL pipeline
At this point all tables, dimensions and facts, have been created.
1. Connect to the sparkify database.
2. We access the song_data directory and use pandas to read the JSON files into a data frame.
3. From the songs data, we create song_data and artist_data, pandas data frames containing the data needed for the tables *songs* and *artists*.
4. Finally we insert the data into their Postgres tables *songs* and *artists*.
5. We repeat the process above for the log data in the log_data directory.
6. Instead of using all of the log data, we use the filter ```page = 'NextSong'``` 
7. Additionally, we do have to transform the ```ts``` column. The requirement is for the ```start_time``` column in *song_plays* to be a timestamp. So we convert ```ts``` from integer to timestamp. 
8. We also use the newly created timestamp column and get more date related fields from it to finally create the *time* data frame, which will eventually turn into the *time* Postgres dimensions table in our database.
9. Next we load user data into our *users* table
10. In order to fill the *songplays* table, we need to get the ```song_id``` and ```artist_id```. So we create a query, which gets both ids using the song title, the artist's name, and the song duration in seconds.
    ```
    song_select = ("""SELECT s.song_id ,s.artist_id
        FROM songs s JOIN artists a 
        ON s.artist_id = a.artist_id
        WHERE s.title = %s AND a.name = %s AND s.duration = %s;""")
    ```
13. Finally, we fill the *songplays* tables with the necessary data.
