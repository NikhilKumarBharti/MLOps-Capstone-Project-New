# Sentiment Analysis MLOps Pipeline

An end-to-end production-grade machine learning system for classifying customer review sentiment (positive/negative), built with industry-standard MLOps practices across the full lifecycle — from experimentation to monitored deployment on Kubernetes.

---

## Overview

This project demonstrates a complete MLOps workflow for a binary text classification task. It covers experiment tracking, data versioning, automated CI/CD pipelines, containerized deployment on AWS EKS, and real-time monitoring through Prometheus and Grafana.

---

## Problem Statement

Given a customer review, predict whether its sentiment is **positive** or **negative**. The system is designed to be reproducible, scalable, and production-ready.

---

## Architecture

```
Raw Data
   │
   ▼
Data Ingestion → Preprocessing → Feature Engineering → Model Building → Evaluation → Registration
   │                                                                                      │
   └──────────────────── DVC Pipeline (dvc repro) ───────────────────────────────────────┘
                                    │
                            MLflow on DagsHub
                         (Experiment Tracking & Model Registry)
                                    │
                              Flask REST API
                                    │
                            Docker Container
                                    │
                         AWS ECR (Image Registry)
                                    │
                  GitHub Actions CI/CD (Build → Test → Deploy)
                                    │
                           AWS EKS (Kubernetes)
                                    │
                    Prometheus (Metrics) → Grafana (Dashboards)
```

---

## Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.10 |
| Experiment Tracking | MLflow, DagsHub |
| Data Versioning | DVC, AWS S3 |
| Pipeline Orchestration | DVC Pipelines (dvc.yaml) |
| Web Framework | Flask |
| Containerization | Docker |
| Container Registry | AWS ECR |
| Orchestration | AWS EKS (Kubernetes) |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus, Grafana |
| Cloud | AWS (S3, ECR, EKS, IAM, EC2) |
| Project Structure | Cookiecutter Data Science |

---

## Project Structure

```
├── src/
│   ├── logger/
│   ├── data_ingestion.py
│   ├── data_preprocessing.py
│   ├── feature_engineering.py
│   ├── model/
│   │   ├── model_building.py
│   │   ├── model_evaluation.py
│   │   └── register_model.py
├── flask_app/
│   ├── app.py
│   ├── templates/
│   └── requirements.txt
├── tests/
├── scripts/
├── .github/
│   └── workflows/
│       └── ci.yaml
├── dvc.yaml
├── params.yaml
├── Dockerfile
└── README.md
```

---

## Pipeline Stages (DVC)

1. **Data Ingestion** — Load raw review data from source
2. **Data Preprocessing** — Clean and normalize text
3. **Feature Engineering** — Transform text into model-ready features
4. **Model Building** — Train classifier with tracked hyperparameters
5. **Model Evaluation** — Evaluate and log metrics to MLflow
6. **Model Registration** — Register best model to MLflow Model Registry

Run the full pipeline:

```bash
dvc repro
```

Check pipeline status:

```bash
dvc status
```

---

## Experiment Tracking

All experiments are tracked on DagsHub via MLflow:

- Hyperparameters logged from `params.yaml`
- Metrics (accuracy, F1, precision, recall) tracked per run
- Model artifacts versioned and registered

---

## Setup and Installation

### 1. Clone the repository and create environment

```bash
git clone <repo-url>
cd <repo-name>
conda create -n atlas python=3.10
conda activate atlas
pip install -r requirements.txt
```

### 2. Configure DagsHub credentials

```bash
export CAPSTONE_TEST=<your_dagshub_token>
```

### 3. Configure AWS credentials

```bash
aws configure
```

### 4. Run the DVC pipeline

```bash
dvc repro
```

### 5. Run the Flask app locally

```bash
cd flask_app
python app.py
```

---

## Docker

Build the image:

```bash
docker build -t capstone-app:latest .
```

Run with environment variable:

```bash
docker run -p 8888:5000 -e CAPSTONE_TEST=<your_dagshub_token> capstone-app:latest
```

---

## CI/CD Pipeline (GitHub Actions)

The workflow is defined in `.github/workflows/ci.yaml` and performs the following on every push:

1. Run unit tests (`tests/`)
2. Build Docker image
3. Push image to AWS ECR
4. Deploy updated image to AWS EKS

Required GitHub Secrets:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_ACCOUNT_ID
ECR_REPOSITORY
CAPSTONE_TEST
```

---

## Kubernetes Deployment (AWS EKS)

Create the EKS cluster:

```bash
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 --nodes-min 1 --nodes-max 1 \
  --managed --version 1.31
```

Verify deployment:

```bash
kubectl get pods
kubectl get svc flask-app-service
```

Access the application at the LoadBalancer external IP on port 5000.

---

## Monitoring

### Prometheus

- Deployed on an EC2 instance (t3.medium)
- Scrapes Flask app metrics every 15 seconds
- Accessible at `http://<prometheus-ec2-ip>:9090`

### Grafana

- Deployed on a separate EC2 instance (t3.medium)
- Connected to Prometheus as a data source
- Accessible at `http://<grafana-ec2-ip>:3000`
- Default credentials: `admin / admin`

---

## AWS Resource Cleanup

```bash
kubectl delete deployment flask-app
kubectl delete service flask-app-service
kubectl delete secret capstone-secret
eksctl delete cluster --name flask-app-cluster --region us-east-1
```

Also delete ECR repository and S3 bucket artifacts as needed. Verify CloudFormation stacks are fully removed.

---

## Data Versioning (DVC + S3)

Remote storage is configured on AWS S3:

```bash
dvc remote add -d myremote s3://<bucket-name>
dvc push
```

Pull versioned data on a fresh clone:

```bash
dvc pull
```

---

## License

This project is intended for educational and portfolio purposes.