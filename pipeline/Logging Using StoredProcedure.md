🔍 Logging and Exception Handling

To ensure visibility, traceability, and resilience in the data pipeline, I implemented a custom logging and exception handling framework using Azure Data Factory and SQL Stored Procedures.

✅ Logging Implementation

Created a centralized logging table: pipeline_logs in the cloud SQL database

Table Schema :

CREATE TABLE pipeline_logs (

    pipeline_name VARCHAR(255),
    
    run_id VARCHAR(255),
    
    start_time DATETIME,
    
    end_time DATETIME,
    
    status VARCHAR(50),
    
    error_message VARCHAR(MAX)
    
);


Developed a stored procedure: sp_log_pipeline_run to insert execution details of each pipeline run.

CREATE PROCEDURE sp_log_pipeline_run

    @pipeline_name VARCHAR(255),
    
    @run_id VARCHAR(255),
    
    @start_time DATETIME,

    @end_time DATETIME,
    
    @status VARCHAR(50),
    
    @error_message VARCHAR(MAX)
    
AS

BEGIN

    INSERT INTO pipeline_logs (pipeline_name, run_id, start_time, end_time, status, error_message)
    
    VALUES (@pipeline_name, @run_id, @start_time, @end_time, @status, @error_message);
    
END


ADF 

![image](https://github.com/user-attachments/assets/adae8723-18be-4c96-bf0e-c3b232128593)


Stored Procedure activities:

To get Error message :

     @concat(activity('old_watermark_value')?.Error?.Message ,'|', activity('new_watermark_value')?.Error?.Message ,'|',activity('Copy Onprem to Cloud')?.Error?.Message ,'|',activity('Stored procedure')?.Error?.Message)

![image](https://github.com/user-attachments/assets/7ccd358e-3d1e-4446-9df1-eaa6c8ab1532)

Table results

![image](https://github.com/user-attachments/assets/8beecc47-3b20-42b4-aeeb-48f71b6b9147)

Ensured graceful failure handling without interrupting the entire workflow.

