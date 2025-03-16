# Design-Medallion-Architecture-Azure
Azure Data Factory, Azure Databricks, DBT

Medallion Architetcure Layers
1. Bronze Layer - Raw Data
2. Silver - Fine Tuned Data
3. Gold - Refined Data



# Medallion Architecture


# Workflow Instructions

1. Create a resource group (You can download template for automation)
2. Create Storage Account - Attach Resource Group -> Enable Publeic Access
3. Add Container to Storage
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
select * from [vb-medallion].information_schema.tables where table_schema = 'SalesT' and table_type='BASIC TABLE'
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




