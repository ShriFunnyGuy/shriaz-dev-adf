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
| 1    | HydrationFile.csv  | Customers         | full      | 1      |
| 2    | HydrationFile.csv  | Inventory         | full      | 1      |
| 3    | HydrationFile.csv  | Orderitems        | full      | 1      |
| 4    | HydrationFile.csv  | Orders            | inc       | 1      |
| 5    | HydrationFile.csv  | Payments          | inc       | 1      |
| 6    | HydrationFile.csv  | Products          | full      | 1      |
| 7    | HydrationFile.csv  | Promotions        | inc       | 1      |
| 8    | HydrationFile.csv  | Returns           | inc       | 1      |
| 9    | HydrationFile.csv  | Reviews           | inc       | 1      |
| 10   | HydrationFile.csv  | ShippingDetails   | inc       | 1      |

- Create a folder named SharePoint on Azure Blob Storage (Note: The Hydration file will be stored on Azure Blob Storage instead of SharePoint).

- Upload the Hydration file to the SharePoint folder on Azure Blob Storage.

- Create a folder structure named **HydrationFileFolder** on Azure Blob Storage to store daily files.
Format: YYYY/MM/DD (e.g., 2025/05/23)

**2.2 Azure Data Factory (ADF) Pipelines**
- Develop an ADF pipeline to move the Hydration file from the SharePoint folder (on Azure Blob Storage) to the **DataLake** [HydrationFileFolder (also on Azure Blob Storage)].

- Develop an ADF pipeline to read the Hydration file (CSV format) and load the data into an Azure SQL Database table.
- Before transferring data from the CSV Hydration file to the SQL table, truncate the previous run's data from the SQL table

![image](https://github.com/user-attachments/assets/4ffed6ba-ffb2-4909-865e-be5c44a28ea3)



**3. Full Load Process**

- Develop an ADF pipeline for a Full Load from the following folder structure:
data/YYYY/MM/DD/HydrationFile.csv

  - The pipeline should read the Hydration CSV file and load the data into the target Azure SQL table where LoadType = 'full'.

**4. Incriment Load Process**

- Develop an ADF pipeline for a Incriment Load from the following folder structure:
data/YYYY/MM/DD/HydrationFile.csv

  - The pipeline should read the Hydration CSV file and load the data into the target Azure SQL table where LoadType = 'inc'
    
**5. Perform cleaning**
    
- Perform cleaning for each table then transform the data then load into **DeltaLake** (merge - DWH -SCD)
