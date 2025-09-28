# Product Backlog: ACME Predictive Maintenance Platform (Revised)

This document outlines the product backlog for the Acme-Sub future state predictive maintenance system. It has been revised to provide greater detail, break down larger user stories, and incorporate industry best practices.

## Feature 1: On-Premises Data Acquisition and Secure Transmission to Cloud

**Description:** This feature covers the setup and configuration of on-premises systems to collect equipment data from SCADA systems and securely transmit it to the cloud for processing. The architecture will ensure data is captured locally and can be buffered during internet outages, a critical requirement for business continuity. This involves establishing a Unified Namespace (UNS) on a local MQTT broker, which acts as a central, structured hub for all OT data.

### User Story:  Install and Configure HiveMQ Broker for High Availability

**Description:** As a Data Engineer, I want to install and configure a HiveMQ MQTT broker cluster on our on-premises servers to ensure high availability and prevent a single point of failure for our local data pipeline.

**Acceptance Criteria:**
- A multi-node HiveMQ cluster is installed and running on the on-premises infrastructure at the pilot plant.
- The cluster is configured for automatic failover.
- Performance is benchmarked to handle the expected message throughput from all SCADA sources.
- Configuration is managed via Infrastructure as Code (IaC) and stored in the GitHub repository.

### User Story:  Define and Implement Unified Namespace (UNS) Topic Structure

**Description:** As an OT Architect, I want to design and implement a standardized, hierarchical topic structure (Unified Namespace) within HiveMQ, based on the ISA-95 standard, so that all OT data is organized, addressable, and context-rich.

**Acceptance Criteria:**
- A document defining the UNS topic hierarchy (e.g., `Enterprise/Site/Area/Line/Cell/Equipment/Tag`) is created and approved.
- The HiveMQ broker is configured to enforce this topic structure.
- Example data from a pilot line is published to the broker using the correct UNS topic format.

### User Story:  Secure HiveMQ Broker Communications

**Description:** As a Security Engineer, I want to secure the on-premises HiveMQ broker by enforcing encryption and strong authentication for all clients so that our OT data is protected from unauthorized access and tampering.

**Acceptance Criteria:**
- TLS encryption is enabled for all communication to and from the broker.
- All clients (e.g., Highbyte, model containers) are required to authenticate using client certificates (mTLS).
- Role-based access control is configured to restrict which clients can publish or subscribe to specific UNS topics.

### User Story:  Configure Highbyte for Ignition Data Publishing

**Description:** As an OT Engineer, I want to configure Highbyte to connect to our Ignition SCADA system, model the relevant data points, and publish them to the HiveMQ broker under the correct UNS topic.

**Acceptance Criteria:**
- A secure connection is established between Highbyte and the Ignition SCADA system.
- Data models for the pilot line's Ignition tags are created in Highbyte.
- Highbyte is configured to publish data to the HiveMQ broker, mapping the data to the defined UNS structure.
- Data flow is validated from Ignition to HiveMQ.

### User Story:  Configure Highbyte for Rockwell Data Publishing

**Description:** As an OT Engineer, I want to configure Highbyte to connect to our Rockwell SCADA system, model the relevant data points, and publish them to the HiveMQ broker under the correct UNS topic.

**Acceptance Criteria:**
- A secure connection is established between Highbyte and the Rockwell SCADA system.
- Data models for the pilot line's Rockwell tags are created in Highbyte.
- Highbyte is configured to publish data to the HiveMQ broker, mapping the data to the defined UNS structure.
- Data flow is validated from Rockwell to HiveMQ.

### User Story:  Establish Secure Bridge from HiveMQ to Event Hubs

**Description:** As a Data Engineer, I want to configure the HiveMQ Enterprise Extension for to securely bridge data to our Event Hubs namespace, using managed identities for authentication to avoid managing secrets.

**Acceptance Criteria:**
- The HiveMQ Extension is installed and configured on the on-prem broker.
- The bridge uses a Managed Identity with least-privilege access to authenticate to Event Hubs.
- The connection uses TLS 1.2+ for end-to-end encryption.
- Data from specific UNS topics is successfully bridged to the target Event Hub.

### User Story:  Implement and Test Data Buffering on HiveMQ Bridge

**Description:** As a Data Engineer, I want to configure and test the buffering mechanism on the HiveMQ to Event Hubs bridge to ensure no data is lost during a simulated internet outage.

**Acceptance Criteria:**
- The HiveMQ bridge is configured with a disk-based buffer of a specified size (e.g., capable of storing 24 hours of data).
- A test is conducted where the internet connection is severed for a period (e.g., 1 hour).
- Upon reconnection, the bridge successfully sends all buffered data to Event Hubs.
- No data loss is verified by comparing on-prem and cloud data counts.

## Feature 2: Cloud Data Ingestion and Processing (Medallion Architecture)

**Description:** This feature covers the ingestion of streaming data from Event Hubs into Microsoft Fabric and processing it through a multi-layered Medallion Architecture (Bronze, Silver, Gold). This ensures data is progressively cleaned, conformed, and aggregated, making it reliable and ready for diverse use cases like analytics and ML.

### User Story:  Provision and Configure Event Hubs for Scalability

**Description:** As a Cloud Engineer, I want to provision an Event Hubs namespace with a partition strategy that supports high-throughput and parallel consumption, so that our data ingestion pipeline is scalable and performant.

**Acceptance Criteria:**
- An Event Hubs namespace is provisioned using Infrastructure as Code (Bicep/Terraform).
- The Event Hub is configured with an appropriate number of partitions based on expected data volume.
- A dedicated consumer group is created for the Fabric streaming job.
- Private endpoint connectivity is enabled to restrict access to within the VNet.

### User Story:  Develop Bronze Ingestion Stream Processing Job

**Description:** As a Data Engineer, I want to develop a Fabric Spark streaming job that reads raw data from Event Hubs and writes it to the Bronze ADLS container as immutable Parquet files, partitioned by date.

**Acceptance Criteria:**
- The Spark job uses the Event Hubs connector to read the data stream.
- The job writes the raw event payload (including metadata) to ADLS Gen2 in Parquet format.
- Data is partitioned by year, month, and day for efficient querying.
- The job uses watermarking to handle late-arriving data and manages checkpoints for stream resiliency.

### User Story:  Implement a Dead-Letter Queue for Ingestion Errors

**Description:** As a Data Engineer, I want to implement a dead-letter mechanism for the Bronze ingestion job, so that any malformed or un-processable messages are captured for later analysis without halting the entire stream.

**Acceptance Criteria:**
- The Spark streaming job includes a `try/except` block to catch parsing errors.
- Messages that fail to process are written to a separate "dead-letter" directory in ADLS.
- An alert is configured to notify the data engineering team when new messages land in the dead-letter directory.

### User Story:  Develop Silver Layer Cleansing and Conforming Job

**Description:** As a Data Engineer, I want to develop a batch or streaming job that processes raw Bronze data, applies data quality rules, and conforms it into a well-defined schema in the Silver layer of the Fabric Lakehouse.

**Acceptance Criteria:**
- The job reads new data from the Bronze layer.
- Data is deduplicated based on a unique message identifier.
- The UNS topic is parsed into structured columns (Site, Area, Line, etc.).
- Data types are enforced, and null values are handled according to defined business rules.
- The cleaned and conformed data is written to Delta tables in the Silver layer.

### User Story:  Develop Gold Layer Aggregation Job for Analytics

**Description:** As a BI Developer, I want to create a data pipeline that aggregates Silver layer data into business-centric models (e.g., hourly/daily summaries of sensor readings) in the Gold layer, optimized for Power BI reporting.

**Acceptance Criteria:**
- A Fabric pipeline or notebook reads from Silver layer Delta tables.
- Data is aggregated into key metrics (e.g., averages, min/max values) over specific time windows.
- The aggregated data is stored in Gold layer Delta tables with a user-friendly schema.
- The resulting tables are performant for DirectQuery from Power BI.

## Feature 3: Predictive Maintenance Model Development and Training

**Description:** This feature encompasses the process of developing, training, and validating the predictive maintenance ML model within the Microsoft Fabric environment. This includes an initial port of the vendor's model, followed by in-house iteration and improvement using MLOps best practices.

### User Story:  Perform Exploratory Data Analysis (EDA) on Gold Layer Data

**Description:** As a Data Scientist, I want to perform a thorough exploratory data analysis on the curated Gold layer data to understand data distributions, identify correlations, and validate assumptions before model development.

**Acceptance Criteria:**
- A Fabric notebook is created for EDA.
- Visualizations (histograms, scatter plots, etc.) are generated to understand key features.
- A summary report of findings, including data quality issues or interesting patterns, is produced.

### User Story:  Translate and Adapt ModelQ's Feature Engineering to Fabric

**Description:** As a Data Scientist, I want to replicate the feature engineering logic from ModelQ's model in a Fabric notebook, using the Gold layer data as the source, so we can own and understand the data preparation process.

**Acceptance Criteria:**
- A Fabric notebook is created that contains the PySpark code for all feature engineering steps.
- The output of the notebook is a feature set that is structurally identical to the one used by the vendor.
- The notebook is parameterized and can be run as part of a larger pipeline.

### User Story:  Integrate Fabric Notebooks with MLflow for Automated Logging

**Description:** As a Data Scientist, I want to integrate our training notebooks with the MLflow tracking service in Fabric to automatically log all relevant experiment information, so we have a reproducible and auditable history of our model development.

**Acceptance Criteria:**
- The training notebook uses `mlflow.autolog()` or manual logging calls.
- For each experiment run, MLflow captures the Git commit hash, parameters, performance metrics, and the model artifact itself.
- Experiment results are visible and comparable in the Fabric UI.

### User Story:  Tune Hyperparameters and Train Final Model

**Description:** As a Data Scientist, I want to use a systematic approach like grid search or Bayesian optimization to tune the model's hyperparameters and train the final model on the full feature set, to ensure we achieve the best possible performance.

**Acceptance Criteria:**
- A notebook is created to perform hyperparameter tuning (e.g., using Hyperopt).
- The results of the tuning process are tracked in MLflow.
- The model with the optimal hyperparameters is re-trained on the entire training dataset.

### User Story:  Validate and Register Production-Candidate Model in MLflow

**Description:** As a Data Scientist, I want to validate the final trained model against a hold-out test set and, if it meets performance criteria, register it in the MLflow Model Registry and transition its stage to "Staging" or "Production".

**Acceptance Criteria:**
- The model's performance is evaluated on an unseen test dataset.
- The performance meets or exceeds the pre-defined business requirements (e.g., >95% precision).
- The model is saved to MLflow and registered in the Model Registry.
- The model version is transitioned to the "Staging" stage, triggering the CI/CD pipeline.

## Feature 4: MLOps and CI/CD Pipeline for Model Deployment

**Description:** This feature focuses on creating a robust, automated CI/CD pipeline using GitHub Actions. This pipeline will be responsible for containerizing the trained ML model, pushing it to a secure registry, and triggering the deployment to the on-premises Kubernetes cluster, following modern MLOps principles.

### User Story:  Develop a Production-Ready Inference Script

**Description:** As an MLOps Engineer, I want to develop a lightweight Python web service (e.g., using Flask or FastAPI) that loads the trained model and exposes a REST API endpoint for inference, so the model can be easily consumed within a container.

**Acceptance Criteria:**
- A Python script for the web service is created.
- The script includes a `/predict` endpoint that accepts input data and returns a model prediction.
- The script includes a `/health` endpoint for health checks.
- The script is stored in the model's GitHub repository.

### User Story:  Author a Dockerfile for Model Containerization

**Description:** As an MLOps Engineer, I want to author a multi-stage Dockerfile that efficiently packages the inference script, the trained model, and all necessary dependencies into a minimal, secure container image.

**Acceptance Criteria:**
- A `Dockerfile` is created in the GitHub repository.
- The Dockerfile uses a specific, versioned base image (e.g., `python:3.9-slim`).
- The resulting Docker image is small and free of unnecessary build dependencies.
- The image can be built and run successfully on a local machine.

### User Story:  Provision Container Registry (ACR) for Docker Images

**Description:** As a Cloud Engineer, I want to provision a secure Container Registry (ACR) to store our production model container images, so that we have a private, managed registry for our deployments.

**Acceptance Criteria:**
- An ACR instance is provisioned in using IaC.
- The registry is configured with private access, accessible only from our CI/CD agents and the on-prem Kubernetes cluster.
- A repository is created for the predictive maintenance model images.

### User Story:  Implement CI Pipeline to Build and Push Model Container

**Description:** As an MLOps Engineer, I want to create a GitHub Actions CI workflow that automatically builds, tags, and pushes the model's Docker image to ACR whenever a new model is tagged for release in the Git repository.

**Acceptance Criteria:**
- A GitHub Actions workflow file is created.
- The workflow is triggered by a new Git tag (e.g., `v1.2.0`).
- The workflow authenticates to ACR using a service principal or managed identity.
- The Docker image is built and pushed to ACR with a tag corresponding to the Git tag.

### User Story:  Implement CD Pipeline to Deploy Model via IoT Edge

**Description:** As an MLOps Engineer, I want to extend the GitHub Actions workflow to act as a CD pipeline that, after a successful image push, updates the IoT Edge deployment manifest to roll out the new container version to the on-prem cluster.

**Acceptance Criteria:**
- The workflow retrieves the IoT Edge deployment manifest from the repository.
- It updates the image version for the model module in the manifest.
- It uses the CLI or a dedicated GitHub Action to apply the updated manifest to the target IoT Hub device/cluster.
- The pipeline includes a manual approval step before deploying to production.

## Feature 5: On-Premises Model Deployment and Inference

**Description:** This feature covers the setup of the on-premises infrastructure to host the ML model and run inference locally. This is the most critical component for ensuring the plant can continue to operate and benefit from model predictions even during a complete internet outage.

### User Story:  Install and Configure a Production-Grade Kubernetes (K8s) Cluster

**Description:** As an Infrastructure Engineer, I want to install and configure a production-grade Kubernetes cluster on our on-prem servers, so we have a resilient and scalable platform for running our containerized ML model.

**Acceptance Criteria:**
- A multi-node K8s cluster is installed and configured.
- The cluster has role-based access control (RBAC) enabled.
- A private container registry (ACR) is configured as a trusted source for images.

### User Story:  Configure K8s Network Policies for Secure Broker Communication

**Description:** As a Security Engineer, I want to implement Kubernetes Network Policies to strictly control traffic between the ML model pod and the HiveMQ broker, ensuring the model can only communicate with the intended service.

**Acceptance Criteria:**
- A Network Policy is defined that allows egress traffic from the model pod only to the HiveMQ broker's IP and port.
- A Network Policy is defined that allows ingress traffic to the model pod only from trusted sources if it exposes an API.
- All other ingress and egress traffic from the model pod is denied by default.

### User Story:  Install and Register IoT Edge on the K8s Cluster

**Description:** As an Infrastructure Engineer, I want to install the IoT Edge runtime on our on-prem Kubernetes cluster and register it with our IoT Hub, so that it can be managed and receive deployment instructions from the cloud.

**Acceptance Criteria:**
- The IoT Edge daemonset is successfully deployed to the K8s cluster.
- The edge device is successfully registered in IoT Hub and shows a connected state.
- The `iotedge` command-line tool can be used to check the status of the runtime and modules.

### User Story:  Deploy and Validate the ML Model Container on the Edge

**Description:** As an MLOps Engineer, I want to perform the first deployment of the ML model container to the on-prem cluster via the CD pipeline and run a smoke test to validate that it's running correctly.

**Acceptance Criteria:**
- The CD pipeline successfully deploys the model container to the K8s cluster.
- The model pod starts and is in a `Running` state.
- The pod logs show a successful connection to the HiveMQ broker.
- A test message sent to the input topic results in a valid prediction on the output topic.

### User Story:  Implement Local Logging and Monitoring for the Deployed Model

**Description:** As an OT Engineer, I want to ensure that logs from the running model container are collected and available locally on the on-prem network, so that we can monitor the model's health and troubleshoot issues even if the cloud connection is down.

**Acceptance Criteria:**
- The Kubernetes cluster is configured to collect logs from all pods (e.g., using Fluentd or a similar log aggregator).
- Logs are forwarded to a local, on-prem log storage solution (e.g., an ELK stack or a simple file share).
- Operators have a documented procedure for accessing and searching these local logs during an outage.

## Feature 6: Data Governance, Security, and Identity Management

**Description:** This feature ensures that the entire system is secure, compliant, and well-governed, with proper access controls and monitoring in place across both and on-premises environments. It implements a Zero Trust approach wherever possible.

### User Story:  Implement Least-Privilege Access with PIM-Managed Groups

**Description:** As a Security Administrator, I want to use Entra ID Privileged Identity Management (PIM) for Groups to provide just-in-time (JIT) access to privileged roles in Azure, so that we minimize standing permissions and reduce our security exposure.

**Acceptance Criteria:**
- Entra ID groups are created for different privilege levels (e.g., `Fabric-Admins`, `ADLS-Contributors`).
- These groups are enabled in PIM, requiring users to request and justify temporary access.
- Access reviews are scheduled quarterly to recertify the need for membership in these groups.

### User Story:  Secure Data in ADLS with Private Endpoints and Firewall Rules

**Description:** As a Cloud Security Engineer, I want to secure our ADLS Gen2 account by disabling public access and using private endpoints and VNet service endpoints, so that data can only be accessed from trusted services and networks.

**Acceptance Criteria:**
- The ADLS account's public network access is disabled.
- A private endpoint for the ADLS account is created in the main VNet.
- Fabric is configured to access ADLS through the trusted workspace access mechanism.
- All access attempts are logged and can be audited.

### User Story:  Centralize Threat Detection with Microsoft Sentinel

**Description:** As a Security Administrator, I want to onboard the Acme-Sub subscription to the central ACME Microsoft Sentinel workspace and enable relevant data connectors, so that we have a single pane of glass for threat detection and incident response.

**Acceptance Criteria:**
- The Activity, Entra ID, and Microsoft Defender for Cloud data connectors are enabled for the Acme-Sub subscription.
- Logs are confirmed to be flowing into the Sentinel Log Analytics workspace.
- At least one custom alert rule is created based on a potential threat scenario relevant to this project (e.g., anomalous data egress from ADLS).

## Feature 7: Reporting and Analytics

**Description:** This feature focuses on creating Power BI reports and dashboards to provide insights to various stakeholders, from plant operators to executives, by combining OT data with business data. The goal is to move from simple operational reporting to strategic, data-driven decision-making.

### User Story:  Integrate EntERPSys_A Financial Data into Fabric Gold Layer

**Description:** As a BI Developer, I want to use the EntERPSys_A connector in Fabric to ingest financial data (e.g., cost of goods, maintenance costs) into the Gold layer of the Lakehouse, so it can be joined with our OT data.

**Acceptance Criteria:**
- A connection to the ACME tenant's EntERPSys_A instance is established in Fabric.
- A dataflow or pipeline is created to incrementally load relevant financial tables into the Gold layer.
- The data is available as Delta tables and can be queried.

### User Story:  Develop Executive KPI Dashboard in Power BI

**Description:** As a BI Developer, I want to create a Power BI dashboard for executives that marries OT data (e.g., equipment uptime, predicted failures) with financial data from EntERPSys_A to provide high-level business KPIs.

**Acceptance Criteria:**
- A Power BI semantic model is created in DirectLake mode over the Gold layer tables.
- The dashboard includes KPIs for OEE (Overall Equipment Effectiveness) with associated dollar values.
- A "what-if" analysis feature is included to estimate cost savings from preventing predicted failures.
- The report is published as a Power BI App and shared with the executive user group.

### User Story:  Create Power BI Report for Model Performance Monitoring

**Description:** As a Data Analyst, I want to create a Power BI report that connects to MLflow and our Lakehouse to visualize the ongoing performance of the production model, so we can detect data or concept drift.

**Acceptance Criteria:**
- The Power BI report connects to the Gold layer and the MLflow tracking server (via API or exported data).
- The report visualizes the distribution of input features over time to detect data drift.
- It compares model prediction distributions to actual outcomes to detect concept drift.
- The report is published and made available to the data science team.