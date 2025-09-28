# Product Backlog: ACME Predictive Maintenance Platform (Revised)

This document outlines the product backlog for the Acme-Sub future state predictive maintenance system. It has been revised to provide greater detail, break down larger user stories, and incorporate industry best practices.

## Feature 1: On-Premises Data Acquisition and Secure Transmission to Cloud

**Description:** This feature covers the setup and configuration of on-premises systems to collect equipment data from SCADA systems and securely transmit it to the cloud for processing. The architecture will ensure data is captured locally and can be buffered during internet outages, a critical requirement for business continuity. This involves establishing a Unified Namespace (UNS) on a local MQTT broker, which acts as a central, structured hub for all OT data.

### User Story: Install and Configure HiveMQ Broker for High Availability

**Description:** As a Data Engineer, I want to install and configure a HiveMQ MQTT broker cluster on our on-premises servers to ensure high availability and prevent a single point of failure for our local data pipeline.

**Acceptance Criteria:**
- A multi-node HiveMQ cluster is installed and running on the on-premises infrastructure at the pilot plant.
- The cluster is configured for automatic failover.
- Performance is benchmarked to handle the expected message throughput from all SCADA sources.
- Configuration is managed via Infrastructure as Code (IaC) and stored in the GitHub repository.

#### Task: Provision On-Premises Virtual Machines for HiveMQ Cluster
**Description:** Provision the necessary virtual machine resources on the on-premises server stack to host the HiveMQ cluster nodes.
- **Technical Details:** Provision three VMs with recommended specifications for HiveMQ (e.g., 4 vCPU, 8 GB RAM, 50 GB disk). Ensure all VMs are on the same network subnet to allow for cluster discovery. Configure static IP addresses for each node.
- **Expected Outcomes:** Three configured and running VMs ready for HiveMQ installation.
- **Definition of Done:** VMs are provisioned, networked, and accessible. OS is updated, and required Java runtime is installed.
- **Dependencies:** Access to on-prem virtualization platform (e.g., VMware).

#### Task: Install and Form HiveMQ Cluster
**Description:** Install the HiveMQ software on each provisioned VM and configure them to form a high-availability cluster.
- **Technical Details:** Download and install the HiveMQ platform on each VM. Modify the `config.xml` file on each node to enable clustering. Use a static discovery mechanism by listing the IP addresses of all cluster nodes. Start the nodes and verify that they form a cluster by checking the HiveMQ Control Center or logs.
- **Expected Outcomes:** A functioning multi-node HiveMQ cluster.
- **Definition of Done:** The cluster is formed and stable. All nodes report a successful connection to each other. The cluster status is green in the Control Center.
- **Dependencies:** Provisioned VMs.

#### Task: Configure Cluster Load Balancer
**Description:** Set up a load balancer in front of the HiveMQ cluster to distribute client connections evenly across all nodes.
- **Technical Details:** Configure a TCP load balancer (e.g., using NGINX, HAProxy, or a hardware load balancer) to route traffic to the HiveMQ nodes on the MQTT port (1883). Configure a health check to monitor the status of each node.
- **Expected Outcomes:** A single endpoint for clients to connect to the HiveMQ cluster.
- **Definition of Done:** The load balancer is configured and successfully routing traffic. Failover is tested by shutting down one HiveMQ node and verifying that clients remain connected.
- **Dependencies:** A running HiveMQ cluster.

### User Story: Define and Implement Unified Namespace (UNS) Topic Structure

**Description:** As an OT Architect, I want to design and implement a standardized, hierarchical topic structure (Unified Namespace) within HiveMQ, based on the ISA-95 standard, so that all OT data is organized, addressable, and context-rich.

**Acceptance Criteria:**
- A document defining the UNS topic hierarchy (e.g., `Enterprise/Site/Area/Line/Cell/Equipment/Tag`) is created and approved.
- The HiveMQ broker is configured to enforce this topic structure.
- Example data from a pilot line is published to the broker using the correct UNS topic format.

#### Task: Design the UNS Hierarchy
**Description:** Design the specific topic hierarchy for Acme-Sub based on the ISA-95 model and internal naming conventions.
- **Technical Details:** Collaborate with OT engineers and data scientists to define the levels of the hierarchy. Create a document that specifies the naming convention for each level (e.g., `ACME/Acme-Sub/Line1/Temp`). Include definitions for standard tags and metadata.
- **Expected Outcomes:** A finalized UNS design document.
- **Definition of Done:** The design document is reviewed and approved by all stakeholders. The document is checked into the project's GitHub repository.
- **Dependencies:** Input from OT and data teams.

### User Story: Secure HiveMQ Broker Communications

**Description:** As a Security Engineer, I want to secure the on-premises HiveMQ broker by enforcing encryption and strong authentication for all clients so that our OT data is protected from unauthorized access and tampering.

**Acceptance Criteria:**
- TLS encryption is enabled for all communication to and from the broker.
- All clients (e.g., Highbyte, model containers) are required to authenticate using client certificates (mTLS).
- Role-based access control is configured to restrict which clients can publish or subscribe to specific UNS topics.

#### Task: Generate and Deploy TLS Certificates
**Description:** Generate a Certificate Authority (CA) and issue TLS certificates for the HiveMQ broker and all connecting clients.
- **Technical Details:** Use a tool like OpenSSL to create a private CA. Generate a server certificate for the HiveMQ cluster and individual client certificates for each application that will connect (e.g., Highbyte). Configure the HiveMQ broker to use the server certificate and trust the private CA. Distribute client certificates securely.
- **Expected Outcomes:** A set of TLS certificates for the broker and clients.
- **Definition of Done:** Certificates are generated and securely stored. The HiveMQ broker is configured for TLS.
- **Dependencies:** None.

#### Task: Configure mTLS Authentication
**Description:** Configure the HiveMQ broker to require mutual TLS (mTLS), where both the server and the client must present a valid, trusted certificate to establish a connection.
- **Technical Details:** In the HiveMQ `config.xml`, configure the TLS listener to require client authentication (`<client-authentication-mode>REQUIRED</client-authentication-mode>`). Verify that clients without a valid certificate are rejected.
- **Expected Outcomes:** The broker rejects connections from clients without a trusted certificate.
- **Definition of Done:** A test client without a certificate fails to connect. A test client with a valid certificate connects successfully.
- **Dependencies:** TLS certificates generated and deployed.

### User Story: Configure Highbyte for Ignition Data Publishing

**Description:** As an OT Engineer, I want to configure Highbyte to connect to our Ignition SCADA system, model the relevant data points, and publish them to the HiveMQ broker under the correct UNS topic.

**Acceptance Criteria:**
- A secure connection is established between Highbyte and the Ignition SCADA system.
- Data models for the pilot line's Ignition tags are created in Highbyte.
- Highbyte is configured to publish data to the HiveMQ broker, mapping the data to the defined UNS structure.
- Data flow is validated from Ignition to HiveMQ.

#### Task: Create Highbyte Connection to Ignition
**Description:** Establish a connection from the Highbyte Intelligence Hub to the Ignition SCADA system.
- **Technical Details:** In the Highbyte UI, create a new data source connection. Select the appropriate connector for Ignition (likely OPC-UA or a similar standard). Provide the endpoint URL and credentials for the Ignition server.
- **Expected Outcomes:** A successful connection to the Ignition SCADA system.
- **Definition of Done:** The connection status in Highbyte shows as "Connected". Highbyte can browse the tag hierarchy in Ignition.
- **Dependencies:** Ignition SCADA system running and accessible on the network.

### User Story: Configure Highbyte for Rockwell Data Publishing

**Description:** As an OT Engineer, I want to configure Highbyte to connect to our Rockwell SCADA system, model the relevant data points, and publish them to the HiveMQ broker under the correct UNS topic.

**Acceptance Criteria:**
- A secure connection is established between Highbyte and the Rockwell SCADA system.
- Data models for the pilot line's Rockwell tags are created in Highbyte.
- Highbyte is configured to publish data to the HiveMQ broker, mapping the data to the defined UNS structure.
- Data flow is validated from Rockwell to HiveMQ.

#### Task: Create Highbyte Connection to Rockwell
**Description:** Establish a connection from the Highbyte Intelligence Hub to the Rockwell SCADA system.
- **Technical Details:** In the Highbyte UI, create a new data source connection for the Rockwell system, using the appropriate connector (e.g., EtherNet/IP). Provide the necessary connection details.
- **Expected Outcomes:** A successful connection to the Rockwell SCADA system.
- **Definition of Done:** The connection status in Highbyte shows as "Connected".
- **Dependencies:** Rockwell SCADA system running and accessible.

### User Story: Establish Secure Bridge from HiveMQ to Event Hubs

**Description:** As a Data Engineer, I want to configure the HiveMQ Enterprise Extension for to securely bridge data to our Event Hubs namespace, using managed identities for authentication to avoid managing secrets.

**Acceptance Criteria:**
- The HiveMQ Extension is installed and configured on the on-prem broker.
- The bridge uses a Managed Identity with least-privilege access to authenticate to Event Hubs.
- The connection uses TLS 1.2+ for end-to-end encryption.
- Data from specific UNS topics is successfully bridged to the target Event Hub.

#### Task: Configure HiveMQ Extension
**Description:** Install and configure the HiveMQ Enterprise Extension for Microsoft on the HiveMQ cluster.
- **Technical Details:** Download the extension from the HiveMQ Marketplace. Place the extension files in the `extensions` folder of each HiveMQ node. Create a configuration file (`azure-extension.xml`) specifying the Event Hubs namespace, authentication method (Managed Identity), and topic mappings.
- **Expected Outcomes:** The extension is loaded and running on the HiveMQ cluster.
- **Definition of Done:** The HiveMQ logs show that the extension has started successfully.
- **Dependencies:** A running HiveMQ cluster; an Event Hubs namespace.

### User Story: Implement and Test Data Buffering on HiveMQ Bridge

**Description:** As a Data Engineer, I want to configure and test the buffering mechanism on the HiveMQ to Event Hubs bridge to ensure no data is lost during a simulated internet outage.

**Acceptance Criteria:**
- The HiveMQ bridge is configured with a disk-based buffer of a specified size (e.g., capable of storing 24 hours of data).
- A test is conducted where the internet connection is severed for a period (e.g., 1 hour).
- Upon reconnection, the bridge successfully sends all buffered data to Event Hubs.
- No data loss is verified by comparing on-prem and cloud data counts.

#### Task: Configure Disk-Based Buffering
**Description:** Configure the HiveMQ Extension to use a persistent, disk-based buffer for outgoing messages.
- **Technical Details:** In the `azure-extension.xml` configuration, set the `mode` to `disk` for the message buffer. Specify the maximum size of the buffer on disk. Ensure the on-prem servers have sufficient disk space allocated.
- **Expected Outcomes:** The bridge will write messages to disk if it cannot connect to Event Hubs.
- **Definition of Done:** The configuration is applied and the broker is restarted. The buffer directory is created on the server.
- **Dependencies:** HiveMQ Extension is installed.

## Feature 2: Cloud Data Ingestion and Processing (Medallion Architecture)

**Description:** This feature covers the ingestion of streaming data from Event Hubs into Microsoft Fabric and processing it through a multi-layered Medallion Architecture (Bronze, Silver, Gold). This ensures data is progressively cleaned, conformed, and aggregated, making it reliable and ready for diverse use cases like analytics and ML.

### User Story: Provision and Configure Event Hubs for Scalability

**Description:** As a Cloud Engineer, I want to provision an Event Hubs namespace with a partition strategy that supports high-throughput and parallel consumption, so that our data ingestion pipeline is scalable and performant.

**Acceptance Criteria:**
- An Event Hubs namespace is provisioned using Infrastructure as Code (Bicep/Terraform).
- The Event Hub is configured with an appropriate number of partitions based on expected data volume.
- A dedicated consumer group is created for the Fabric streaming job.
- Private endpoint connectivity is enabled to restrict access to within the VNet.

#### Task: Write Bicep/Terraform for Event Hubs
**Description:** Author an Infrastructure as Code (IaC) template to define the Event Hubs namespace and its configuration.
- **Technical Details:** Create a Bicep or Terraform file that defines the `Microsoft.EventHub/namespaces` resource. Include parameters for SKU (e.g., Standard), capacity (Throughput Units), and partition count. Define a child resource for the Event Hub itself and a consumer group.
- **Expected Outcomes:** A reusable IaC template for deploying Event Hubs.
- **Definition of Done:** The template is complete and checked into the GitHub repository. It has been successfully deployed to a development environment.
- **Dependencies:** An subscription and resource group.

### User Story: Develop Bronze Ingestion Stream Processing Job

**Description:** As a Data Engineer, I want to develop a Fabric Spark streaming job that reads raw data from Event Hubs and writes it to the Bronze ADLS container as immutable Parquet files, partitioned by date.

**Acceptance Criteria:**
- The Spark job uses the Event Hubs connector to read the data stream.
- The job writes the raw event payload (including metadata) to ADLS Gen2 in Parquet format.
- Data is partitioned by year, month, and day for efficient querying.
- The job uses watermarking to handle late-arriving data and manages checkpoints for stream resiliency.

#### Task: Develop Spark Streaming Notebook
**Description:** Write a PySpark notebook in Microsoft Fabric to perform the stream ingestion.
- **Technical Details:** Use `spark.readStream.format("eventhubs")` to connect to the Event Hubs endpoint. Configure the connection with the necessary credentials (e.g., connection string stored in Key Vault). Use `writeStream.format("parquet")` to write the data, specifying the output path and partitioning columns.
- **Expected Outcomes:** A Fabric notebook that can read from Event Hubs and write to ADLS.
- **Definition of Done:** The notebook runs without errors and writes data to the specified ADLS location.
- **Dependencies:** Event Hubs provisioned and receiving data.

### User Story: Implement a Dead-Letter Queue for Ingestion Errors

**Description:** As a Data Engineer, I want to implement a dead-letter mechanism for the Bronze ingestion job, so that any malformed or un-processable messages are captured for later analysis without halting the entire stream.

**Acceptance Criteria:**
- The Spark streaming job includes a `try/except` block to catch parsing errors.
- Messages that fail to process are written to a separate "dead-letter" directory in ADLS.
- An alert is configured to notify the data engineering team when new messages land in the dead-letter directory.

#### Task: Add Error Handling to Spark Job
**Description:** Modify the Spark streaming notebook to include error handling logic.
- **Technical Details:** Wrap the data processing logic in a `try/except` block. In the `except` block, log the error and write the original message data to a separate ADLS path (e.g., `/bronze/dead-letter/`).
- **Expected Outcomes:** The streaming job will not fail on bad records.
- **Definition of Done:** When a malformed message is sent to Event Hubs, the job continues to run, and the bad message is saved to the dead-letter location.
- **Dependencies:** A working Bronze ingestion job.

### User Story: Develop Silver Layer Cleansing and Conforming Job

**Description:** As a Data Engineer, I want to develop a batch or streaming job that processes raw Bronze data, applies data quality rules, and conforms it into a well-defined schema in the Silver layer of the Fabric Lakehouse.

**Acceptance Criteria:**
- The job reads new data from the Bronze layer.
- Data is deduplicated based on a unique message identifier.
- The UNS topic is parsed into structured columns (Site, Area, Line, etc.).
- Data types are enforced, and null values are handled according to defined business rules.
- The cleaned and conformed data is written to Delta tables in the Silver layer.

#### Task: Create Silver Processing Notebook
**Description:** Develop a Fabric notebook to read from the Bronze Parquet files and write to Silver Delta tables.
- **Technical Details:** Use Spark SQL or DataFrame operations to read the Bronze data. Apply transformations such as `split()` to parse the topic string and `withColumn()` to cast data types. Use `dropDuplicates()` to handle duplicate messages. Write the transformed DataFrame to a Delta table using `.format("delta")`.
- **Expected Outcomes:** A notebook that transforms Bronze data into a clean, structured format.
- **Definition of Done:** The notebook runs successfully. The Silver Delta table is created and populated with clean data. The table schema matches the defined specification.
- **Dependencies:** Data present in the Bronze layer.

### User Story: Develop Gold Layer Aggregation Job for Analytics

**Description:** As a BI Developer, I want to create a data pipeline that aggregates Silver layer data into business-centric models (e.g., hourly/daily summaries of sensor readings) in the Gold layer, optimized for Power BI reporting.

**Acceptance Criteria:**
- A Fabric pipeline or notebook reads from Silver layer Delta tables.
- Data is aggregated into key metrics (e.g., averages, min/max values) over specific time windows.
- The aggregated data is stored in Gold layer Delta tables with a user-friendly schema.
- The resulting tables are performant for DirectQuery from Power BI.

#### Task: Create Gold Aggregation Notebook
**Description:** Develop a Fabric notebook to create aggregated tables for BI reporting.
- **Technical Details:** Read data from the Silver Delta tables. Use `groupBy()` and windowing functions (`window()`) in Spark to aggregate data over time (e.g., hourly, daily). Select and rename columns to create a user-friendly, denormalized table.
- **Expected Outcomes:** A notebook that produces aggregated Gold tables.
- **Definition of Done:** The notebook runs successfully. The Gold tables are created and contain the correct aggregated data.
- **Dependencies:** Clean data in the Silver layer.

## Feature 3: Predictive Maintenance Model Development and Training

**Description:** This feature encompasses the process of developing, training, and validating the predictive maintenance ML model within the Microsoft Fabric environment. This includes an initial port of the vendor's model, followed by in-house iteration and improvement using MLOps best practices.

### User Story: Perform Exploratory Data Analysis (EDA) on Gold Layer Data

**Description:** As a Data Scientist, I want to perform a thorough exploratory data analysis on the curated Gold layer data to understand data distributions, identify correlations, and validate assumptions before model development.

**Acceptance Criteria:**
- A Fabric notebook is created for EDA.
- Visualizations (histograms, scatter plots, etc.) are generated to understand key features.
- A summary report of findings, including data quality issues or interesting patterns, is produced.

#### Task: Create EDA Notebook
**Description:** Develop a Fabric notebook to load Gold layer data and perform exploratory analysis.
- **Technical Details:** Load data from a Gold Delta table into a Spark DataFrame. Convert to a Pandas DataFrame for use with libraries like Matplotlib and Seaborn. Generate plots to visualize feature distributions, correlations, and relationships with the target variable.
- **Expected Outcomes:** A notebook with detailed data analysis and visualizations.
- **Definition of Done:** The notebook is complete and has been reviewed by the data science team. Key findings are documented.
- **Dependencies:** Aggregated data in the Gold layer.

### User Story: Translate and Adapt ModelQ's Feature Engineering to Fabric

**Description:** As a Data Scientist, I want to replicate the feature engineering logic from ModelQ's model in a Fabric notebook, using the Gold layer data as the source, so we can own and understand the data preparation process.

**Acceptance Criteria:**
- A Fabric notebook is created that contains the PySpark code for all feature engineering steps.
- The output of the notebook is a feature set that is structurally identical to the one used by the vendor.
- The notebook is parameterized and can be run as part of a larger pipeline.

#### Task: Implement Feature Engineering Logic
**Description:** Write the PySpark code in a Fabric notebook to implement the feature engineering transformations.
- **Technical Details:** Based on the logic from ModelQ, implement transformations such as creating lag features, rolling averages, and other domain-specific features. Use Spark UDFs if necessary for complex transformations. The final output should be a single DataFrame ready for model training.
- **Expected Outcomes:** A feature engineering notebook.
- **Definition of Done:** The notebook runs and produces the expected feature set. The code is commented and follows project coding standards.
- **Dependencies:** EDA is complete; logic from ModelQ is available.

### User Story: Integrate Fabric Notebooks with MLflow for Automated Logging

**Description:** As a Data Scientist, I want to integrate our training notebooks with the MLflow tracking service in Fabric to automatically log all relevant experiment information, so we have a reproducible and auditable history of our model development.

**Acceptance Criteria:**
- The training notebook uses `mlflow.autolog()` or manual logging calls.
- For each experiment run, MLflow captures the Git commit hash, parameters, performance metrics, and the model artifact itself.
- Experiment results are visible and comparable in the Fabric UI.

#### Task: Add MLflow Logging to Training Notebook
**Description:** Modify the model training notebook to include MLflow logging.
- **Technical Details:** Add `import mlflow` to the notebook. Use `mlflow.start_run()` to begin a new run. Log parameters using `mlflow.log_param()`, metrics with `mlflow.log_metric()`, and the trained model with `mlflow.spark.log_model()`.
- **Expected Outcomes:** The training notebook will log all experiment details to MLflow.
- **Definition of Done:** After running the notebook, a new experiment run appears in the MLflow UI in Fabric with all the specified information.
- **Dependencies:** A model training notebook.

### User Story: Tune Hyperparameters and Train Final Model

**Description:** As a Data Scientist, I want to use a systematic approach like grid search or Bayesian optimization to tune the model's hyperparameters and train the final model on the full feature set, to ensure we achieve the best possible performance.

**Acceptance Criteria:**
- A notebook is created to perform hyperparameter tuning (e.g., using Hyperopt).
- The results of the tuning process are tracked in MLflow.
- The model with the optimal hyperparameters is re-trained on the entire training dataset.

#### Task: Implement Hyperparameter Tuning
**Description:** Develop a notebook to automate the process of hyperparameter tuning.
- **Technical Details:** Use a library like Hyperopt, integrated with Spark via `SparkTrials`, to distribute the tuning process. Define the search space for the model's hyperparameters. Log the results of each trial to MLflow.
- **Expected Outcomes:** A notebook that systematically finds the best hyperparameters.
- **Definition of Done:** The tuning job completes. The best set of hyperparameters is identified and logged.
- **Dependencies:** Feature engineering notebook is complete.

### User Story: Validate and Register Production-Candidate Model in MLflow

**Description:** As a Data Scientist, I want to validate the final trained model against a hold-out test set and, if it meets performance criteria, register it in the MLflow Model Registry and transition its stage to "Staging" or "Production".

**Acceptance Criteria:**
- The model's performance is evaluated on an unseen test dataset.
- The performance meets or exceeds the pre-defined business requirements (e.g., >95% precision).
- The model is saved to MLflow and registered in the Model Registry.
- The model version is transitioned to the "Staging" stage, triggering the CI/CD pipeline.

#### Task: Create Model Validation and Registration Notebook
**Description:** Develop a notebook that loads the trained model, evaluates it on a test set, and registers it in the MLflow Model Registry.
- **Technical Details:** Load the model from an MLflow run. Score it against a previously unseen test dataset. Calculate performance metrics. If the metrics meet the required threshold, use `mlflow.register_model()` to create a new version in the registry.
- **Expected Outcomes:** A new, validated model version in the MLflow Model Registry.
- **Definition of Done:** The notebook runs successfully. A new model version is created in the registry and is visible in the Fabric UI.
- **Dependencies:** A trained model from the previous step.

## Feature 4: MLOps and CI/CD Pipeline for Model Deployment

**Description:** This feature focuses on creating a robust, automated CI/CD pipeline using GitHub Actions. This pipeline will be responsible for containerizing the trained ML model, pushing it to a secure registry, and triggering the deployment to the on-premises Kubernetes cluster, following modern MLOps principles.

### User Story: Develop a Production-Ready Inference Script

**Description:** As an MLOps Engineer, I want to develop a lightweight Python web service (e.g., using Flask or FastAPI) that loads the trained model and exposes a REST API endpoint for inference, so the model can be easily consumed within a container.

**Acceptance Criteria:**
- A Python script for the web service is created.
- The script includes a `/predict` endpoint that accepts input data and returns a model prediction.
- The script includes a `/health` endpoint for health checks.
- The script is stored in the model's GitHub repository.

#### Task: Write FastAPI Inference Service
**Description:** Create a Python script using FastAPI to serve the ML model.
- **Technical Details:** Use FastAPI to create the web server. Implement a `/health` endpoint that returns a 200 OK status. Implement a `/predict` endpoint that accepts a JSON payload, uses the loaded MLflow model to make a prediction, and returns the result as JSON.
- **Expected Outcomes:** A Python script for the inference service.
- **Definition of Done:** The script runs locally and responds to requests on both endpoints correctly.
- **Dependencies:** A trained model in a format that can be loaded (e.g., MLflow model format).

### User Story: Author a Dockerfile for Model Containerization

**Description:** As an MLOps Engineer, I want to author a multi-stage Dockerfile that efficiently packages the inference script, the trained model, and all necessary dependencies into a minimal, secure container image.

**Acceptance Criteria:**
- A `Dockerfile` is created in the GitHub repository.
- The Dockerfile uses a specific, versioned base image (e.g., `python:3.9-slim`).
- The resulting Docker image is small and free of unnecessary build dependencies.
- The image can be built and run successfully on a local machine.

#### Task: Create Multi-Stage Dockerfile
**Description:** Write a multi-stage Dockerfile to create an optimized production image.
- **Technical Details:** Use a first stage (e.g., `python:3.9`) to install dependencies from `requirements.txt`. In a second stage, copy the installed dependencies and the application code into a minimal base image (e.g., `python:3.9-slim`). Set the `CMD` to run the FastAPI server.
- **Expected Outcomes:** A `Dockerfile` in the project repository.
- **Definition of Done:** The `docker build` command completes successfully. The resulting image is smaller than a single-stage build would be.
- **Dependencies:** The FastAPI inference script and a `requirements.txt` file.

### User Story: Provision Container Registry (ACR) for Docker Images

**Description:** As a Cloud Engineer, I want to provision a secure Container Registry (ACR) to store our production model container images, so that we have a private, managed registry for our deployments.

**Acceptance Criteria:**
- An ACR instance is provisioned in using IaC.
- The registry is configured with private access, accessible only from our CI/CD agents and the on-prem Kubernetes cluster.
- A repository is created for the predictive maintenance model images.

#### Task: Write Bicep/Terraform for ACR
**Description:** Author an IaC template to define the Container Registry.
- **Technical Details:** Create a Bicep or Terraform file that defines the `Microsoft.ContainerRegistry/registries` resource. Set the SKU to `Premium` to allow for private endpoints. Configure network rules to restrict access.
- **Expected Outcomes:** A reusable IaC template for deploying ACR.
- **Definition of Done:** The template is checked into GitHub and successfully deployed.
- **Dependencies:** An subscription and resource group.

### User Story: Implement CI Pipeline to Build and Push Model Container

**Description:** As an MLOps Engineer, I want to create a GitHub Actions CI workflow that automatically builds, tags, and pushes the model's Docker image to ACR whenever a new model is tagged for release in the Git repository.

**Acceptance Criteria:**
- A GitHub Actions workflow file is created.
- The workflow is triggered by a new Git tag (e.g., `v1.2.0`).
- The workflow authenticates to ACR using a service principal or managed identity.
- The Docker image is built and pushed to ACR with a tag corresponding to the Git tag.

#### Task: Create GitHub Actions CI Workflow
**Description:** Create the YAML file for the GitHub Actions workflow.
- **Technical Details:** Create a `.github/workflows/ci.yml` file. Use the `on: push: tags:` trigger. Use the `azure/login` action to authenticate to Azure. Use the `docker/build-push-action` to build the image and push it to ACR.
- **Expected Outcomes:** A CI workflow file in the repository.
- **Definition of Done:** When a new tag is pushed to GitHub, the workflow runs successfully and the new Docker image appears in ACR.
- **Dependencies:** An ACR instance; a Dockerfile.

### User Story: Implement CD Pipeline to Deploy Model via IoT Edge

**Description:** As an MLOps Engineer, I want to extend the GitHub Actions workflow to act as a CD pipeline that, after a successful image push, updates the IoT Edge deployment manifest to roll out the new container version to the on-prem cluster.

**Acceptance Criteria:**
- The workflow retrieves the IoT Edge deployment manifest from the repository.
- It updates the image version for the model module in the manifest.
- It uses the CLI or a dedicated GitHub Action to apply the updated manifest to the target IoT Hub device/cluster.
- The pipeline includes a manual approval step before deploying to production.

#### Task: Create GitHub Actions CD Workflow
**Description:** Add a deployment job to the GitHub Actions workflow.
- **Technical Details:** Add a new job to `ci.yml` that depends on the build job. Use a tool like `sed` or `yq` to update the image tag in the `deployment.template.json` file. Use the `azure/iotedge-deploy` action or the CLI (`az iot edge set-modules`) to apply the manifest.
- **Expected Outcomes:** A CD workflow that automates deployments.
- **Definition of Done:** When the workflow runs, the new module version is deployed to the IoT Edge device, which can be verified in the portal.
- **Dependencies:** A CI workflow; an IoT Edge device registered in IoT Hub.

## Feature 5: On-Premises Model Deployment and Inference

**Description:** This feature covers the setup of the on-premises infrastructure to host the ML model and run inference locally. This is the most critical component for ensuring the plant can continue to operate and benefit from model predictions even during a complete internet outage.

### User Story: Install and Configure a Production-Grade Kubernetes (K8s) Cluster

**Description:** As an Infrastructure Engineer, I want to install and configure a production-grade Kubernetes cluster on our on-prem servers, so we have a resilient and scalable platform for running our containerized ML model.

**Acceptance Criteria:**
- A multi-node K8s cluster is installed and configured.
- The cluster has role-based access control (RBAC) enabled.
- A private container registry (ACR) is configured as a trusted source for images.

#### Task: Install Kubernetes on On-Prem Servers
**Description:** Install a Kubernetes distribution on the provisioned on-prem servers.
- **Technical Details:** Choose a K8s distribution suitable for on-premises (e.g., K3s, RKE2, or kubeadm). Follow the official documentation to install the control plane on one node and join the other nodes as workers. Verify cluster status with `kubectl get nodes`.
- **Expected Outcomes:** A running Kubernetes cluster.
- **Definition of Done:** All nodes are in the `Ready` state. `kubectl` commands can be run against the cluster.
- **Dependencies:** Provisioned on-prem servers.

### User Story: Configure K8s Network Policies for Secure Broker Communication

**Description:** As a Security Engineer, I want to implement Kubernetes Network Policies to strictly control traffic between the ML model pod and the HiveMQ broker, ensuring the model can only communicate with the intended service.

**Acceptance Criteria:**
- A Network Policy is defined that allows egress traffic from the model pod only to the HiveMQ broker's IP and port.
- A Network Policy is defined that allows ingress traffic to the model pod only from trusted sources if it exposes an API.
- All other ingress and egress traffic from the model pod is denied by default.

#### Task: Define and Apply Network Policies
**Description:** Create YAML files for the Kubernetes Network Policies and apply them to the cluster.
- **Technical Details:** Create a `NetworkPolicy` manifest. Define a `podSelector` to target the ML model pods. Create an `egress` rule that specifies the IP address and port of the HiveMQ broker. Apply the policy using `kubectl apply -f <policy-file>.yaml`.
- **Expected Outcomes:** A Network Policy resource created in the cluster.
- **Definition of Done:** Test connectivity from the model pod to an unauthorized endpoint (should fail). Test connectivity to the HiveMQ broker (should succeed).
- **Dependencies:** A running K8s cluster with a CNI that supports Network Policies (e.g., Calico).

### User Story: Install and Register IoT Edge on the K8s Cluster

**Description:** As an Infrastructure Engineer, I want to install the IoT Edge runtime on our on-prem Kubernetes cluster and register it with our IoT Hub, so that it can be managed and receive deployment instructions from the cloud.

**Acceptance Criteria:**
- The IoT Edge daemonset is successfully deployed to the K8s cluster.
- The edge device is successfully registered in IoT Hub and shows a connected state.
- The `iotedge` command-line tool can be used to check the status of the runtime and modules.

#### Task: Deploy IoT Edge to Kubernetes
**Description:** Deploy the IoT Edge runtime to the on-prem Kubernetes cluster using Helm or YAML manifests.
- **Technical Details:** Add the Microsoft Helm chart repository. Use Helm to install the `iotedge` chart, providing the device connection string from IoT Hub as a parameter. Verify that the `edgeAgent` and `edgeHub` pods are created and running.
- **Expected Outcomes:** IoT Edge runtime installed on the cluster.
- **Definition of Done:** The `edgeAgent` and `edgeHub` pods are in the `Running` state. The device appears as "Connected" in the IoT Hub portal.
- **Dependencies:** A running K8s cluster; an IoT Hub instance.

### User Story: Deploy and Validate the ML Model Container on the Edge

**Description:** As an MLOps Engineer, I want to perform the first deployment of the ML model container to the on-prem cluster via the CD pipeline and run a smoke test to validate that it's running correctly.

**Acceptance Criteria:**
- The CD pipeline successfully deploys the model container to the K8s cluster.
- The model pod starts and is in a `Running` state.
- The pod logs show a successful connection to the HiveMQ broker.
- A test message sent to the input topic results in a valid prediction on the output topic.

#### Task: Run Initial Deployment Pipeline
**Description:** Trigger the CD pipeline to deploy the model to the on-prem cluster for the first time.
- **Technical Details:** Manually trigger the GitHub Actions CD workflow. Monitor the pipeline execution to ensure all steps pass. After the pipeline completes, use `kubectl get pods` to verify that the model pod has been created.
- **Expected Outcomes:** The model container is deployed and running on the K8s cluster.
- **Definition of Done:** The pod is running. A `kubectl logs` command on the pod shows no startup errors.
- **Dependencies:** A completed CD pipeline; a running IoT Edge on K8s setup.

### User Story: Implement Local Logging and Monitoring for the Deployed Model

**Description:** As an OT Engineer, I want to ensure that logs from the running model container are collected and available locally on the on-prem network, so that we can monitor the model's health and troubleshoot issues even if the cloud connection is down.

**Acceptance Criteria:**
- The Kubernetes cluster is configured to collect logs from all pods (e.g., using Fluentd or a similar log aggregator).
- Logs are forwarded to a local, on-prem log storage solution (e.g., an ELK stack or a simple file share).
- Operators have a documented procedure for accessing and searching these local logs during an outage.

#### Task: Deploy a Local Log Aggregator
**Description:** Deploy a log aggregation tool like Fluentd as a DaemonSet on the Kubernetes cluster.
- **Technical Details:** Use a Helm chart to deploy Fluentd to the cluster. Configure Fluentd to collect logs from all containers. Configure the output plugin to forward logs to an on-prem Elasticsearch instance or a local file system.
- **Expected Outcomes:** A log aggregation pipeline running on the cluster.
- **Definition of Done:** Fluentd is running on all nodes. Logs from the model pod are appearing in the configured local storage location.
- **Dependencies:** A running K8s cluster.

## Feature 6: Data Governance, Security, and Identity Management

**Description:** This feature ensures that the entire system is secure, compliant, and well-governed, with proper access controls and monitoring in place across both and on-premises environments. It implements a Zero Trust approach wherever possible.

### User Story: Implement Least-Privilege Access with PIM-Managed Groups

**Description:** As a Security Administrator, I want to use Entra ID Privileged Identity Management (PIM) for Groups to provide just-in-time (JIT) access to privileged roles in Azure, so that we minimize standing permissions and reduce our security exposure.

**Acceptance Criteria:**
- Entra ID groups are created for different privilege levels (e.g., `Fabric-Admins`, `ADLS-Contributors`).
- These groups are enabled in PIM, requiring users to request and justify temporary access.
- Access reviews are scheduled quarterly to recertify the need for membership in these groups.

#### Task: Configure PIM for Groups
**Description:** Onboard the necessary Entra ID groups to PIM and configure the access settings.
- **Technical Details:** In the Entra ID portal, navigate to PIM -> Groups. Select the target groups and bring them under PIM management. Configure the role settings to require approval for activation and set a maximum duration.
- **Expected Outcomes:** Privileged access to resources will be JIT.
- **Definition of Done:** A user successfully requests, is granted, and uses temporary access to a privileged role.
- **Dependencies:** Entra ID P2 license.

### User Story: Secure Data in ADLS with Private Endpoints and Firewall Rules

**Description:** As a Cloud Security Engineer, I want to secure our ADLS Gen2 account by disabling public access and using private endpoints and VNet service endpoints, so that data can only be accessed from trusted services and networks.

**Acceptance Criteria:**
- The ADLS account's public network access is disabled.
- A private endpoint for the ADLS account is created in the main VNet.
- Fabric is configured to access ADLS through the trusted workspace access mechanism.
- All access attempts are logged and can be audited.

#### Task: Configure ADLS Networking
**Description:** Modify the ADLS account's networking settings to disable public access and create a private endpoint.
- **Technical Details:** In the ADLS networking blade, set "Public network access" to "Disabled". Create a new private endpoint, connecting it to the ADLS `dfs` sub-resource and placing it in the appropriate VNet/subnet.
- **Expected Outcomes:** The ADLS account is no longer accessible from the public internet.
- **Definition of Done:** An attempt to access the ADLS account from a public IP address fails. A VM within the VNet can access the account via the private endpoint.
- **Dependencies:** An ADLS account and a VNet.

### User Story: Centralize Threat Detection with Microsoft Sentinel

**Description:** As a Security Administrator, I want to onboard the Acme-Sub subscription to the central ACME Microsoft Sentinel workspace and enable relevant data connectors, so that we have a single pane of glass for threat detection and incident response.

**Acceptance Criteria:**
- The Activity, Entra ID, and Microsoft Defender for Cloud data connectors are enabled for the Acme-Sub subscription.
- Logs are confirmed to be flowing into the Sentinel Log Analytics workspace.
- At least one custom alert rule is created based on a potential threat scenario relevant to this project (e.g., anomalous data egress from ADLS).

#### Task: Configure Sentinel Data Connectors
**Description:** Connect the Acme-Sub subscription's log sources to the ACME Sentinel workspace.
- **Technical Details:** From the ACME Sentinel workspace, navigate to Data Connectors. Open the connector page for Activity Log, Entra ID, etc. Follow the instructions to configure the connection to the Acme-Sub tenant and subscription.
- **Expected Outcomes:** Logs from the Acme-Sub tenant are ingested into the ACME Sentinel.
- **Definition of Done:** Data is visible in the connector dashboards and can be queried in Log Analytics.
- **Dependencies:** Appropriate cross-tenant permissions (Lighthouse).

## Feature 7: Reporting and Analytics

**Description:** This feature focuses on creating Power BI reports and dashboards to provide insights to various stakeholders, from plant operators to executives, by combining OT data with business data. The goal is to move from simple operational reporting to strategic, data-driven decision-making.

### User Story: Integrate EntERPSys_A Financial Data into Fabric Gold Layer

**Description:** As a BI Developer, I want to use the EntERPSys_A connector in Fabric to ingest financial data (e.g., cost of goods, maintenance costs) into the Gold layer of the Lakehouse, so it can be joined with our OT data.

**Acceptance Criteria:**
- A connection to the ACME tenant's EntERPSys_A EntERPSys_A instance is established in Fabric.
- A dataflow or pipeline is created to incrementally load relevant financial tables into the Gold layer.
- The data is available as Delta tables and can be queried.

#### Task: Create Fabric Dataflow for EntERPSys_A
**Description:** Use a Dataflow Gen2 in Fabric to connect to and ingest data from EntERPSys_A.
- **Technical Details:** Create a new Dataflow Gen2. Use the built-in EntERPSys_A connector. Authenticate using an organizational account with permissions to the ACME tenant. Select the required tables and configure incremental refresh if possible.
- **Expected Outcomes:** A dataflow that copies EntERPSys_A data to the Gold layer.
- **Definition of Done:** The dataflow runs successfully and the target Delta tables in the Gold layer are populated.
- **Dependencies:** Access credentials for the EntERPSys_A EntERPSys_A environment.

### User Story: Develop Executive KPI Dashboard in Power BI

**Description:** As a BI Developer, I want to create a Power BI dashboard for executives that marries OT data (e.g., equipment uptime, predicted failures) with financial data from EntERPSys_A to provide high-level business KPIs.

**Acceptance Criteria:**
- A Power BI semantic model is created in DirectLake mode over the Gold layer tables.
- The dashboard includes KPIs for OEE (Overall Equipment Effectiveness) with associated dollar values.
- A "what-if" analysis feature is included to estimate cost savings from preventing predicted failures.
- The report is published as a Power BI App and shared with the executive user group.

#### Task: Create Power BI Semantic Model
**Description:** Create a new Power BI semantic model in the Fabric workspace that connects to the Gold layer tables.
- **Technical Details:** Create a new semantic model. Add the Gold layer Delta tables as sources. Use the modeling view to define relationships between the OT and financial data tables. Create DAX measures for key KPIs.
- **Expected Outcomes:** A well-structured Power BI semantic model.
- **Definition of Done:** The model is created, relationships are correct, and key measures are defined.
- **Dependencies:** Gold layer tables are available.

### User Story: Create Power BI Report for Model Performance Monitoring

**Description:** As a Data Analyst, I want to create a Power BI report that connects to MLflow and our Lakehouse to visualize the ongoing performance of the production model, so we can detect data or concept drift.

**Acceptance Criteria:**
- The Power BI report connects to the Gold layer and the MLflow tracking server (via API or exported data).
- The report visualizes the distribution of input features over time to detect data drift.
- It compares model prediction distributions to actual outcomes to detect concept drift.
- The report is published and made available to the data science team.

#### Task: Connect Power BI to MLflow
**Description:** Develop a method to get MLflow experiment data into Power BI.
- **Technical Details:** Write a Fabric notebook to periodically query the MLflow API (using `mlflow.search_runs()`), extract relevant metrics and parameters into a DataFrame, and save it as a Delta table in the Gold layer. This table can then be easily imported into Power BI.
- **Expected Outcomes:** A table in the Gold layer containing MLflow experiment data.
- **Definition of Done:** The notebook runs and successfully populates the MLflow data table.
- **Dependencies:** MLflow experiments have been run.