# Apache-Airflow-ETL-Data-Pipelines
Build a Streaming ETL Pipeline using Kafka

# SCENARIO
Project that aims to de-congest the national highways by analyzing the road traffic data from different toll plazas. Each highway is operated by a different toll operator with a different IT setup that uses different file formats. Your job is to collect data available in different formats and consolidate it into a single file.

# OBJECTIVES
In this task you will author an Apache Airflow DAG that will:
- Extract data from a csv file
- Extract data from a tsv file
- Extract data from a fixed width file
- Transform the data
- Load the transformed data into the staging area

# PROCESS
### 1. Create a directory structure for staging area as follows:

    sudo mkdir -p /home/project/airflow/dags/finalassignment/staging
    
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/5baa5ae1-0e12-4786-aca9-b2421d385fd6)

### 2. The compressed file contains data of toll stations of different companies.

    tolldata.tgz (tollplaza-data.tsv, vehicle-data.csv, payment-data.txt, fileformats.txt)
    
- File "tollplaza-data.tsv"

![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/bd32978c-9e29-4505-a223-152babc1661c)

- File "vehicle-data.csv"
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/b0f618a7-c0bc-4ab0-8bdd-525aac72c42d)

- File "payment-data.txt"
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/821ff660-f01e-4f93-b54b-e1b0f741e9fc)

### 3. Create a DAG

* Library import
```
    from datetime import timedelta
    from airflow import DAG
    from airflow.operators.bash_operator import BashOperator 
    from airflow.utils.dates import days_ago
``` 
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/54649443-4eb4-408a-80ee-fa178097b0ca)

* Define DAG arguments
  
    default_args = { 
    'owner': 'Hoang Dung', 
    'start_date': days_ago(0), 
    'email': ['hd@mail.com'], 
    'email_on_failure': True, 
    'email_on_retry': True, 
    'retries': 1, 
    'retry_delay': timedelta(minutes=5), 
    }
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/3d10fd9b-f179-4ef7-99b8-cde4bd6d3139)

* Define the DAG
  
    dag = DAG( 
    'ETL_toll_data_01', 
    default_args=default_args, 
    description='Apache Airflow Final Assignment', 
    schedule_interval=timedelta(days=1), 
    )
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/87dbab79-1a64-4570-acce-21b0a23c924f)

* Create a task to unzip data
  
- Create a task to unzip data
  
    unzip_data = BashOperator( 
    task_id='unzip_data', 
    bash_command='tar -xvzf /home/linux/airflow/dags/finalassignment/tolldata.tgz -C /home/linux/airflow/dags/finalassignment', 
    dag=dag, 
    )
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/7ad18ebc-e025-45fa-9809-e5ba310987a9)

- Create a task to extract data from csv file (This task should extract the fields Rowid, Timestamp, Anonymized Vehicle number, and Vehicle type from the "vehicle-data.csv" file and save them into a file named "csv_data.csv")
  
    extract_data_from_csv = BashOperator( 
    task_id='extract_data_from_csv', 
    bash_command='cut -f1-4 -d "," /home/linux/airflow/dags/finalassignment/vehicle-data.csv > /home/linux/airflow/dags/finalassignment/staging/csv_data.csv', 
    dag=dag, 
    )
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/981427ce-f7ea-46b3-923b-8a7d3c7f6311)

- Create a task to extract data from tsv file (This task should extract the fields Number of axles, Tollplaza id, and Tollplaza code from the "tollplaza-data.tsv" file and save it into a file named "tsv_data.csv".)
  
    extract_data_from_tsv = BashOperator( 
    task_id='extract_data_from_tsv', 
    bash_command='cut -f5- /home/linux/airflow/dags/finalassignment/tollplaza-data.tsv | tr -d "\r" | tr "[:blank:]" "," > /home/linux/airflow/dags/finalassignment/staging/tsv_data.csv', 
    dag=dag, 
    )
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/94686cec-6cc5-447f-924b-8628f380433f)

- Create a task to extract data from fixed width file (This task should extract the fields Type of Payment code, and Vehicle Code from the fixed width file "payment-data.txt" and save it into a file named "fixed_width_data.csv".)
  
    extract_data_from_fixed_width = BashOperator(
    task_id='extract_data_from_fixed_width',
    bash_command="awk '{print $(NF-1), $NF}' payment-data.txt | tr ' ' ',' > /home/linux/airflow/dags/finalassignment/staging/fixed_width_data.csv",
    dag=dag,
    )
  
![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/dd617bf5-d41a-4fa1-85bd-5d0ebeec21c4)

- Create a task to consolidate data extracted from previous tasks
This task should create a single csv file named extracted_data.csv by combining data from the following files: csv_data.csv, tsv_data.csv, fixed_width_data.csv

    consolidate_data = BashOperator(
    task_id='consolidate_data',
    bash_command='paste -d "," csv_data.csv tsv_data.csv fixed_width_data.csv > extracted_data.csv',
    dag=dag,
    )

- Transform and load the data
  
    transform_data = BashOperator(
    task_id='transform_data',
    bash_command="awk '{ $5 = toupper($5) } 1' extracted_data.csv > transformed_data.csv",
    dag=dag,
    )
  
- Define the task pipeline
  
    unzip_data >> extract_data_from_csv >> extract_data_from_tsv >> extract_data_from_fixed_width >> consolidate_data >> transform_data

  ![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/0fe55520-52f4-4eb7-abe8-eca1ad84e87c)

### 4. Testing and running on the Apache Airflow Web GUI

![image](https://github.com/CodeWorld-X/Apache-Airflow-ETL-Data-Pipelines/assets/129016922/f3ad88be-6509-42f0-8fea-4bf794d9129e)











