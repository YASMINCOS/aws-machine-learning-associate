# AWS Certified Machine Learning Engineer – Associate (MLA-C01)
## Exam Preparation Guide

> This guide covers every domain and topic tested on the MLA-C01 exam.
> Built from hands-on study, practice exams, and deep dives into AWS documentation.

---

## Exam Overview

| Property | Detail |
|---|---|
| **Exam code** | MLA-C01 |
| **Duration** | 170 minutes |
| **Questions** | 65 questions |
| **Passing score** | 720 / 1000 |
| **Format** | Multiple choice, multiple select, ordering/sequencing |
| **Price** | $300 USD |

## Domain Weights

| Domain | Topic | Weight |
|---|---|---|
| 1 | Data Preparation for Machine Learning | **28%** |
| 2 | ML Model Development | **26%** |
| 3 | Deployment and Orchestration of ML Workflows | **22%** |
| 4 | ML Solution Monitoring, Maintenance, and Security | **24%** |

---

# DOMAIN 1 — Data Preparation for Machine Learning (28%)

## 1.1 Data Formats

| Format | Type | Best for | Avoid when |
|---|---|---|---|
| **CSV** | Row-based, text | Small datasets, debugging | Large data, analytics |
| **Parquet** | Columnar, binary | Analytics, ML training, large tabular | Streaming row-by-row |
| **JSON / JSON Lines** | Row-based, text | Nested data, logs, APIs | Large flat tabular |
| **Avro** | Row-based, binary | Streaming, evolving schemas (Kafka) | Heavy analytics |
| **RecordIO-Protobuf** | Binary, SageMaker-native | **SageMaker built-in algorithms** | Non-SageMaker |
| **ORC** | Columnar | Hive/Hadoop ecosystems | Outside Hadoop |

### Why Parquet for ML
- Column pruning → read only columns needed → cheaper Athena queries
- Compresses 5–10× better than CSV
- Splittable → parallel distributed reads
- Schema embedded → no guessing

### Exam keyword → Format
- "Large tabular ML data" → **Parquet**
- "SageMaker built-in algorithm, fastest training" → **RecordIO-Protobuf**
- "Streaming with evolving schema" → **Avro**
- "Athena queries on S3, minimize cost" → **Parquet**

---

## 1.2 Data Ingestion Services

### Batch ingestion
| Service | Purpose |
|---|---|
| **AWS Glue** | Managed Spark/Python ETL, schema crawling, Data Catalog |
| **AWS DMS + CDC** | Continuous DB replication from on-prem/RDS → S3 |
| **AWS DataSync** | Bulk file transfer NFS/SMB → S3/EFS/FSx |
| **AWS Snowball** | Physical device transfer for huge data over slow network |
| **AWS Transfer Family** | SFTP/FTPS → S3 |

### Streaming ingestion
| Service | Purpose |
|---|---|
| **Kinesis Data Streams** | Raw stream, replay, multiple consumers |
| **Kinesis Data Firehose** | Managed delivery to S3/Redshift/OpenSearch — no code |
| **Amazon MSK** | Managed Kafka |
| **Managed Apache Flink** | Stream processing, windows, joins, aggregations |

### Decision rules
| Scenario | Answer |
|---|---|
| "Continuous DB changes to S3" | DMS with CDC |
| "100 TB, slow internet, one-time" | Snowball |
| "5 TB, 1 Gbps, one-time" | DataSync |
| "Stream → S3 without code" | Firehose |
| "Stream aggregations with windows" | Managed Flink |
| "Discover schema of S3 CSV files" | Glue Crawler |
| "SQL over S3 files" | Athena |

---

## 1.3 Storage for ML

| Storage | Type | Use case |
|---|---|---|
| **S3** | Object | Default — datasets, artifacts, everything |
| **EBS** | Block (single instance) | Training instance scratch disk, notebook disk |
| **EFS** | NFS (multi-instance) | Shared notebooks, SageMaker Studio home dirs |
| **FSx for Lustre** | High-performance filesystem | Distributed training, huge datasets read repeatedly |
| **DynamoDB** | Key-value | Feature Store online — sub-100ms inference lookups |

### S3 training input modes
| Mode | How it works | Use when |
|---|---|---|
| **File mode** | Full copy to EBS before training | Small/medium datasets |
| **Pipe mode** | Stream from S3 during training | Large datasets, supported algorithms |
| **FastFile mode** | Lazy mount, on-demand reads | Large datasets, random access |

---

## 1.4 SageMaker Feature Store

- **Online store** → DynamoDB-backed, sub-100ms for real-time inference
- **Offline store** → S3 (Parquet), time-travel queries for point-in-time correct features
- Solves **train/serve skew** — same features in training and inference
- `FeatureGroup` = logical container for related features
- Each record has a **record identifier** + **event time**

---

## 1.5 Data Transformation Tools

| Tool | Where | Use |
|---|---|---|
| **SageMaker Data Wrangler** | Inside Studio | Visual ML prep, 300+ transforms, export to Pipeline |
| **AWS Glue DataBrew** | Outside Studio | No-code visual prep for business users |
| **AWS Glue** | Managed Spark ETL | Large-scale custom ETL, Data Catalog |
| **SageMaker Processing Jobs** | SageMaker | Reproducible containerized prep steps |
| **Amazon Athena** | Serverless SQL | Ad-hoc SQL queries on S3 data |

---

## 1.6 Feature Engineering

### Scaling (critical — memorize all)
| Scaler | Formula | Use when | Avoid when |
|---|---|---|---|
| **Standard / Z-score** | `(x - mean) / std` | Normal distribution, no outliers | Outliers present |
| **Min-Max** | `(x - min) / (max - min)` | Neural nets, need [0,1] range | Outliers present |
| **Robust Scaler** | `(x - median) / IQR` | **Outliers present**  | Clean data |
| **Max Absolute** | `x / max(|x|)` | Sparse data, preserve zeros | Dense data |
| **L1 Normalization** | row / Σ|row| | Text vectors (TF-IDF) | Tabular features |
| **L2 Normalization** | row / ||row|| | Embeddings | Tabular features |

> **Standard Scaler = Z-score normalization** (same thing — exam trap)

### Categorical Encoding
| Method | Use when | Risk |
|---|---|---|
| **One-Hot** | Nominal, low cardinality (<20) | Cardinality explosion |
| **Ordinal** | True ordered (low/med/high) | Implies false order if not ordinal |
| **Target Encoding** | High cardinality (100+) | Leakage if not done with CV |
| **Frequency** | High cardinality, simple | Loses identity |
| **Hashing** | Very high cardinality (1000+), streaming | Hash collisions |

### Missing value imputation
| Strategy | When |
|---|---|
| Drop rows | Few rows missing, large dataset |
| Drop column | Column >50% missing |
| **Mean** | Numeric, no outliers, normal distribution |
| **Median** | Numeric WITH outliers or skew |
| **Mode** | Categorical |
| **Forward fill** | Time series |
| **Indicator + impute** | Missingness itself has signal |

### Numeric transforms
| Transform | Use when |
|---|---|
| **Log** | Right-skewed data, large positive values |
| **Box-Cox** | Make distribution normal (strictly positive) |
| **Yeo-Johnson** | Make distribution normal (any sign) |
| **Binning** | Convert continuous to categorical |
| **Cyclical (sin/cos)** | Hour of day, day of week, month — preserves cyclicality |

---

## 1.7 Data Quality

| Tool | Use |
|---|---|
| **Glue Data Quality (DQDL)** | Continuous rules-based validation in ETL pipeline |
| **Data Wrangler Insights Report** | One-click profile: missing values, outliers, imbalance |
| **SageMaker Model Monitor (Data Quality)** | Drift detection in production |

---

## 1.8 Bias Detection (Pre-training)

> All pre-training bias metrics are in **SageMaker Clarify**

| Metric | Measures |
|---|---|
| **DPL** | Difference in proportion of positive outcomes across groups |
| **CI** | Class imbalance between facets |
| **KL / JS** | Distribution divergence between facet labels |
| **KS** | Max difference in cumulative distributions |

---

## 1.9 Data Splitting

| Strategy | When |
|---|---|
| **Random** | i.i.d. balanced data |
| **Stratified** | Imbalanced classification — preserves class proportions |
| **Ordered / Chronological** | **Time series — NEVER random** |
| **Group split** | Same entity in same split (prevents leakage) |

---

## 1.10 Data Leakage (very testable)

| Type | What it is | Prevention |
|---|---|---|
| **Target leakage** | Feature derived from the label | Audit features — "would I know this at prediction time?" |
| **Temporal leakage** | Random split on time series | Chronological split only |
| **Preprocessing leakage** | Scaler fit on full dataset | Split FIRST, then fit on train only |
| **Group leakage** | Same user in train and test | Group split |

---

## 1.11 Data Labeling

| Component | Purpose |
|---|---|
| **Ground Truth** | Managed labeling service |
| **Active learning** | Auto-labels confident examples; humans label uncertain ones (reduces cost) |
| **Private workforce** | Your team — for sensitive/proprietary data |
| **Public workforce** | Mechanical Turk — cheap, for non-sensitive data |
| **Ground Truth Plus** | Fully managed (AWS handles workforce coordination) |

---

# DOMAIN 2 — ML Model Development (26%)

## 2.1 Problem Framing → Algorithm Selection

| Problem type | SageMaker built-in | AWS managed service |
|---|---|---|
| Binary classification | XGBoost, Linear Learner | Comprehend, Fraud Detector |
| Multiclass | XGBoost, Linear Learner, K-NN, BlazingText | Comprehend, Rekognition |
| Regression | XGBoost, Linear Learner, K-NN | Forecast |
| Time-series forecasting | **DeepAR** | Amazon Forecast |
| Clustering | K-Means | — |
| Anomaly detection | **Random Cut Forest**, IP Insights | — |
| Dimensionality reduction | PCA | — |
| Word embeddings | **BlazingText (Word2Vec)** | — |
| Text classification | **BlazingText (supervised)**, Text Classification TF | Comprehend |
| Topic modeling | **LDA**, NTM | Comprehend (topic modeling) |
| Recommendations (sparse) | **Factorization Machines** | Personalize |
| Image classification | Image Classification (built-in) | Rekognition Custom Labels |
| Object detection | Object Detection (built-in) | Rekognition |
| Translation / summarization | Seq2Seq, JumpStart transformers | Amazon Translate |
| NLP entities/sentiment | Comprehend | — |
| Medical entities (PHI) | — | **Comprehend Medical** |
| Foundation models (API) | — | **Amazon Bedrock** |
| Foundation models (fine-tune) | JumpStart | Bedrock (custom models) |

---

## 2.2 Key Built-in Algorithm Details

### XGBoost
- Best general-purpose tabular algorithm
- **Scale-invariant** (no scaling needed)
- Objectives: `binary:logistic`, `multi:softmax`, `reg:squarederror`
- Key HPs: `num_round`, `max_depth`, `eta`, `subsample`

### Linear Learner
- `predictor_type`: `binary_classifier`, `multiclass_classifier`, `regressor`
- `target_precision` / `target_recall` — optimize constrained metrics
- Supports L1/L2 regularization internally

### DeepAR
- Probabilistic forecasting — outputs quantile distributions
- Key HPs: `context_length`, `prediction_length`, `epochs`, `likelihood`
- Input: JSON Lines format
- Default metric: **Average wQL**

### BlazingText
| Mode | Purpose | Input format |
|---|---|---|
| `supervised` | Text classification | `__label__X text` per line |
| `skipgram` | Word2Vec (rare words) | Plain text, sentence per line |
| `cbow` | Word2Vec (faster) | Plain text, sentence per line |
| `batch_skipgram` | Distributed Word2Vec (multi-CPU) | Plain text, sentence per line |

### LDA (Latent Dirichlet Allocation)
- **Unsupervised topic modeling**
- No labels needed — discovers hidden themes
- Key HP: `num_topics` (you choose; try several, evaluate perplexity)
- Output: topic distribution per document + word distribution per topic

### Random Cut Forest (RCF)
- **Unsupervised anomaly detection**
- Outputs anomaly score per record
- Key HPs: `num_trees`, `num_samples_per_tree`

### Factorization Machines
- Sparse high-dimensional data (user-item interactions, CTR)
- `predictor_type`: `binary_classifier` or `regressor`
- Input: **RecordIO-Protobuf only**

---

## 2.3 Evaluation Metrics

### Classification
| Metric | Use when | Direction |
|---|---|---|
| **Accuracy** | Balanced classes | Higher ↑ |
| **BalancedAccuracy** | Imbalanced, equal class importance | Higher ↑ |
| **Precision** | FP is costly (spam filter) | Higher ↑ |
| **Recall (TPR)** | FN is costly (cancer, fraud, security) | Higher ↑ |
| **F1** | Imbalanced, balance P and R | Higher ↑ |
| **AUC-ROC** | Balanced, threshold-independent ranking | Higher ↑ |
| **AUC-PR** | Heavily imbalanced, rare positives | Higher ↑ |
| **Log Loss** | Probability calibration matters | Lower ↓ |

> **Accuracy on imbalanced data = always wrong answer**
> **Standard Scaler = Z-score = same thing**
> **AUC measures ranking; Log Loss measures calibration**

### Regression
| Metric | Use when | Direction |
|---|---|---|
| **RMSE** | Default, large errors costly | Lower ↓ |
| **MAE** | Outliers exist, don't let them dominate | Lower ↓ |
| **MAPE** | Cross-scale comparison | Lower ↓ |
| **R²** | % variance explained | Higher ↑ |

### Forecasting
| Metric | Use when | Direction |
|---|---|---|
| **Average wQL** | Probabilistic forecast (P10/P50/P90) — **default Canvas/Forecast** | Lower ↓ |
| **MAPE / WAPE** | Percentage-based forecast error | Lower ↓ |

> **DeepAR + Canvas + Forecast + quantiles → Average wQL**

---

## 2.4 Hyperparameter Tuning (AMT)

| Strategy | When to use |
|---|---|
| **Bayesian** | Few expensive trials (20–50 budget) |
| **Random** | Many cheap parallel trials |
| **Grid** | Small discrete search spaces only |
| **Hyperband** | Deep learning — intelligent early stopping |

- **Warm start** — reuse previous tuning results
- **Early stopping** — abort underperforming jobs automatically
- Objective metric + direction (Maximize/Minimize)

---

## 2.5 Regularization

| Technique | Use |
|---|---|
| L1 (Lasso) | Feature selection — pushes weights to zero |
| L2 (Ridge) | Small weights — prevents large coefficients |
| Dropout | Neural nets — random neuron drop during training |
| Early stopping | Stop when val loss stops improving |
| Data augmentation | Images, text — increases effective dataset size |

### Overfitting vs Underfitting
| Symptom | Diagnosis | Fix |
|---|---|---|
| Train low, val high | Overfitting | More regularization, more data, simpler model |
| Train high, val high | Underfitting | More capacity, more features, longer training |
| Loss = NaN | Divergence | Lower learning rate, gradient clipping |

---

## 2.6 Transfer Learning and Fine-tuning

| Approach | When |
|---|---|
| Feature extraction (freeze base) | Small dataset, similar domain |
| Fine-tuning | Larger dataset, want better fit |
| PEFT / LoRA | LLMs, limited compute — train adapters only |
| Full fine-tuning | Different domain, lots of data |

> **"Fine-tune LLM efficiently with small dataset"** → **PEFT/LoRA via JumpStart**

---

## 2.7 Training Cost Optimization

| Technique | Savings |
|---|---|
| **Managed Spot Training + checkpointing** | Up to 90% |
| **Trainium (trn1/trn2) + Neuron SDK** | Lower cost than equivalent GPU |
| **SageMaker Training Compiler** | Faster DL compilation → less wall time |
| Distributed training (data parallel) | Faster wall clock |

---

## 2.8 Distributed Training

| Strategy | When |
|---|---|
| **Data parallelism** | Dataset huge, model fits on one GPU |
| **Model parallelism (SMP)** | Model too large for one GPU (LLMs) |
| **EFA (Elastic Fabric Adapter)** | Low-latency inter-node networking |
| **Trainium trn1/trn2** | Cost-optimized large DL training |

---

## 2.9 SageMaker Clarify

### Pre-training bias
- DPL, CI, KL, JS, LP, TVD, KS, CDDL

### Post-training bias
- DPPL, DI (Disparate Impact — the "4/5 rule"), RD, DAR, DRR

### Explainability
- **SHAP** = local explanation for a specific prediction
- **PDP** = global feature effect (how prediction changes as feature varies)

> DPL = pre-training bias (data)
> DPPL = post-training bias (model)
> SHAP = individual prediction explanation
> PDP = global feature behavior

---

# DOMAIN 3 — Deployment and Orchestration (22%)

## 3.1 Endpoint Types

| Type | Payload | Timeout | GPU | Scales to zero | Cost |
|---|---|---|---|---|---|
| **Real-time** | 6 MB | 60s | Yes | No (min 1 instance) | Always-on |
| **Serverless** | 4 MB | 60s | **No** | **Yes** | Per ms |
| **Asynchronous** | **1 GB** | **1 hour** | Yes | **Yes** | Per use |
| **Batch Transform** | No limit | N/A | Yes | N/A | Per job |

### When to use which
| Scenario | Answer |
|---|---|
| "Payload 800 MB" | Async |
| "Inference takes 30 min" | Async |
| "Sporadic traffic, pay zero idle" | Serverless |
| "Constant real-time <100ms" | Real-time |
| "Score millions offline weekly" | Batch Transform |
| "100 small models, lightly used" | Multi-Model Endpoint |
| "Small model + S3 trigger" | Lambda |
| "Large/GPU model + S3 trigger" | Lambda → Async Endpoint |

---

## 3.2 Multi-Model Endpoint (MME)

- Host multiple models in ONE container, ONE instance
- Models loaded into memory on demand from S3
- Great for: many similar small models, same framework
- Cost: fraction of hosting N separate endpoints

---

## 3.3 Production Variants and Shadow

| Feature | Purpose |
|---|---|
| **Production Variants** | Split real traffic with weights (A/B test) |
| **Shadow Variant** | Copy of traffic, predictions NOT shown to users |

---

## 3.4 Deployment Strategies

| Strategy | How | Risk | Auto-rollback |
|---|---|---|---|
| **All-at-once** | 100% at once | High | No |
| **Canary** | Small % first, then 100% | Medium | Via alarms |
| **Linear** | Equal steps over time | Low | Via alarms |
| **Blue/Green** | Parallel environment, switch when ready | Low | **Yes — via CloudWatch alarms** |
| **Shadow** | Copy of traffic, no user impact | Minimal | N/A |

> "Auto-rollback on metric degradation" → **Blue/Green with CloudWatch alarms**
> "Equal steps, e.g., +10% every 5 min" → **Linear**
> "Test without exposing to users" → **Shadow**

---

## 3.5 Container Paths

| Path | When |
|---|---|
| Built-in algorithm | No code needed |
| Script mode (framework containers) | Custom script, AWS-managed container |
| BYOC | Custom framework or system libs |

### BYOC contract
- Listen on port **8080**
- Expose `/ping` (health check)
- Expose `/invocations` (inference)
- Read model from `/opt/ml/model/`

---

## 3.6 SageMaker Neo

- Compiles models for specific hardware (Inferentia, ARM, GPU, x86)
- Smaller model, faster inference, cheaper hardware
- Used for: edge devices, cost optimization

---

## 3.7 CloudFormation Resources

```
Deployment chain: Model → EndpointConfig → Endpoint
```

| Resource | Purpose |
|---|---|
| `AWS::SageMaker::Model` | Artifact + container reference |
| `AWS::SageMaker::EndpointConfig` | Instance type, variants, KmsKeyId, DataCapture |
| `AWS::SageMaker::Endpoint` | Actual hosted URL + DeploymentConfig |
| `AWS::SageMaker::ModelPackage` | Versioned model in Registry |
| `AWS::SageMaker::ModelPackageGroup` | Container for model versions |
| `AWS::SageMaker::Project` | MLOps template (Service Catalog) |
| `AWS::SageMaker::Pipeline` | Pipeline workflow definition |
| `AWS::SageMaker::FeatureGroup` | Feature Store group |
| `AWS::SageMaker::Domain` | Studio domain |
| `AWS::SageMaker::MonitoringSchedule` | Model Monitor schedule |

> KmsKeyId (endpoint encryption) → **EndpointConfig**
> DataCaptureConfig → **EndpointConfig**
> DeploymentConfig (Blue/Green) → **Endpoint**

---

## 3.8 Orchestration Tools

| Tool | Best for |
|---|---|
| **SageMaker Pipelines** | Pure ML workflows — step caching, lineage, native ML steps |
| **AWS Step Functions** | Mixed workflows — ML + business logic + any AWS service |
| **Amazon MWAA** | Teams already using Apache Airflow |

### SageMaker Pipeline step types
`ProcessingStep`, `TrainingStep`, `TuningStep`, `TransformStep`, `CreateModelStep`, `RegisterModel`, `ConditionStep`, `LambdaStep`, `EMRStep`, `ClarifyCheckStep`, `QualityCheckStep`, `FailStep`

---

## 3.9 CI/CD for ML

| Service | Purpose |
|---|---|
| **CodePipeline** | Orchestrate CI/CD flow |
| **CodeBuild** | Build images, run tests |
| **CodeDeploy** | Application deployment |
| **CodeArtifact** | Private package repository |

### Two-pipeline pattern
1. **Build pipeline** — source → build → train → evaluate → register (Pending)
2. **Deploy pipeline** — picks Approved model → deploy (Blue/Green)

### Event-driven triggers
- New data in S3 → EventBridge → Pipeline (retraining)
- Model Approved → EventBridge → CodePipeline (deployment)

---

## 3.10 Model Registry

| Status | Meaning |
|---|---|
| `PendingManualApproval` | Waiting for review |
| `Approved` | Ready to deploy |
| `Rejected` | Not deployable |

---

## 3.11 Inference Recommender

- Tests multiple instance types automatically
- Returns cost/latency/throughput ranking
- **Default job** (~45 min): quick recommendations
- **Advanced job** (~2h): load test with your dataset + SLA
- > "Which instance should I use?" → **Inference Recommender**

---

# DOMAIN 4 — Monitoring, Maintenance, and Security (24%)

## 4.1 SageMaker Model Monitor — 4 Types

| Monitor | Detects | Baseline | Ground truth? |
|---|---|---|---|
| **DefaultModelMonitor** (Data Quality) | Feature distribution drift | Training dataset stats | No |
| **ModelQualityMonitor** | Accuracy/F1/RMSE drop | Metrics on labeled set | **Yes** |
| **ModelBiasMonitor** | Bias drift (DPL, etc.) | Clarify bias from training | Depends |
| **ModelExplainabilityMonitor** | SHAP importance drift | SHAP from training | No |

### Setup order
```
Enable Data Capture → Create Baseline → Configure Monitor → Schedule
```

### Drift types
| Type | What changed | Monitor |
|---|---|---|
| Data Quality | Input feature distribution | DefaultModelMonitor |
| Model Quality | Prediction accuracy | ModelQualityMonitor |
| Bias | Discrimination across groups | ModelBiasMonitor |
| Feature Attribution | SHAP importance ranking | ModelExplainabilityMonitor |
| Concept drift | X→Y relationship | Retraining (model quality drops) |
| Virtual drift | Feature distribution changed but X→Y intact | Data Quality monitor |

---

## 4.2 CloudWatch Metrics for Endpoints

| Metric | Measures |
|---|---|
| `ModelLatency` | Time model spent computing |
| `OverheadLatency` | Time SageMaker added (networking, serialization) |
| `Invocations` | Total requests |
| `Invocation5XXErrors` | Server-side errors |
| `InvocationsPerInstance` | Used for auto scaling target tracking |
| `CPUUtilization`, `GPUUtilization` | Resource usage |

> **ModelLatency high** → model code is the bottleneck
> **OverheadLatency high** → SageMaker infrastructure bottleneck

---

## 4.3 Auto Scaling

| Policy | When |
|---|---|
| **Target Tracking** | Maintain a metric (e.g., InvocationsPerInstance) — most common |
| **Step Scaling** | Alarm-based steps — fast burst response |
| **Scheduled Scaling** | Predictable time-based peaks |
| **Predictive Scaling** | ML-based forecast of future load |

> Combine Scheduled + Target Tracking for known spikes + general variance

---

## 4.4 Debugger and Profiler

| Tool | Detects |
|---|---|
| **SageMaker Debugger** | Training issues: vanishing gradient, overfitting, dead ReLU, poor init |
| **SageMaker Profiler** | System bottlenecks: CPU/GPU under-utilization, I/O slowness |

---

## 4.5 IAM for SageMaker

### iam:PassRole
- User calling `CreateTrainingJob` / `CreateEndpoint` must have `iam:PassRole` on the execution role
- **Most common IAM error** in SageMaker scenarios

### Execution role needs
- `s3:GetObject` / `s3:PutObject` on data buckets
- `ecr:GetAuthorizationToken`, `ecr:BatchGetImage`, `ecr:GetDownloadUrlForLayer`
- `logs:CreateLogStream`, `logs:PutLogEvents`
- `kms:Decrypt` / `Encrypt` / `GenerateDataKey` (if KMS-encrypted)

### Policy evaluation order
```
Explicit Deny → wins always
Allow needed from:
  1. Identity policy
  2. Resource policy (S3, KMS, ECR)
  3. Within Permissions Boundary
  4. Within SCP
  5. Within Session Policy
```

---

## 4.6 KMS Encryption for SageMaker

| What to encrypt | Parameter |
|---|---|
| Training EBS volume | `VolumeKmsKeyId` |
| Model artifact in S3 | `OutputDataConfig.KmsKeyId` |
| Endpoint EBS volume | `KmsKeyId` on EndpointConfig |
| Notebook EBS volume | `KmsKeyId` |
| Feature Store offline | KMS in `OfflineStoreConfig` |
| Inter-node traffic | `EnableInterContainerTrafficEncryption=true` |

> **Dual-policy rule**: both the IAM policy AND the KMS key policy must allow `kms:Decrypt`
> **AccessDenied on encrypted S3** → KMS key policy is missing the permission

---

## 4.7 Networking for SageMaker

### VPC Endpoints (critical — memorize)
| Service | Endpoint type |
|---|---|
| **S3** | **Gateway** (FREE) |
| **DynamoDB** | **Gateway** (FREE) |
| SageMaker API | Interface |
| SageMaker Runtime | Interface |
| ECR API / ECR DKR | Interface |
| CloudWatch Logs | Interface |
| STS | Interface |
| KMS | Interface |
| Secrets Manager | Interface |

### When to use what
| Scenario | Solution |
|---|---|
| Private subnet → S3 | **S3 Gateway VPC Endpoint** |
| Private subnet → SageMaker API | Interface Endpoint |
| Private subnet → PyPI / GitHub | **NAT Gateway** |
| Private subnet → ECR | Interface Endpoint |
| Customer VPC → invoke endpoint | **PrivateLink for SageMaker Runtime** |

### Error diagnosis
| Error | Type | Fix |
|---|---|---|
| `AccessDenied` | IAM | Check identity policy, resource policy, KMS |
| Timeout / connection refused | Network | Check VPC Endpoint, route table, Security Group |

### EnableNetworkIsolation
- `true` = container has **ZERO outbound network**
- Even AWS services are blocked
- All data must be pre-staged in `InputDataConfig`
- More restrictive than VPC mode

---

## 4.8 Security Services

| Service | Purpose |
|---|---|
| **Macie** | Discover PII in S3 buckets |
| **Comprehend** | Extract PII from text via API |
| **Comprehend Medical** | Extract PHI (healthcare) |
| **Inspector** | Scan ECR images for CVEs |
| **GuardDuty** | Detect anomalous API patterns, compromised credentials |
| **Security Hub** | Centralized security findings |
| **CloudTrail** | Log all AWS API calls (mandatory for compliance) |
| **VPC Flow Logs** | Network traffic metadata |
| **Secrets Manager** | Store DB passwords, API keys with rotation |

---

## 4.9 Responsible AI

| Feature | Purpose |
|---|---|
| **SageMaker Model Cards** | Document intended use, limitations, metrics |
| **Model Registry** | Versioned approval workflow |
| **Clarify** | Bias detection + SHAP explanations |
| **ModelBiasMonitor** | Continuous bias drift in production |
| **Bedrock Guardrails** | Content safety for GenAI (topic deny, PII filter) |

---

## 4.10 Cost Optimization

### Training
| Action | Savings |
|---|---|
| Managed Spot + checkpointing | Up to 90% |
| Trainium (trn1/trn2) | Lower than equivalent GPU |
| Training Compiler | Faster = less cost |

### Inference
| Action | Savings |
|---|---|
| Inferentia (inf1/inf2) | Lowest DL inference cost |
| Multi-Model Endpoint | Share instance across models |
| Serverless / Async | Scale to zero |
| SageMaker Neo | Smaller model, cheaper hardware |
| Inference Recommender | Find cheapest instance meeting SLA |

### Studio / notebooks
- Lifecycle configs to **auto-stop idle notebooks**

### Tools
| Tool | Purpose |
|---|---|
| **AWS Budgets** | Set cost thresholds + trigger actions |
| **Cost Explorer** | Analyze and forecast spend |
| **Cost Allocation Tags** | Per-team cost attribution |
| **CUR + Athena** | Per-resource hourly granularity |

---

# EXAM STRATEGY

## Question types

| Type | Strategy |
|---|---|
| **Single answer** | Eliminate wrong types first, then find best match |
| **Multi-select** | Each answer must independently solve part of the problem |
| **Select and order** | 1. Eliminate distractors. 2. Find dependencies. 3. "Reduce work early" |

## Select-and-order golden rules

```
Data optimization:  Filter → Transform (Parquet) → Compress
Data cleaning:      Impute → Handle outliers → Scale
Prevent leakage:    Split FIRST → Fit on train → Apply to test
PCA:                Scale → PCA → Train
Model Monitor:      Enable Data Capture → Baseline → Configure → Schedule
Endpoint chain:     Model → EndpointConfig → Endpoint
ML Pipeline:        Process → Train → Evaluate → Condition → Register
Event retraining:   S3 Event → EventBridge → Lambda → Pipeline
Approval deploy:    Register (Pending) → Approve → EventBridge → CodePipeline → Deploy
Safe deployment:    Shadow → compare offline → Blue/Green + rollback
Edge deployment:    Train → Neo compile → Greengrass deploy
```

## Trap patterns to memorize

| Trap | Correct answer |
|---|---|
| "Imbalanced data, evaluate model" | F1 / AUC-PR / Recall — NOT accuracy |
| "Private subnet, timeout reading S3" | S3 Gateway VPC Endpoint — NOT NAT |
| "Standard Scaler or Z-score?" | Same thing — Standard Scaler = Z-score |
| "Outliers, which scaler?" | Robust Scaler (median + IQR) |
| "User can't create training job — role error" | iam:PassRole missing |
| "AccessDenied on KMS-encrypted S3" | KMS key policy missing kms:Decrypt |
| "Across X AND Y — which chart?" | Heatmap (two dimensions) |
| "Distribution of continuous values" | Histogram (NOT density plot for business) |
| "LLM API, no training" | Bedrock |
| "LLM fine-tuning, efficient" | PEFT/LoRA via JumpStart |
| "Recommendations, no ML expertise" | Amazon Personalize |
| "Medical entities extraction" | Comprehend Medical |
| "EnableNetworkIsolation=true + needs S3" | Pre-stage data in InputDataConfig |
| "GPU inference, lowest cost" | Inferentia (inf1/inf2) |
| "DL training, lowest cost" | Trainium (trn1/trn2) |
| "100 TB, slow internet, one-time migration" | Snowball |
| "Probabilistic forecast with P10/P50/P90" | DeepAR / Average wQL |
| "Model deployed, want zero user risk testing" | Shadow Variant |
| "Auto-rollback on deployment" | Blue/Green + CloudWatch alarms |
| "Multiple notebooks share data" | EFS |
| "Distributed training, large dataset" | FSx for Lustre |


---

## Resources

- [AWS MLA-C01 Exam Guide](https://d1.awsstatic.com/training-and-certification/docs-ml-engineer-associate/AWS-Certified-Machine-Learning-Engineer-Associate_Exam-Guide.pdf)
- [SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [SageMaker Python SDK](https://sagemaker.readthedocs.io/)
- [AWS Skill Builder](https://skillbuilder.aws/)
- [AWS Well-Architected ML Lens](https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/welcome.html)
