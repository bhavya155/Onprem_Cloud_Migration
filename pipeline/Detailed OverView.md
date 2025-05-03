On-prem to Cloud Data Migration

Steps:

Note: passing parameter to dynamic
note : Choose SHIR in the source settings and Azure IR in the sink (target) settings.
 
1 Created Self-hosted Integration Runtime (SHIR) on-premises
    Installed and configured SHIR on on-prem server.
    This allows ADF to securely access on-prem data sources.
   ![image](https://github.com/user-attachments/assets/d3b78d8b-6bec-4e4b-bdf7-e60aab011e20)

2 Created Azure Integration Runtime (IR)
    Used for accessing cloud-based data sources and services.

3 Created Linked Services
    Defined connections for both on-prem (using SHIR) as ls_sqlserver and cloud (using Azure IR) as ls_cloud_sqlserver data sources.
    ![image](https://github.com/user-attachments/assets/3668b086-4366-4542-a8a6-80e43db2d07e)
    ![image](https://github.com/user-attachments/assets/318a81f2-3986-49c8-b052-a399f365c9c9)
 
4 Create Datasets
    Defined the data structures for source ie. SQL SERVER and destination Azure SQL
    ![image](https://github.com/user-attachments/assets/f9f923fe-3e86-48b9-b150-e8c459d0ede2)
    ![image](https://github.com/user-attachments/assets/4e9f8321-a9c2-4abd-9b11-75be06ca42b2)    

5.Develop Pipelines

  Designed and implemented incremental data load logic using watermark columns:

   ![image](https://github.com/user-attachments/assets/d7cdf712-3ad6-455b-9173-3f9e861c9a22)

   suppose my table struture:

        CREATE TABLE Customers (
        customer_id INT PRIMARY KEY,
        
        first_name NVARCHAR(255),
        
        last_name NVARCHAR(255),
        
        email NVARCHAR(255) UNIQUE,  
        
        created_at DATETIME,
        
        updated_at DATETIME
        
         );

  1.Retrieved the old watermark (last updated_at value) from the cloud using a Lookup activity.
    
     create table watermarktable
     (
     
    TableName varchar(255),
    
    WatermarkValue datetime			-- to track the old timestamp value @ ADF Pipelines
     
     );

  Query: 
  
    particular table :
    
         select WatermarkValue as oldmm from watermarktable where tablename= 'categories'

    data driven :    
           
          select watermarkvalue as oldmm from watermarktable where tablename='@{item().TABLE_NAME}'

  ![image](https://github.com/user-attachments/assets/81ea1291-eca0-466f-bde5-88810cc7e56b)

  2. Retrieved the new watermark (latest updated_at value) from the on-premises source using another Lookup activity

  Query :

      particular table :
      
          select max(update_at) as newmm from categories;
          
       data driven :   

          select max(@{item().WaterMark_Column}) as newmm from @{item().TABLE_NAME}
          
  ![image](https://github.com/user-attachments/assets/77ff4d58-2976-4d04-84c9-df5fe1bbe2dc)


  3. Copy Activity

     Applied filters in Copy Activity: updated_at > old_watermark AND updated_at <= new_watermark to extract only new or updated records.

     Implemented upsert logic into the destination table using a MERGE key (e.g., Customer_ID) to update existing records and insert new ones.

  Query :

       particular table :
              select * from categories where updated_at >'2024-10-14 06:52:45.163' and updated_at<='2025-05-03 06:09:45.567' ;

       Data Driven:

         select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} >'@{activity('old_watermark_value').output.firstRow.oldmm}' and @{item().WaterMark_Column}<= '@{activity('new_watermark_value').output.firstRow.newmm}'

         
            Source:
  ![image](https://github.com/user-attachments/assets/cc898688-e376-4ae7-8fcd-2fd5e4704bf5)

           Sink:
  ![image](https://github.com/user-attachments/assets/d1bbe780-0554-41ef-b8ee-2b9e9c07f5aa)

  4.Stored Procedure
  Table :
  
      create table watermarktable
    (
        TableName varchar(255),
        WatermarkValue datetime			-- to track the old timestamp value @ ADF Pipelines
    );
    
  Procedure :
  
  CREATE PROCEDURE usp_write_watermark ( @LastModifiedtime datetime, @TableName sysname )
   AS
   BEGIN
       UPDATE watermarktable
       SET WatermarkValue = @LastModifiedtime WHERE TableName = @TableName
   END
![image](https://github.com/user-attachments/assets/653945ea-558a-49d7-9fa1-908d540b2186)


Meta Data driven Pipeline

Scaled the solution to handle N number of tables through pipeline parameterization:
![image](https://github.com/user-attachments/assets/b5c4926f-074e-4e96-bb64-3b9ed8a569bd)

  Option 1: Passed an array of table configurations directly into the pipeline.

  Option 2: Used a Lookup activity to dynamically read from a JSON config file with metadata:
           Fing Metadata.json file in repo

  ![image](https://github.com/user-attachments/assets/597aa32a-e0d2-412d-b030-6a96ff14dad7)

  output lookup activity :
     "count": 1,
	   "value": [
      		{
      			"TABLE_NAME": "CATEGORIES",
      			"WaterMark_Column": "updated_at",
      			"MERGE_KEY": [
      				"category_id"
      			]
         }

  foreach activity :
        
        passing the output of lookup activity to foreach
        
              @activity('get metadata').output.value

        ![image](https://github.com/user-attachments/assets/85a0446d-d10d-4784-99d7-0a48ef87b9f4)




