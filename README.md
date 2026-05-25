# Social-Media-Data-Integration-and-Analysis-using-AWS-Glue--ETL
## 🏗️ Architecture Overview
S3 (Raw Data) → AWS Glue Crawler → Glue Data Catalog → Glue ETL Job → S3 (Output)

---

## ☁️ AWS Services Used

- **Amazon S3** — Raw data storage and processed output
- **AWS Glue** — Data Catalog, Crawlers, Classifiers, and Visual ETL
- **IAM** — Role-based access control for Glue
- **Amazon S3 Select** — SQL querying on output Parquet files

---

## 🪣 S3 Bucket Structure
etl-cep-279/                  ← Input bucket
├── product-files/            ← Product data (CSV)
└── transaction-files/        ← Transaction data (CSV)
etl-cep-output-279/           ← Output bucket
└── [Parquet output files]

---

## 📋 Project Steps

### 1. S3 Setup
- Created input bucket `etl-cep-279` (US East, N. Virginia)
- Created two subfolders: `product-files/` and `transaction-files/`
- Created output bucket `etl-cep-output-279`

### 2. AWS Glue Data Catalog
- Created database `abc-retail` in AWS Glue

### 3. Classifiers
Set up two CSV classifiers to correctly parse the data files:
- `cust-classifier` — for product data
- `txnclass` — for transaction data

### 4. IAM Role
- Created `glue-role` with permissions for AWS Glue to access S3 and the Data Catalog

### 5. Crawler
- Created crawler `retail_crawl` pointing to `s3://etl-cep-279/transaction-files/`
- Assigned `txnclass` classifier and `glue-role`
- Target database: `abc-retail`
- Crawler ran successfully in ~40 seconds, creating 1 table

### 6. ETL Job (`etl-cep-job1`)
Built a Visual ETL pipeline in AWS Glue Studio with the following nodes:

| Step | Transform | Description |
|---|---|---|
| Source 1 | AWS Glue Data Catalog | Reads `txntransaction_files` table |
| Source 2 | AWS Glue Data Catalog | Reads `product_files` table |
| Transform 1 | Join | Inner join on `productid = product id` |
| Transform 2 | Drop Fields | Removes duplicate/unwanted columns |
| Transform 3 | Regex Extractor | Extracts numeric sales value from `sales` column into `Netsales` using regex `\d+` |
| Transform 4 | Aggregate | Groups by `product_category` and `ship_mode`, computes average `sales` |
| Target | Amazon S3 | Writes output as Parquet to `etl-cep-output-279` |

- Job type: Spark (Glue 5.0, G.1X, 10 DPUs)
- Job ran successfully in **1 minute 43 seconds**

### 7. Output Verification
- Queried output Parquet file using S3 Select (SQL)
- Result: `Standard Class, Auto & Accessories` — confirmed correct aggregation

---

## 🛠️ Tech Stack

| Service | Purpose |
|---|---|
| Amazon S3 | Data lake storage |
| AWS Glue Crawlers | Schema discovery |
| AWS Glue Data Catalog | Centralized metadata |
| AWS Glue Visual ETL | No-code data transformation |
| IAM | Permissions and security |
| S3 Select | SQL on Parquet output |

---

## 📌 Key Concepts Demonstrated

- Cloud data lake architecture on AWS
- ETL pipeline design with join, filter, and aggregation
- Schema inference using Glue Crawlers and custom Classifiers
- IAM role configuration for Glue
- Parquet output format for efficient querying
- SQL querying on S3 using S3 Select
