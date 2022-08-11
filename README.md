# Apache Airflow and DBT using Docker Compose to a Weather DataPipeline
Stand-alone project that utilises openweather API to load histocal data, load them in filesystem, process them and save this data in Postgres Database. After that, this DBT is used to model this data and presents some insights. Everything is orchestrated through Airflow.


## Requirements 
* Install [Docker](https://www.docker.com/products/docker-desktop)
* Install [Docker Compose](https://docs.docker.com/compose/install/)

## Setup 
* Clone this repository

Change directory within the repository and run 
docker build . -f Dockerfile --pull --tag metadataio:0.0.1
After that run `docker-compose up`. This will perform the following:
* Based on the definition of [`docker-compose.yml`] will download the necessary images to run the project. This includes the following services:
  * postgres-airflow: DB for Airflow to connect and store task execution information
  * postgres-dbt: DB for the DBT seed data and SQL models to be stored
  * airflow: Python-based image to execute Airflow scheduler and webserver

## Connections
* Airflow UI: http://localhost:8000

## How to ran the DAGs
Once everything is up and running, navigate to the Airflow UI (see connections above). You will be presented with the list of DAGs, all Off by default.
The dag used to run this example is called "weather"

This DAG will run the steps in the following order: 
- 1_clean_raw_data: Delete all the content in the raw directory (This directory get the data from API in json format whitou any modification)
- 2_clean_staging_data: Delete all the content in the staging directory (This directory the data allready deduplicated and only the necessary columns)
- 3_getapi: Get data from API to the days needed
- 4_postegres_tasks: Create the tables if they dont exist
- 5_truncate_postgres_tasks: Truncate the tables before new load to preserve indepondence
- 6_insert_postgres_tasks: Insert data from staging_data through COPY command into Postgres Database
- 7_dbt_tests: Run the tests defined in dbt. In this case the only test created it was to check if temp is not null 
- 8_dbt_model: Run the models to create dataset1 and dataset2 as materialized views in postgres database.


If everything goes well, you should have the dataset1 and dataset2 models execute successfully and see similar task durations as per below.

## Docker Compose Commands
* Enable the services: `docker-compose up` or `docker-compose up -d` (detatches the terminal from the services' log)
* Disable the services: `docker-compose down` Non-destructive operation.
* Delete the services: `docker-compose rm` Ddeletes all associated data. The database will be empty on next run.
* Re-build the services: `docker-compose build` Re-builds the containers based on the docker-compose.yml definition. Since only the Airflow service is based on local files, this is the only image that is re-build (useful if you apply changes on the `./scripts_airflow/init.sh` file. 

If you need to connect to the running containers, use `docker-compose ps` to view the running services.

For example, to connect to the Airflow service, you can execute `docker exec -it [dockercontainerid] bash`. This will attach your terminal to the selected container and activate a bash terminal.
In this case, you can connect to the postgres container and run 'psql -U airflow'; After that, you can run both 'select * from dataset1' and 'select * from dataset2'


![airflow_lineage](https://user-images.githubusercontent.com/25934101/184164570-7bddb413-a3c5-4e49-96dd-9ca1bb9b60ff.PNG)
