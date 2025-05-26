**Task:**

**1. Data Integration Architecture**

- Source: On-Premises MS SQL Server
- Destination: Azure SQL Database
- Datalake: Azure Blob Storage
- SharePoint: Represented by a folder in Azure Blob Storage (used instead of a SharePoint site)


**2. Hydration File Processing**

**2.1 Preparation**
- Create the Hydration file with Name **HydrationFile.csv** with following columns [sno,sourceSchema,sourceTableName,LoadType,Status].

- Update the Hydration file as needed [This will be hourl/daily/monthly/weekly].
 
| S.No | Source Schema      | Source Table Name | Load Type | Status |
|------|--------------------|-------------------|-----------|--------|
| 1    | ADF                | Customers         | full      | 1      |
| 2    | ADF                | Inventory         | full      | 1      |
| 3    | ADF                | Orderitems        | full      | 1      |
| 4    | ADF                | Orders            | inc       | 1      |
| 5    | ADF                | Payments          | inc       | 1      |
| 6    | ADF                | Products          | full      | 1      |
| 7    | ADF                | Promotions        | inc       | 1      |
| 8    | ADF                | Returns           | inc       | 1      |
| 9    | ADF                | Reviews           | inc       | 1      |
| 10   | ADF                | ShippingDetails   | inc       | 1      |

- Create a folder named SharePoint on Azure Blob Storage (Note: The Hydration file will be stored on Azure Blob Storage instead of SharePoint).

- Upload the Hydration file to the SharePoint folder on Azure Blob Storage.

- Create a folder structure named **HydrationFileFolder** on Azure Blob Storage to store daily files.
Format: YYYY/MM/DD (e.g., 2025/05/23)

**3 Move Hydration file and Load data into Table**
- Develop an ADF pipeline to move the Hydration file from the SharePoint folder (on Azure Blob Storage) to the **DataLake** [HydrationFileFolder (also on Azure Blob Storage)].

- Develop an ADF pipeline to read the Hydration file (CSV format) and load the data into an Azure SQL Database table.
- Before transferring data from the CSV Hydration file to the SQL table, truncate the previous run's data from the SQL table

![image](https://github.com/user-attachments/assets/4ffed6ba-ffb2-4909-865e-be5c44a28ea3)



**4. Full Load Process**

- Develop an ADF pipeline to perform a full load from the on-premises SQL tables to the bronze layer in Azure Blob Storage, storing the data as a file named in the format: schemaname_tablename.csv (e.g., .csv or .parquet).
  - The pipeline should read table names from the Azure SQL table named "tablelist" and filter records where LoadType = 'full'.
    
  - Use a ForEach activity to iterate over each table entry.
 
  - Inside the loop, use a Copy Data activity to extract data from each source table and save it as a .csv file.
 
  - Files should be stored in the Bronze Layer folder using the following structure:YYYY/MM/DD/schemaname_tablename.csv
  - ![image](https://github.com/user-attachments/assets/b8faed1f-eb06-4168-a51a-37a9212408c9)

  - ![image](https://github.com/user-attachments/assets/4e5934bb-6593-4b90-b5ee-bb7d55fec2d5)

**5. Integration**
 - Create a master pipeline that orchestrates the following processes:
    - Move the Hydration file from the SharePoint folder (Azure Blob Storage) to the Data Lake. **(Refer above Point 3.)**
    - Load data from the Hydration file into the Azure SQL table. **(Refer above Point 3.)**
    - Execute the Full Load process from the on-premises SQL source to the Bronze Layer in Azure Blob Storag **(Refer above Point 4.)**
      ![image](https://github.com/user-attachments/assets/e85cbbe9-35f8-4f8f-869b-c49cca7b33ad)

      
**6. Incriment Load Process**

- Create Watermarktable to storedata modification information
  - The table should have the following columns:
    - Id INT,
    - TableName VARCHAR(30)
    - WatermarkDateTime DATETIME

  - Run a full data load first, then initialize the WatermarkTable with the current data timestamps before starting incremental loads.

    - insert into ADF.WatermarkTable values(2,'Inventory',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(3,'Orders',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(4,'OrderItems',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(5,'Payments',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(6,'Products',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(7,'Promotions',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(8,'Returns',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(9,'Reviews',dateadd(day,-60,getdate()))
    - insert into ADF.WatermarkTable values(10,'ShippingDetails',dateadd(day,-60,getdate()))

- Develop an ADF pipeline for a Incriment Load from the following folder structure:
data/YYYY/MM/DD/HydrationFile.csv

  - The pipeline should read the Hydration CSV file and load the data into the target Azure SQL table where LoadType = 'inc'
    
**7. Perform cleaning**
    
- Perform cleaning for each table then transform the data then load into **DeltaLake** (merge - DWH -SCD)
