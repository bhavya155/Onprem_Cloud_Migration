On-Premises to Cloud Data Migration

Project Overview: Migrated enterprise data from an on-premises SQL Server  to Azure SQL Server using Azure Data Factory.

Detalied overview :    [pipeline/Analysis.md](https://github.com/bhavya155/Onprem_Cloud_Migration/blob/7c5a0adbaed2ab7e70e5958617937cb58bf9cfad/pipeline/Analysis.md)

🚀 Objective
To modernize the data platform and enable cloud-based analytics by securely and efficiently transferring large volumes of structured from on-prem to Azure.

🖼️ Architecture
On-Prem to Azure SQL Architecture
![onprem_to_cloud_architecture](https://github.com/user-attachments/assets/a27a4c46-bae9-4fe9-b7be-dc880fc8e04d)

🛠 Tools & Technologies

Azure Data Factory (ADF)

Self-hosted Integration Runtime (SHIR)

Azure Data Lake Storage Gen2 (ADLS)

Azure SQL Database

🔧 Key Implementation Steps

Installed and configured SHIR to securely access on-prem SQL and file systems.

Created parameterized pipelines in ADF to ingest data in parallel batches.

Handled schema drift using Mapping Data Flows and dynamic columns.

Automated pipeline triggers using Tumbling Window & Event-based triggers.

💡 Features

Supports both full and incremental loads

Dynamic linked services and datasets for reusability

Error handling and retry logic built into pipelines

Monitoring dashboards via Azure Monitor and Log Analytics

🏆 Outcome

Reduced manual effort by 90%, improved data availability for analytics, and established a scalable, automated cloud pipeline for future use cases.
