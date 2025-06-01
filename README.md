# Big Data & Cloud Computing - Project #2



## References ML MIMIC

Alguns artigos para estudar a parte de ML e previsão LOS.

- https://fxis.ai/edu/how-to-use-mimic-iii-benchmarks-for-machine-learning-projects/
- https://colab.research.google.com/github/ponder-org/ponder-notebooks/blob/main/bigquery/example/MIMIC-III%20Health%20Record%20Analysis.ipynb#scrollTo=6a6ce75c-4527-487f-9eea-505f39c96165
- https://theoriginalsam.github.io/publications/13.pdf
- https://github.com/bmmalone/mimic-preprocessing
- https://github.com/YerevaNN/mimic3-benchmarks?tab=readme-ov-file#length-of-stay-prediction



# Descrição para o LOS Forecast


# ICU Length of Stay Prediction - MIMIC-III Pipeline

## 🎯 Objective
Predict ICU stay duration using PySpark ML on MIMIC-III dataset

## 📊 Data & Constraints
- **Sources**: 6 MIMIC-III tables (CHARTEVENTS, LABEVENTS, ICUSTAYS, etc.)
- **Filters**: Age 18-80, LOS 0.1-15 days, valid time sequences
- **Timeframe**: Vitals (first 24h), Labs (6h pre to 24h post ICU)

## 🔧 Features (39 total)
- **Demographics (2)**: Age, gender
- **Admission (8)**: Emergency/elective, timing, insurance
- **ICU Units (6)**: Care unit types, transfers
- **Vitals (11)**: HR, BP, RR, temp, SpO2 (avg/std)
- **Labs (8)**: Creatinine, glucose, electrolytes, blood counts
- **Diagnoses (4)**: Total count, sepsis, respiratory failure

## 🤖 Models & Results
- **Linear Regression**: R² 0.3-0.5, RMSE 2-3 days
- **Random Forest**: R² 0.4-0.6, RMSE 2-3 days (100 trees, depth 8)

## ☁️ Infrastructure
- **GCP Dataproc**: 6x e2-highmem-4 workers (28 vCPUs, 224GB RAM)
- **Optimizations**: Smart sampling, aggressive filtering, 80/20 split





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

