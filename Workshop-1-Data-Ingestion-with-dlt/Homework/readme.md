# NYC Taxi Data Pipeline using dlt and DuckDB

This project demonstrates how to extract, transform, and load (ETL) NYC Taxi data from a REST API into DuckDB using the `dlt` library. The dataset is retrieved from the following API:

🔹 **API URL:** [NYC Taxi API](https://us-central1-dlthub-analytics.cloudfunctions.net/data_engineering_zoomcamp_api)
🔹 **Data format:** Paginated JSON (1,000 records per page)
🔹 **Pagination Rule:** Stops when an empty page is returned

##  Project Overview
This notebook:
1. Installs and configures `dlt`.
2. Defines a data pipeline to extract NYC Taxi data.
3. Loads the extracted data into DuckDB.
4. Performs data exploration and analysis.

Question 1: Dlt version
Before running the notebook, install the required dependencies:
```sh
!pip install dlt[duckdb]
```

Check the `dlt` version:
```python
import dlt
print("dlt version:", dlt.__version__)
```

Question 2: Define & Run the Pipeline
The pipeline is implemented using `dlt.resource` for data extraction and `RESTClient` for pagination.

### Define API Source & Pagination
```import dlt
from dlt.sources.helpers.rest_client import RESTClient
from dlt.sources.helpers.rest_client.paginators import PageNumberPaginator

#Use the @dlt.resource decorator to define the API source
@dlt.resource(name="rides",write_disposition="replace")   # <--- The name of the resource (will be used as the table name)
def ny_taxi():
    client = RESTClient(
        base_url="https://us-central1-dlthub-analytics.cloudfunctions.net/data_engineering_zoomcamp_api",
        paginator=PageNumberPaginator(
            base_page=1,
            total_path=None
        )
    )

    for page in client.paginate("data_engineering_zoomcamp_api"):    # <--- API endpoint for retrieving taxi ride data
        yield page


pipeline = dlt.pipeline(
    pipeline_name="ny_taxi_pipeline",
    destination="duckdb",
    dataset_name="ny_taxi_data"
)


#Load the data into DuckDB 
load_info = pipeline.run(ny_taxi)
print(load_info)
```

##  Explore the Data
### Connect to DuckDB and Inspect Tables
```#Start a connection to your database using native duckdb connection and look what tables were generated
import duckdb
from google.colab import data_table
data_table.enable_dataframe_formatter()

# A database '<pipeline_name>.duckdb' was created in working directory so just connect to it

# Connect to the DuckDB database
conn = duckdb.connect(f"{pipeline.pipeline_name}.duckdb")

# Set search path to the dataset
conn.sql(f"SET search_path = '{pipeline.dataset_name}'")

# Describe the dataset
conn.sql("DESCRIBE").df()
```

### Question 3: Explore the loaded data
```#Inspect the table rides
df = pipeline.dataset(dataset_type="default").rides.df()
df
```
```
df = pipeline.dataset(dataset_type="default").rides.df()

# Print the number of records
print("Total records extracted:", len(df))
```
### Question 4: Trip Duration Analysis
```python
with pipeline.sql_client() as client:
    res = client.execute_sql(
            """
            SELECT
            AVG(date_diff('minute', trip_pickup_date_time, trip_dropoff_date_time))
            FROM rides;
            """
        )
    # Prints column values of the first row
    print(res)
```

## 📌 Expected Outputs
- **Number of tables created:** 4
- **Total records extracted:** 10,000 
- **Average trip duration:** ~12.3049 minutes

## 📜 Notes
- If duplicate records appear, ensure `write_disposition="replace"` is set.
- If data size varies, check API pagination behavior.





