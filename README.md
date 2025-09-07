# 📊 IBM HR Attrition Analytics Project  (Azure + Python + Power BI)

📊 **End-to-End Data Project** showcasing Data Engineering + Analytics.  
Data flows from **Azure Blob → ADF Pipelines → Azure SQL → Power BI** for reporting.  

## 🚀 Project Overview  
This project analyzes IBM’s HR dataset to uncover key insights about employee attrition.  
The workflow integrates **Python (data cleaning)**, **Azure (data pipeline & storage)**, **SQL Database**, and **Power BI (visual analytics)**.  

🔑 **End Goal**: Deliver an automated analytics pipeline that keeps HR dashboards always up to date.  

---
## 🏗️ Architecture
![Architect](docs/Dashboard1.jpg)  

## 🖼 Dashboard Preview  

### Executive Summary Dashboard  
![Dashboard 1](docs/Dashboard1.jpg)  

### Attrition Deep Dive Dashboard  
![Dashboard 2](docs/Dashboard2.jpg)  

*(Interactive version available if hosted on Power BI Service — link can be added here)*  

## 🔑 Key Insights
- **Attrition Rate:** ~16% of employees have left.  
- **Job Satisfaction:** Higher attrition in *Low* and *Medium* satisfaction groups.  
- **Distance from Home:** Attrition higher for employees with longer commutes (>20 km).  
- **Environment & Work-Life Balance:** Poor ratings correlate strongly with attrition.  
- **Demographics:** Unmarried employees and those with fewer years at company showed higher attrition.  

---

## 🚀 Key Learnings

Designed an end-to-end automated HR pipeline in Azure

Demonstrated secure credential handling with .env

Applied Python-based cleaning and logging for reproducibility

Automated ETL with ADF triggers

Showcased DirectQuery integration into Power BI, proving scalability
---

## 🗂 Project Workflow  

**Raw → Cleaned → SQL DB → Power BI**  

## 📌 Project Workflow

---

1. **Data Ingestion**  
   - Raw HR Attrition CSV uploaded to **Azure Blob Storage**.

## 🐍 Python Cleaning

Python was used to pull raw data directly from **Azure Blob Storage**, clean it, and then push the cleaned dataset back into the **clean container**.  

### Steps:
 **Read raw CSV from Blob, connect and download** 
![Read raw CSV from Blob, connect](docs/read1.PNG)
![Download in colab](docs/Download2.PNG)

3. **Data Cleaning & Transformation**  
   - Python scripts clean and preprocess data (missing values, encoding categorical fields, etc.).
```
import pandas as pd
import numpy as np
from datetime import datetime
import os

# Ensure logs folder exists (safe for every run)
os.makedirs("logs", exist_ok=True)

# Load raw data
df = pd.read_csv("IBM.csv")


# Logging info
log = {
    "file_name": "IBM.csv",
    "rows": len(df),
    "nulls": df.isnull().sum().to_dict(),
    "duplicates": df.duplicated().sum(),
    "outliers": {}
}

# Handle duplicates
df = df.drop_duplicates()

# Handle missing values
df = df.fillna({
    "MonthlyIncome": df["MonthlyIncome"].median(),
    "WorkLifeBalance": df["WorkLifeBalance"].mode()[0]
})

# Outlier detection for Age only (business rule: valid ages 18–62)
outliers_age = df[(df["Age"] < 18) | (df["Age"] > 62)]
log["outliers"]["Age"] = len(outliers_age)
df["Age"] = np.clip(df["Age"], 18, 62)  # winsorize Age only

# No winsorization for MonthlyIncome
log["outliers"]["MonthlyIncome"] = "Skipped (kept original values)"

# ✅ Ensure EmployeeNumber exists
if "EmployeeNumber" not in df.columns:
    # Generate new IDs if missing
    df.insert(0, "EmployeeNumber", range(1, len(df) + 1))

# Remapping the values for dashboard
for col, mapping in mappings.items():
  if col in df.columns:
    df[col] = df[col].replace(mapping)

# Save cleaned data
file_name = f"IBM_cleaned_{datetime.today().strftime('%Y%m%d')}.csv"
df.to_csv(file_name, index=False)

# Save log
log_df = pd.DataFrame([log])
log_df.to_csv("logs/validation_log.csv", mode="a", header=False, index=False)
```

Key steps performed:  
- Removed duplicates  
- Filled missing values (`MonthlyIncome`, `WorkLifeBalance`)  
- Winsorized `Age` (18–62 years)  
- Preserved true salary distribution (no winsorization for `MonthlyIncome`)  
- Mapped coded categorical fields (Education, JobSatisfaction, etc.)  

```python
# Example snippet
mappings = {
    "JobSatisfaction": {1: "Low", 2: "Medium", 3: "High", 4: "Very High"},
    "WorkLifeBalance": {1: "Bad", 2: "Good", 3: "Better", 4: "Best"}
```
   - Cleaned data stored back in Blob for ADF ingestion.
![Connecting to BLOB](docs/connect1.PNG)
![Dashboard 2](docs/download.PNG)


### Azure Blob Storage (Raw → Clean containers)  
![Blob Storage](docs/Blob_raw.PNG) 

4. **Data Orchestration (Azure Data Factory)**  
   - ADF pipeline copies cleaned data from Blob → Azure SQL Database.
   - Triggers ensure automated refresh.
![ADF_Process](docs/ADF1.PNG)
![Mappinig](docs/ADF2.PNG)
![Trigger](docs/Trigger.PNG)

###🔄 Azure Data Factory (ADF) Pipeline

![ADF_PIPELINE](docs/ADF_Pipeline.PNG)

Source: Clean container (IBM_cleaned_*.csv)

Sink: Azure SQL Database (Employee_Clean table)

Schema Mapping: Adjusted to handle text fields (e.g., Education → VARCHAR)

Trigger: Automated run on new file arrival / scheduled refresh

5. **Data Visualization (Power BI)**  
    ***🔗 Data Integration with Power BI
   
![Data_Loading](docs/Data_Load.JPG)

   -Power BI was connected to Azure SQL Database via DirectQuery.

Key point:

No local CSVs or static extracts used

Data flows directly from SQL DB → BI layer

Any update in pipeline automatically flows into reports

---





}
df["JobSatisfaction"] = df["JobSatisfaction"].map(mappings["JobSatisfaction"])
