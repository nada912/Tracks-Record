# Tracks-Records -  Visualization project
Preprocessing of various files containing the listening history of different users.

##How to start and run the project :

### Prerequisites : 
Have a data bricks account (If not, create an account with your professional email on databricks).
### Step 1 : Create a cluster on Databricks
	On the menu bar, click on “Compute” then “Create cluster” button.
	Choose a name then create.
### Step 2 : Import the data to Databricks
	-   On the menu bar, click on “Catalog”, then “Create table” button.
	-   Name the target directory “data” then upload all the text files at once.
### Step 3 : Open the notebook
    On the menu bar, click “New” -> “Notebook”.
    Once the notebook is created, click on “File” -> “Import” to import the notebook available on Github.
    Now execute the cells to get and process the data then load it to the database.
### Step 4 : At this point everything is done, you can visualize the dashboard on PowerBi.
Connect To PowerBi and select “Get Data from other source”
Select on the left sidebar “Database” -> “PostgreSQL Database”
Connect with the following and open the Report Tracks.pbix file :

	Host : http://aws-0-eu-central-1.pooler.supabase.com
	Database name: postgre
	user :  postgres.vozazcxddifukxwhauox
	password: PasswordTracks2024.
Reconnect to the database in case you have trouble visualizing the dashboard

# Conception and implementation process:
## 1. What Is It?
This project is designed to analyze and visualize music listening behavior. It processes raw data from multiple text files containing music listening records and provides insightful visualizations of user habits, including favorite artists, albums, and listening patterns over time. 

## 2. What Does It Do?

The project performs the following key functions:

### Data Ingestion: 
Given the context of the project, the data was available on Teams, so we downloaded the files locally on our computers.
### Data Processing (Databricks): 
#### Data extraction
The raw data within the RDD was processed to extract the listener names from the file paths. Additionally, the content was split into individual records, each representing a unique music listening event. Each record was parsed into five distinct fields: listener name, artist/band, album, track, and date.

	from pyspark.sql import SparkSession
	from pyspark.sql import Row
	rdd_transformed = rdd.map(lambda e: (e[0].split("/")[-1].split(".")[0], e[1])) \
	                     .flatMapValues(lambda v: v.split("\n")) \
	                     .map(lambda e: (e[0], e[1].split(","))) \
	                     .map(lambda e: (
	                         e[0],                              # Listener name
	                         e[1][0],                           # Artist name
	                         e[1][1] if len(e[1]) > 1 else None,  # Album name (if it exists)
	                         e[1][2] if len(e[1]) > 2 else None,  # Track name (if it exists)
	                         e[1][3] if len(e[1]) > 3 else None   # Date (if it exists)
	                     ))
	
	columns = ["listener", "artist / band", "album", "track", "date"]
	# create a structured PySpark DataFrame with our specified schema for processing
	df = spark.createDataFrame(rdd_transformed, schema=columns) 
	display(df)
#### splitting date and time for granular analysis
The original "date" column was split into two separate columns: listened_date and hour.

	df_formatted = df.withColumn(
	    "listened_date",
	    F.date_format(F.to_date(F.col("date"), "dd MMM yyyy HH:mm"), "dd/MM/yy") 
	).withColumn(
	    "hour",
	    F.date_format(F.to_timestamp(F.col("date"), "dd MMM yyyy HH:mm"), "HH:mm") 
	)
#### handling missing values, removing empty rows and dropping column date.
We then had to make sure that any rows that were redundant or completely empty were removed from the dataset to ensure data integrity and reduce potential noise in the analysis.

	df_formatted = df_formatted.select(
	    [F.when(F.col(c) == '', None).otherwise(F.col(c)).alias(c) for c in df_formatted.columns]
	)
 	df_formatted = df_formatted.drop("date")
  	df_formatted = df_formatted.dropDuplicates()
   	df_formatted = df_formatted.dropna(subset=[col for col in df_formatted.columns if col != 'listener'], how='all')

### Database Integration :
The processed data was stored in a PostgreSQL database hosted on Supabase.

	jdbc_url = "jdbc:postgresql://aws-0-eu-central-1.pooler.supabase.com:6543/postgres"
	properties = {
	    "user": "postgres.dbwrzjfnxqhllunphygs",
	    "password": "PasswordTracks2024.",
	    "driver": "org.postgresql.Driver"
	}
	df_formatted.write.jdbc(url=jdbc_url, table="public.tracks_record", mode="overwrite", properties=properties)
## PowerBI Dashboard
### The star model
In this project, we used a star schema to structure our data model, enhancing query performance and enabling efficient reporting. The star schema consists of a central fact table here called “public tracks_record” surrounded by multiple dimension tables, which simplifies data relationships and improves readability. Each table is smaller and easier to join which makes it easier to filter and query our data.

Therefore, the dimensions table here are:

	Year Dimension: A table that contains a list of all unique years. 
	Week Dimension: A table that contains a list of all unique weeks.  
	Listener Dimension: A table that contains information about listeners. 
	Track Dimension: A table that contains information about the tracks being listened to.
	Album Dimension: A table that contains information about the albums.
	Artist Dimension: A table that contains information about the artists.

### The requested KPIs
Here’s the list of the requested KPIs for this project to which we added three others we though were relevant:

	Most listened track of all time
	Most listened track for each week
	Most listened album of all time
	Most listened album for each week
	Cross tabulation of the number of listened tracks by listener and by artist
	Ranking the 10 biggest listeners of all time
	Ranking the 10 biggest listeners for each week

### Additional KPIs
For additional KPIs, we decided on the following:

	Top 10 most listened artist
 	Top 10 most listened artist per year
 	Top 10 most active hours
  	
