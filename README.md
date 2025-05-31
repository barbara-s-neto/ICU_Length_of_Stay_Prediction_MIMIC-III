# Big Data & Cloud Computing - Project #2



## References ML MIMIC

Alguns artigos para estudar a parte de ML e previsão LOS.

- https://fxis.ai/edu/how-to-use-mimic-iii-benchmarks-for-machine-learning-projects/
- https://colab.research.google.com/github/ponder-org/ponder-notebooks/blob/main/bigquery/example/MIMIC-III%20Health%20Record%20Analysis.ipynb#scrollTo=6a6ce75c-4527-487f-9eea-505f39c96165
- https://theoriginalsam.github.io/publications/13.pdf
- https://github.com/bmmalone/mimic-preprocessing
- https://github.com/YerevaNN/mimic3-benchmarks?tab=readme-ov-file#length-of-stay-prediction



# Descrição para o LOS Forecast

##  ICU Length of Stay Prediction System

## 📊 Data Pipeline Overview

### 1. Data Loading & Setup
- **Spark Configuration**: Optimized settings for adaptive query execution, Arrow integration, and Kryo serialization
- **MIMIC-III Tables**: Loads 6 key medical datasets (CHARTEVENTS, LABEVENTS, ICUSTAYS, PATIENTS, ADMISSIONS, DIAGNOSES_ICD)
- **Target Variable**: ICU Length of Stay in days (continuous regression problem)

### 2. Feature Engineering (5 Categories)

#### 👤 Demographics (2 features)
- Age at ICU admission, Gender (binary)

#### 🏥 Admission Characteristics (8 features)
- Emergency vs elective admission, came from ER, insurance type, ethnicity
- Time-based: weekend/night admissions, hours from hospital to ICU admission

#### 🏢 ICU Unit Types (6 features)  
- First care unit (MICU, SICU, CSRU, CCU, TSICU)
- Whether patient changed ICU units during stay

#### 🫀 Vital Signs (11 features)
- Heart rate, blood pressure, respiratory rate, temperature, oxygen saturation
- Statistics: average, min, max, standard deviation (first 24 hours)

#### 🧪 Laboratory Values (8 features)
- Creatinine, glucose, electrolytes, blood counts, pH
- Uses first available values within 24 hours of ICU admission

#### 🩺 Diagnosis Features (4 features)
- Total number of diagnoses (comorbidity burden)
- Specific conditions: sepsis, respiratory failure, cardiac arrest

### 3. Data Preprocessing
- **Missing Value Handling**: Population-based median imputation for clinical values
- **Outlier Removal**: Filters extreme LOS (>30 days) and invalid records
- **Feature Scaling**: StandardScaler for normalization (mean=0, std=1)
- **Train/Test Split**: 80/20 split with stratification

### 4. Machine Learning Models

#### 📈 Linear Regression
- L2 regularization (Ridge regression)
- Fast, interpretable baseline model

#### 🌲 Random Forest
- 100 trees, max depth 10
- Handles non-linear relationships and feature interactions
- Built-in feature importance

### 5. Model Evaluation
- **RMSE**: Root Mean Square Error (days)
- **MAE**: Mean Absolute Error (days)  
- **R²**: Coefficient of determination (explained variance)

## 🔧 Technical Architecture

```
MIMIC-III Raw Data → Feature Engineering → ML Pipeline → Predictions
     ↓                      ↓                 ↓            ↓
   6 Tables          39 Features      Train/Test      LOS Forecast
   (Millions         (5 Categories)    (80/20)        (Days)
   of records)
```

## 🎯 Use Case

This system predicts how long a patient will stay in the ICU based on:
- **Patient characteristics** when they arrive
- **Clinical measurements** in the first 24 hours
- **Hospital operational factors**

**Applications**: Resource planning, bed management, care coordination, and early identification of patients likely to have extended stays.

## 🚀 Key Features

- **Scalable**: Built on PySpark for handling millions of medical records
- **Comprehensive**: 39 carefully engineered features across 5 clinical domains
- **Real-time Ready**: Uses only data available within 24 hours of ICU admission
- **Validated**: Multiple model comparison with standard regression metrics
- **Production-Ready**: Proper train/test split, feature scaling, and error handling





# ☁️ Google Cloud Setup & Deployment

### Prerequisites
- Google Cloud Platform account with billing enabled
- Project ID: `bdcc2025-456512`
- Enabled APIs: Dataproc, Compute Engine, Cloud Storage

### Storage Setup
```bash
# Create bucket for MIMIC-III data
gsutil mb gs://dataproc-staging-europe-west4-719881989993-sa4vn92s

# Upload MIMIC-III CSV files (compressed)
gsutil -m cp -r mimic-iii-clinical-database-1.4/ gs://your-bucket/mimic-data/
```

### Dataproc Cluster Configuration
```bash
# Create optimized cluster with Jupyter
gcloud dataproc clusters create mimic-cluster \
    --num-workers=6 \
    --worker-machine-type=e2-highmem-4 \
    --worker-boot-disk-size=100GB \
    --optional-components=JUPYTER \
    --enable-component-gateway \
    --region=europe-west4 \
    --project=bdcc2025-456512
```

### Cluster Specifications
- **Master Node**: 1x e2-highmem-4 (4 vCPUs, 32GB RAM)
- **Worker Nodes**: 6x e2-highmem-4 (4 vCPUs, 32GB RAM each)
- **Total Resources**: 28 vCPUs, 224GB RAM
- **Storage**: 100GB boot disk per node
- **Jupyter Access**: Via Component Gateway (secure web interface)

### Service Account Permissions
```bash
# Grant necessary permissions
gcloud projects add-iam-policy-binding bdcc2025-456512 \
    --member="serviceAccount:719881989993-compute@developer.gserviceaccount.com" \
    --role="roles/storage.objectAdmin"
```


## 🎯 Use Case

This system predicts how long a patient will stay in the ICU based on:
- **Patient characteristics** when they arrive
- **Clinical measurements** in the first 24 hours
- **Hospital operational factors**

**Applications**: Resource planning, bed management, care coordination, and early identification of patients likely to have extended stays.

## 🚀 Key Features

- **Scalable**: Built on PySpark for handling millions of medical records
- **Comprehensive**: 39 carefully engineered features across 5 clinical domains
- **Real-time Ready**: Uses only data available within 24 hours of ICU admission
- **Validated**: Multiple model comparison with standard regression metrics
- **Production-Ready**: Proper train/test split, feature scaling, and error handling

