# Design-Medallion-Architecture-Azure
Azure Data Factory, Azure Databricks, DBT

Medallion Architetcure Layers
1. Bronze Layer - Raw Data
2. Silver - Fine Tuned Data
3. Gold - Refined Data



# Medallion Architecture

![architecture](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742221900/diagram-export-17-03-2025-10_31_09_elljhp.png)

Demo Snapshots

![1](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742224140/Screenshot_2025-03-17_at_11.06.26_AM_roukan.png)

![2](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742224141/Screenshot_2025-03-17_at_11.07.26_AM_jzifio.png)

![3](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742224140/Screenshot_2025-03-17_at_11.07.03_AM_d3cqct.png)

![4](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742224142/Screenshot_2025-03-17_at_11.06.08_AM_vcj06c.png)

![6](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742225243/Screenshot_2025-03-17_at_11.27.06_AM_jwerr7.png)

![5](https://res.cloudinary.com/vaibhav-codexpress/image/upload/v1742224141/Screenshot_2025-03-17_at_11.08.10_AM_h0zob3.png)

# Demo

[![demo](http://img.youtube.com/vi/VKSf1NfdjTU/0.jpg)]("https://www.youtube.com/watch?v=VKSf1NfdjTU")

# Workflow Instructions

1. Create a resource group (You can download template for automation) 
2. Create Storage Account - Attach Resource Group -> Enable Public Access
3. Add Container to Storage -> Data Storage
    1. Bronze
    2. Silver
    3. Gold
4. Create Azure Data Factory (ADF) -> Attach Resource group, Public endpoint
5. Key vaults -> Storing all keys and secrets -> Create Key vault pair, attach resource group, permissions - vault access policy,allow access from all networks
6. Go to Data Factory -> Launch Studio
7. Go to SQL Databases -> Create Database -> ATttach resource group,public end point
8. Create SQL server 

Goto Query Editor for querying your data

```
select * from [medallion-vb-db].information_schema.tables where table_schema = 'SalesLT' and table_type='BASE TABLE'
```

GOTO Data Factory

Manage -> Linked Service -> New -> Azure -> Azure SQL Database, select subscription

Create Linked Service (It defines connection infromation to a data store or compute (link access from data factory to azure))

Create Linked service fro dumping data to Azure Data Lake Storage Gen 2

Goto Author Section -> Create Pipeline

Create Dataset -> Azure SQL Database

Pipeline -> General -> lookup (fetching data for us) -> provide query in settings

foreach

copy data (configure source)

create new data set -> Azure sql Database

Sql Table -> Schema,& parameters

Sink -> (where we are sending our dataset)

New dataset -> azuew data lake storage -> parquet 

Addding parameters

Populating data in azure databricks workspace

# Create Azure Databricks workspace

1. Create Databricks Workspace
2. Go to resource
3. Launch Workspace
4. Add New Folder to Workspace
5. Add Notebook to workspace fro transfromation
6. Create Secret Scope  (to connect azure databricks to azure data factory - 
7. [#secrets/createScope](https://adb-494607385849073.13.azuredatabricks.net/?o=494607385849073#secrets/createScope))
8. Go to Key vaults -> Objects -> Go to secrets -> Create Secrets
    a. Get secret text from storage accounts -> (Security + Networking) -> Access keys and add it to key vault secret variables
    b. Add user to access policies
    c. Go to properties and copy vault uri
    d. uri will be the dns name in secrete scope in databricks as well as the resource id and also provide scope name
    e. Manage principle -> all users

# Base Databricks Notebook

Check storage account connection with databricks and configure the cluster

Establish Connection between Databricks and Datafactory

add databricks notebook to pipeline in ADF and configure a linked service for it

Get access token from user settings in databricks for connecting ADF -> Developer

Check Connection in Notebook

```
dbutils.fs.mount(
    source='wasbs://bronze@medallionvbst.blob.core.windows.net',
    mount_point='/mnt/bronze',
    extra_configs={'fs.azure.account.key.medallionvbst.blob.core.windows.net': dbutils.secrets.get(scope='databricksScope', key='storageAccountKey')}
)

dbutils.fs.mount(
    source='wasbs://silver@medallionvbst.blob.core.windows.net',
    mount_point='/mnt/silver',
    extra_configs={'fs.azure.account.key.medallionvbst.blob.core.windows.net': dbutils.secrets.get(scope='databricksScope', key='storageAccountKey')}
)

dbutils.fs.mount(
    source='wasbs://gold@medallionvbst.blob.core.windows.net',
    mount_point='/mnt/gold',
    extra_configs={'fs.azure.account.key.medallionvbst.blob.core.windows.net': dbutils.secrets.get(scope='databricksScope', key='storageAccountKey')}
)

dbutils.fs.ls('/mnt/bronze')
```

```
spark.sql('create database testing')
```

set up databricks notebook in pipeline - 3 param, linked service and browsing the notebook

```
fileName = dbutils.widgets.get('fileName')
tableSchema = dbutils.widgets.get('table_schema')
tableName = dbutils.widgets.get('table_name')

#create databse if not exists
spark.sql(f'CREATE DATABASE IF NOT EXISTS {tableSchema}')

spark.sql("""CREATE TABLE IF NOT EXISTS """ + tableSchema + "." + tableName + """
          USING PARQUET
          LOCATION '/mnt/bronze/"""+ fileName + """/"""+tableSchema+"""."""+tableName+""".parquet'
          """)


```

# Set up DBT

```
databricks configure --token
databricks secrets list-scopes
databricks fs ls
dbt init
```

http path -> compute -> advanced -> odbc
not use unity catalog

13:22:44  Profile medallion_vb written to /Users/vaibhavbansal/.dbt/profiles.yml using target's profile_template.yml and your supplied values. Run 'dbt debug' to validate the connection.

```
dbt snapshot
dbt test
dbt run
dbt docs generate
dbt docs serve
```

