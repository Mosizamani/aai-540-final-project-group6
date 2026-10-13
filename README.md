# RF Intrusion Detection MLOps Project

This project classifies six RF activity classes from spectrogram images and demonstrates an end-to-end AWS/SageMaker workflow: data preparation, CNN training, batch inference, monitoring, and CI/CD promotion.

## Where to run the notebooks

Run Notebooks **01, 03, 04, and 06** in **Amazon SageMaker Studio**. They use the SageMaker environment and/or AWS services such as S3, Athena, CloudWatch, Model Registry, and Batch Transform.

Run Notebooks **02 and 05 locally** through Docker Desktop with NVIDIA GPU access. They train CNN models and should not be run as SageMaker Studio training jobs for this project: the local GPU container is the intended environment and avoids SageMaker session-time limits. After local training, upload the required `.keras` artifacts to the configured S3 model locations before running the related SageMaker workflows.

## Configure your AWS environment

The notebooks do not store AWS credentials. In SageMaker Studio, authentication comes from the attached IAM role; when running locally, use an AWS CLI profile, AWS SSO, or environment-based credentials.

Before running Notebooks 03, 04, or 06, create your personal project configuration from the committed template:

```powershell
Copy-Item .config/project.example.json .config/project.json
```

Edit `.config/project.json` with your AWS Region, S3 bucket, role ARN, and project resource names. This personal file is ignored by Git; do not commit credentials or account-specific values. The tracked `.config/project.example.json` documents every required setting.

## Notebook guide

Open the notebooks in this project order:

1. `RF_Intrusion_01_Original_Setup_and_EDA.ipynb` — introduces the RF intrusion problem, explores the original dataset, and documents the initial project setup.
2. `RF_Intrusion_02_Local_Training_CNN_v1_and_v2.ipynb` — runs local Docker/Desktop TensorFlow GPU training and evaluation for the baseline CNN v1 and improved CNN v2. It saves `.keras` artifacts, metrics JSON, and confusion-matrix images without using SageMaker or S3.
3. `RF_Intrusion_03_Data_Lake_v2_Training_and_Baseline_Batch_Transform.ipynb` — creates the 40/10/10/40 curated data lake, uploads raw/curated data and metadata to S3, builds Athena metadata, loads or evaluates v1/v2 artifacts, and demonstrates v2 Batch Transform.
4. `RF_Intrusion_04_Model_and_Data_Monitoring.ipynb` — compares model and input-data behavior with saved baselines, detects drift, publishes CloudWatch model-quality and Batch Transform infrastructure metrics, creates alarms and a dashboard, produces SageMaker-style monitoring reports, and verifies report uploads to S3.
5. `RF_Intrusion_05_CNN_v3_v4_Training_and_Evaluation.ipynb` — runs local Docker/Desktop TensorFlow GPU training and evaluation for v3 and v4. V3 is intentionally weak to demonstrate a failed promotion gate; v4 is the promotion candidate.
6. `RF_Intrusion_06_v3_v4_CI_CD_Promotion_and_Batch_Transform.ipynb` — the final v3/v4 promotion workflow. It uses SageMaker SDK v3 to validate, evaluate/package, performance-gate, register a Model Package, create a SageMaker Model, and run Batch Transform for an already-uploaded candidate artifact.

## Workflow relationship

Notebook 02 produces the local v1/v2 `.keras` artifacts. Notebook 03 builds the data lake and uses those baseline artifacts for evaluation and the v2 Batch Transform demonstration. Notebook 04 uses baseline metrics and completed inference results to assess model-quality and data-quality drift, then surfaces operational signals through CloudWatch and uploads monitoring reports to S3. Notebook 05 produces the local v3/v4 candidate `.keras` artifacts. Notebook 06 is the final promotion path: it does not retrain a model, but promotes a supplied v3/v4 artifact only when its accuracy and macro-F1 gates pass before Model Registry registration and Batch Transform.

## Local GPU training

Notebooks 02 and 05 include the Docker Desktop workflow for local NVIDIA GPU training. They require Docker Desktop with the WSL 2 engine, a WSL-compatible NVIDIA driver, and a successful `docker run --rm --gpus all ubuntu nvidia-smi` check before launching the TensorFlow Jupyter image. Do not run their CNN training cells in SageMaker Studio; use the local GPU container instead.

## Data, metadata, and local model artifacts

- `rf_intrusion_assignment_split_metadata.csv` records each image's source and assigned dataset split.
- `rf_intrusion_athena_metadata.csv` adds record IDs and S3 locations for Athena queries.
- `monitoring_baseline.json` supplies the baseline consumed by the monitoring notebook.
- `improved_cnn_rf_intrusion_v2_metadata.json` documents CNN v2 architecture and evaluation results.
- `artifacts/tensorflow_training/` is the local output location used by Notebook 02 for v1/v2 Keras artifacts, evaluation metadata, and confusion matrices.
- `models/tensorflow_training/` contains the retained local v1–v4 Keras artifacts, evaluation metadata, confusion matrices, and deployment packages. Its four `.keras` files total about 20.7 MiB.

Generated local model artifacts are intentionally ignored by Git because they are reproducible binaries and may include duplicate deployment packages. Keep them locally for evaluation and upload the required version to S3 for SageMaker evaluation, registry, and CI/CD; do not commit them to the repository.

AWS-mutating cells are clearly marked or commented in the SageMaker notebooks. Review S3 bucket names, IAM role ARNs, and artifact paths before running them in a different account.
