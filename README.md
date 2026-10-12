# RF Intrusion Detection MLOps Project

This project classifies six RF activity classes from spectrogram images and demonstrates an end-to-end AWS/SageMaker workflow: data preparation, CNN training, batch inference, monitoring, and CI/CD promotion.

## Where to run the notebooks

Run this project in **Amazon SageMaker Studio** unless a notebook says otherwise. Notebooks 01, 02, 03, and 05 use the SageMaker environment and/or AWS services such as S3, Athena, CloudWatch, Model Registry, and Batch Transform.

**Exception — Notebook 04:** `RF_Intrusion_04_CNN_v3_v4_Training_and_Evaluation.ipynb` must be run locally through Docker Desktop with NVIDIA GPU access. Running its CNN v3/v4 training in SageMaker Studio is not appropriate for this project because the training run is expected to exceed the available SageMaker session time before completing. Notebook 04 saves the resulting artifacts in `models/tensorflow_training/`; upload the required `.keras` files to the configured S3 locations before running Notebook 05.

## Configure your AWS environment

The notebooks do not store AWS credentials. In SageMaker Studio, authentication comes from the attached IAM role; when running locally, use an AWS CLI profile, AWS SSO, or environment-based credentials.

Before running Notebooks 02, 03, or 05, create your personal project configuration from the committed template:

```powershell
Copy-Item .config/project.example.json .config/project.json
```

Edit `.config/project.json` with your AWS Region, S3 bucket, role ARN, and project resource names. This personal file is ignored by Git; do not commit credentials or account-specific values. The tracked `.config/project.example.json` documents every required setting.

## Notebook guide

Open the notebooks in this project order:

1. `RF_Intrusion_01_Original_Setup_and_EDA.ipynb` — introduces the RF intrusion problem, explores the original dataset, and documents the initial project setup.
2. `RF_Intrusion_02_Data_Lake_v2_Training_and_Baseline_Batch_Transform.ipynb` — the baseline CNN v1/v2 workflow: creates the 40/10/10/40 split, uploads the raw and curated data lake, builds Athena metadata, trains/evaluates v1 and v2, and demonstrates v2 Batch Transform.
3. `RF_Intrusion_03_Model_and_Data_Monitoring.ipynb` — this is the monitoring workflow. It compares model and input-data behavior with saved baselines, detects drift, publishes CloudWatch model-quality and Batch Transform infrastructure metrics, creates alarms and a dashboard, produces SageMaker-style monitoring reports, and verifies report uploads to S3.
4. `RF_Intrusion_04_CNN_v3_v4_Training_and_Evaluation.ipynb` — runs local Docker/Desktop TensorFlow GPU training for v3 and v4. V3 is intentionally weak to demonstrate a failed promotion gate; v4 is the promoted candidate.
5. `RF_Intrusion_05_v3_v4_CI_CD_Promotion_and_Batch_Transform.ipynb` — the final v3/v4 promotion workflow. It uses SageMaker SDK v3 to validate, evaluate/package, performance-gate, register, create a model, and run Batch Transform for an already-uploaded candidate artifact.

## Workflow relationship

Notebook 02 is the baseline v1/v2 path: it creates the curated dataset, trains the original models, and demonstrates v2 Batch Transform. Notebook 04 produces the v3/v4 candidate `.keras` artifacts used by Notebook 05. Notebook 05 is the final promotion path: it does not retrain a model, but promotes a supplied v3/v4 artifact only when its accuracy and macro-F1 gates pass before registration and Batch Transform. Notebook 03 is the finalized monitoring implementation: it uses baseline metrics and completed inference results to assess model-quality and data-quality drift, then surfaces operational signals through CloudWatch and uploads monitoring reports to S3.

## Local GPU training

Notebook 04 includes the Docker Desktop workflow for local NVIDIA GPU training. It requires Docker Desktop with the WSL 2 engine, a WSL-compatible NVIDIA driver, and a successful `docker run --rm --gpus all ubuntu nvidia-smi` check before launching the TensorFlow Jupyter image. Do not run its v3/v4 training cells in SageMaker Studio; use the local GPU container instead.

## Data, metadata, and local model artifacts

- `rf_intrusion_assignment_split_metadata.csv` records each image's source and assigned dataset split.
- `rf_intrusion_athena_metadata.csv` adds record IDs and S3 locations for Athena queries.
- `monitoring_baseline.json` supplies the baseline consumed by the monitoring notebook.
- `improved_cnn_rf_intrusion_v2_metadata.json` documents CNN v2 architecture and evaluation results.
- `models/tensorflow_training/` contains the local Keras model artifacts, evaluation metadata, confusion matrices, and deployment ZIP packages created by Notebook 04. The folder currently contains v1–v4 artifacts; its four `.keras` files total about 20.7 MiB.

`models/tensorflow_training/` is intentionally ignored by Git because it contains reproducible, generated binary artifacts and duplicate deployment packages. Keep the models locally for evaluation and upload the required version to S3 for CI/CD; do not commit them to the repository.

AWS-mutating cells are clearly marked or commented in the SageMaker notebooks. Review S3 bucket names, IAM role ARNs, and artifact paths before running them in a different account.
