# Tracks-Records -  Vissualization project
Preprocessing of various files containing the listening history of different users.

##How to start and run the project :

### Prerequisites : 
    Have a data bricks account (If not, create an account with your professional email on databricks).
### Step 1 : Create a cluster on Databricks
    On the menu bar, click on “Compute” then “Create cluster” button.
    Choose a name then create.
### Step 2 : Import the data to Databricks
	    -   On the menu bar, click on “Catalog”, then “Create table” button.
	    - 	Name the target directory “data” then upload all the text files at once.
### Step 3 : Open the notebook
    On the menu bar, click “New” -> “Notebook”.
    Once the notebook is created, click on “File” -> “Import” to import the notebook available on Github.
    Now execute the cells to get and process the data then load it to the database.
### Step 4 : At this point everything is done, you can visualize the dashboard on PowerBi.

# Conception and implementation process:
## 1. What Is It?
    This project is designed to analyze and visualize music listening behavior. It processes raw data from multiple text files containing music listening records and provides insightful visualizations of user habits, including favorite artists, albums, and listening patterns over time. 

## 2. What Does It Do?

The project performs the following key functions:

### Data Ingestion: 
Given the context of the project, the data was available on Teams, so we downloaded the files locally on our computers.
### Data Processing: 
    Cleans and transforms the data
    splitting date and time for granular analysis
    handling missing values, and removing empty rows.

### Database Integration:
The processed data was stored in a PostgreSQL database hosted on Supabase.

### Visualization:
