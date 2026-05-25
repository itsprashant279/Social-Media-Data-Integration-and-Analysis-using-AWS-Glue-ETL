# Social-Media-Data-Integration-and-Analysis-using-AWS-Glue--ETL

An end-to-end cloud data engineering project on AWS that ingests Twitter and blog
social media data from Amazon S3, processes it through an AWS Glue ETL pipeline,
and outputs tweet engagement counts grouped by user for analysis.

---

## 🏗️ Architecture Overview
S3 (Raw Data) → AWS Glue Crawlers → Glue Data Catalog → Glue ETL Job → S3 (Output Parquet)

---

## ☁️ AWS Services Used

- **Amazon S3** — Raw data storage and Parquet output
- **AWS Glue** — Data Catalog, Crawlers, Classifiers, and Visual ETL
- **IAM** — Role-based access control for Glue
- **Amazon S3 Select** — SQL querying on output Parquet files

---

## 🪣 S3 Bucket Structure
etl-twitter-blog1/          ← Input bucket
├── blog-data/              ← Blog post data (CSV)
└── etl-social-media/       ← Twitter/social media data (CSV)
etl-cep-output1/            ← Output bucket
└── [29 Parquet output files]

---

## 📋 Project Steps

### 1. S3 Setup
- Created input bucket `etl-twitter-blog1` (US East, N. Virginia)
- Created two subfolders: `blog-data/` and `etl-social-media/`
- Created output bucket `etl-cep-output1`

### 2. AWS Glue Data Catalog
- Created database `social_media_data` in AWS Glue

### 3. Classifiers
Set up two CSV classifiers for schema detection:
- `twitter_data` — for social media/tweet data
- `blog_data` — for blog post data

### 4. IAM Role
- Reused `glue-role` with full permissions for AWS Glue to access S3 and the Data Catalog

### 5. Crawlers
Created and ran two crawlers, both targeting `social_media_data` database:

| Crawler | Source | Status | Duration |
|---|---|---|---|
| `tweet-crawl` | `etl-social-media/` folder | Completed | 1 min 7s |
| `blog-crawler` | `blog-data/` folder | Completed | 1 min 9s |

Both ran successfully and created tables in the Glue Data Catalog.

### 6. ETL Job (`etl-cep-job`)
Built a Visual ETL pipeline in AWS Glue Studio with the following nodes:

| Step | Transform | Description |
|---|---|---|
| Source 1 | AWS Glue Data Catalog | Reads `etl_social_media` table (tweets) |
| Source 2 | AWS Glue Data Catalog | Reads `blog_data` table |
| Transform 1 | Join | Inner join on `user id = user id` |
| Transform 2 | Drop Fields | Removes duplicate `user id` column from right side |
| Transform 3 | Regex Extractor | Extracts hashtags from `tweet text` column using regex `#(\w+)` into new `hashtags` column |
| Transform 4 | Aggregate | Groups by `tweet id`, counts tweet occurrences |
| Target | Amazon S3 | Writes output as Parquet (Snappy compressed) to `etl-cep-output1` |

- Job type: Spark (Glue 5.0, G.1X, 10 DPUs)
- Job ran successfully in **1 minute 14 seconds**
- Output: **29 Parquet part files** generated

### 7. Output Verification
Queried output Parquet files using S3 Select:

```sql
SELECT * FROM s3object s LIMIT 5
```

Sample results showed tweet counts per user ID:

| User ID | Tweet Count |
|---------|-------------|
| 1032    | 8           |
| 1054    | 1           |
| 1073    | 2           |
| 1010    | 4           |

---

## 🛠️ Tech Stack

| Service | Purpose |
|---|---|
| Amazon S3 | Data lake storage |
| AWS Glue Crawlers | Schema discovery for both data sources |
| AWS Glue Data Catalog | Centralized metadata store |
| AWS Glue Visual ETL | No-code data transformation pipeline |
| IAM | Permissions and security |
| S3 Select | SQL querying on Parquet output |

---

## 📌 Key Concepts Demonstrated

- Multi-source social media data ingestion
- Joining heterogeneous datasets (tweets + blog posts) on a common key
- Hashtag extraction using Regex in Glue ETL
- Engagement aggregation (tweet count per user)
- Parquet output with Snappy compression for efficient storage
- SQL querying on S3 using S3 Select

---

## 🚀 How to Reproduce

1. Upload your Twitter and blog CSV files to the respective S3 folders
2. Run both Glue crawlers to register schemas in the Data Catalog
3. Open the Glue Visual ETL job and run `etl-cep-job`
4. Query output Parquet files in `etl-cep-output1` using S3 Select
