# Product Backlog: JIMO-2 Autonomous Science

This document outlines the product backlog for the JIMO-2 future state autonomous science system. It has been revised to provide greater detail, break down larger user stories, and incorporate flight mission best practices.

## Feature 1: Flight Segment Data Acquisition and Secure Downlink

**Description:** This feature covers the setup and configuration of on-board systems to collect instrument data and securely transmit it to the ground for processing. The architecture will ensure data is captured on-board and can be buffered during communication blackouts, a critical requirement for mission continuity. This involves establishing a standardized data format on the on-board bus, which acts as a central, structured hub for all instrument and telemetry data.

### User Story: Implement and Configure On-Board Data Bus (SpaceWire) for High Availability

**Description:** As a Flight Software (FSW) Engineer, I want to implement and configure the SpaceWire data bus service on our redundant flight computers to ensure high availability and prevent a single point of failure for our on-board data pipeline.

**Acceptance Criteria:**
- Redundant SpaceWire interfaces (Side A / Side B) are operational on the flight computers.
- The C&DH (Command & Data Handling) FSW is configured for automatic failover.
- Performance is benchmarked to handle the expected data throughput from all instruments.
- Configuration is managed via the FSW build process and stored in the GitHub repository.

#### Task: Activate and Test Redundant RAD750 Flight Computers
**Description:** Provision and test the redundant RAD750 flight computers (Side A and Side B) to host the FSW services.
- **Technical Details:** Power on both RAD750 computers. Ensure both are running the bootloader and base FSW. Configure a watchdog timer and heartbeat mechanism between the two for failover.
- **Expected Outcomes:** Two configured and running flight computers ready for the C&DH FSW load.
- **Definition of Done:** Both computers are provisioned, networked via their cross-links, and accessible. The base OS and C&DH services are installed.
- **Dependencies:** Access to the "flatsat" or "iron bird" testbed.

#### Task: Deploy and Link SpaceWire FSW Service
**Description:** Install the SpaceWire FSW driver on each flight computer and configure them to manage the data bus.
- **Technical Details:** Deploy the FSW module for the SpaceWire driver. Modify the FSW configuration tables to initialize the bus and listen for instrument packets. Verify that both RAD750s can see the bus.
- **Expected Outcomes:** A functioning, redundant SpaceWire bus service.
- **Definition of Done:** The bus is formed and stable. Both computers report a successful connection to the bus. The bus status is "nominal" in telemetry.
- **Dependencies:** Provisioned flight computers.

#### Task: Configure C&DH for Bus Failover
**Description:** Set up a failover mechanism in the C&DH service to automatically switch control of the bus from Side A to Side B.
- **Technical Details:** Configure a health check to monitor the status of the primary bus controller. In the event of a failure (e.g., no heartbeat), the redundant computer takes control of the bus.
- **Expected Outcomes:** A single, virtual endpoint for instruments to publish data.
- **Definition of Done:** The failover is configured and successfully tested. A test is run by powering down the primary computer and verifying that instruments remain connected.
- **Dependencies:** A running SpaceWire FSW service.

### User Story: Define and Implement CCSDS Packet Topic Structure

**Description:** As a Systems Engineer, I want to design and implement a standardized, hierarchical packet structure (based on the CCSDS standard) within the FSW, so that all telemetry data is organized, addressable, and context-rich.

**Acceptance Criteria:**
- A document defining the telemetry packet hierarchy (e.g., `Mission/Spacecraft/Instrument/PacketID/Measurement`) is created and approved.
- The C&DH FSW is configured to route packets based on this structure.
- Example data from a pilot instrument (e.g., Radar) is published to the bus using the correct packet format.

#### Task: Design the Telemetry Packet Hierarchy
**Description:** Design the specific packet Application ID (APID) hierarchy for JIMO-2 based on the CCSDS model and mission conventions.
- **Technical Details:** Collaborate with instrument and FSW engineers to define the APIDs. Create a document that specifies the naming convention for each packet. Include definitions for standard telemetry and metadata.
- **Expected Outcomes:** A finalized Interface Control Document (ICD) for telemetry.
- **Definition of Done:** The ICD is reviewed and approved by all stakeholders. The document is checked into the project's GitHub repository.
- **Dependencies:** Input from instrument and science teams.

### User Story: Secure On-Board Data Bus Communications

**Description:** As an FSW Assurance Engineer, I want to secure the on-board data bus by enforcing checksums and authenticated commands for all modules so that our instrument data is protected from corruption or unauthorized commanding.

**Acceptance Criteria:**
- CRC or other checksums are enabled for all data packets on the bus.
- All critical command packets (e.g., FSW module loading) are required to be authenticated.
- Access control is configured to restrict which modules can publish or subscribe to specific APIDs.

#### Task: Implement Bus-Level CRC and Authenticated Packets
**Description:** Generate a Certificate Authority (CA) and issue TLS certificates for the HiveMQ broker and all connecting clients.
- **Technical Details:** Modify the FSW C&DH service to append and validate CRC checksums on all packets. Implement a command authentication mechanism (e.g., based on a shared secret or sequence counter) for critical commands.
- **Expected Outcomes:** A set of FSW modules for data validation.
- **Definition of Done:** Packets are generated and validated. The C&DH service is configured to reject invalid packets.
- **Dependencies:** None.

#### Task: Configure FSW Software Bus (CFS-SB) Packet Filters
**Description:** Configure the Core Flight Software (CFS) Software Bus (SB) to require validation for all packets and commands.
- **Technical Details:** In the CFS-SB configuration tables, configure the APID filters to restrict which FSW modules can send or receive specific packets. Verify that modules without valid permissions are rejected.
- **Expected Outcomes:** The bus rejects commands from unauthorized FSW modules.
- **Definition of Done:** A test module without permission fails to publish. A test module with valid permission succeeds.
- **Dependencies:** Checksums and authentication implemented.

### User Story: Configure Radar Interface for Data Publishing

**Description:** As an Instrument Engineer, I want to configure the Ice-Penetrating Radar to connect to the C&DH, model the relevant data points, and publish them to the SpaceWire bus under the correct CCSDS packet definition.

**Acceptance Criteria:**
- A secure connection is established between the radar and the C&DH.
- Data models for the pilot radar tags are created in the FSW.
- The radar is configured to publish data to the SpaceWire bus, mapping the data to the defined packet structure.
- Data flow is validated from the radar to the C&DH.

#### Task: Create C&DH Connection to Radar
**Description:** Establish a connection from the C&DH service to the Ice-Penetrating Radar.
- **Technical Details:** In the FSW, create a new data source connection. Select the appropriate driver for the radar (likely SpaceWire). Provide the endpoint URL and credentials for the instrument.
- **Expected Outcomes:** A successful connection to the radar.
- **Definition of Done:** The connection status in C&DH telemetry shows as "Connected". The C&DH can browse the health and status of the instrument.
- **Dependencies:** Radar instrument running and accessible on the testbed.

### User Story: Configure Mass Spectrometer Interface for Data Publishing

**Description:** As an Instrument Engineer, I want to configure the Mass Spectrometer to connect to the C&DH, model the relevant data points, and publish them to the SpaceWire bus under the correct CCSDS packet definition.

**Acceptance Criteria:**
- A secure connection is established between the mass spec and the C&DH.
- Data models for the pilot mass spec tags are created in the FSW.
- The mass spec is configured to publish data to the SpaceWire bus, mapping the data to the defined packet structure.
- Data flow is validated from the mass spec to the C&DH.

#### Task: Create C&DH Connection to Mass Spectrometer
**Description:** Establish a connection from the C&DH service to the Mass Spectrometer.
- **Technical Details:** In the FSW, create a new data source connection for the mass spec, using the appropriate connector. Provide the necessary connection details.
- **Expected Outcomes:** A successful connection to the Mass Spectrometer.
- **Definition of Done:** The connection status in C&DH telemetry shows as "Connected".
- **Dependencies:** Mass Spectrometer running and accessible.

### User Story: Establish Secure Downlink from C&DH to DSN

**Description:** As a Downlink Engineer, I want to configure the C&DH data buffer to securely stream data to the Deep Space Network (DSN) receiver, using managed identities for authentication to avoid managing secrets.

**Acceptance Criteria:**
- The FSW Downlink Extension is configured on the on-board C&DH.
- The downlink uses a Managed Identity with least-privilege access to authenticate to the GDS.
- The connection uses appropriate space-ground protocols for end-to-end validation.
- Data from specific APIDs is successfully downlinked to the target GDS.

#### Task: Configure C&DH Downlink Service
**Description:** Install and configure the FSW Downlink Service on the C&DH.
- **Technical Details:** Deploy the extension FSW module. Create a configuration file that specifies the DSN ground station, authentication method (Managed Identity), and APID mappings.
- **Expected Outcomes:** The extension is loaded and running on the C&DH.
- **Definition of Done:** The FSW telemetry shows that the extension has started successfully.
- **Dependencies:** A running C&DH; a GDS instance.

### User Story: Implement and Test On-Board Data Buffering

**Description:** As a FSW Engineer, I want to configure and test the buffering mechanism on the C&DH downlink service to ensure no data is lost during a simulated communications blackout.

**Acceptance Criteria:**
- The C&DH service is configured with a solid-state buffer of a specified size (e.g., capable of storing 3 full science orbits of data).
- A test is conducted where the DSN connection is severed for a period (e.g., 1 hour).
- Upon reconnection, the C&DH successfully sends all buffered data to the GDS.
- No data loss is verified by comparing on-board and ground data counts.

#### Task: Configure Solid-State Buffering
**Description:** Configure the FSW Downlink Service to use a persistent, solid-state buffer for outgoing telemetry.
- **Technical Details:** In the FSW configuration, set the `mode` to `persistent` for the telemetry buffer. Specify the maximum size of the buffer on the solid-state recorder.
- **Expected Outcomes:** The C&DH will write telemetry to the SSDR if it cannot connect to the DSN.
- **Definition of Done:** The configuration is applied and the C&DH is restarted. The buffer partition is created on the SSDR.
- **Dependencies:** Downlink Service is installed.

## Feature 2: Ground Data Ingestion and Processing (Science Data Levels)

**Description:** This feature covers the ingestion of streaming telemetry from the DSN into the Ground Data System (GDS) and processing it through a multi-layered Science Data Level architecture (Level 0, 2, 3). This ensures data is progressively cleaned, calibrated, and aggregated, making it reliable and ready for diverse use cases like analytics and ML.

### User Story: Provision and Configure GDS for Scalability

**Description:** As a Ground Segment Engineer, I want to provision a GDS ingest pipeline with a partition strategy that supports high-throughput and parallel consumption, so that our data ingestion pipeline is scalable and performant.

**Acceptance Criteria:**
- A GDS ingest point is provisioned using Infrastructure as Code (Terraform).
- The pipeline is configured with an appropriate number of partitions based on expected data volume.
- A dedicated consumer group is created for the SDPP streaming job.
- Private endpoint connectivity is enabled to restrict access to within the NASA VNet.

#### Task: Write Bicep/Terraform for GDS Ingest
**Description:** Author an Infrastructure as Code (IaC) template to define the GDS ingest pipeline and its configuration.
- **Technical Details:** Create a Terraform file that defines the ingest resources. Include parameters for SKU, capacity, and partition count. Define a child resource for the data stream itself and a consumer group.
- **Expected Outcomes:** A reusable IaC template for deploying the GDS.
- **Definition of Done:** The template is complete and checked into the GitHub repository. It has been successfully deployed to a development environment.
- **Dependencies:** A mission subscription and resource group.

### User Story: Develop Level 0 Ingestion Processing Job

**Description:** As a Data Engineer, I want to develop a SDPP Spark streaming job that reads raw data from the GDS and writes it to the Level 0 PDS Archive as immutable FITS files, partitioned by date.

**Acceptance Criteria:**
- The Spark job uses the GDS connector to read the data stream.
- The job writes the raw event payload (including metadata) to the PDS in FITS format.
- Data is partitioned by year, month, and day for efficient querying.
- The job uses watermarking to handle late-arriving data and manages checkpoints for stream resiliency.

#### Task: Develop Spark Streaming Notebook
**Description:** Write a PySpark notebook in the Science Data Platform to perform the stream ingestion.
- **Technical Details:** Use `spark.readStream.format("gds-connector")` to connect to the GDS endpoint. Configure the connection with the necessary credentials. Use `writeStream.format("fits")` to write the data, specifying the output path and partitioning columns.
- **Expected Outcomes:** A notebook that can read from GDS and write to the PDS.
- **Definition of Done:** The notebook runs without errors and writes data to the specified PDS location.
- **Dependencies:** GDS provisioned and receiving data.

### User Story: Implement a Bad Telemetry Queue for Ingestion Errors

**Description:** As a Data Engineer, I want to implement a bad-telemetry mechanism for the Level 0 ingestion job, so that any corrupted or un-processable frames are captured for later analysis without halting the entire stream.

**Acceptance Criteria:**
- The Spark streaming job includes a `try/except` block to catch parsing errors.
- Frames that fail to process are written to a separate "quarantine" directory in the PDS.
- An alert is configured to notify the data engineering team when new messages land in the quarantine directory.

#### Task: Add Error Handling to Spark Job
**Description:** Modify the Spark streaming notebook to include error handling logic.
- **Technical Details:** Wrap the data processing logic in a `try/except` block. In the `except` block, log the error and write the original frame data to a separate PDS path (e.g., `/level0/quarantine/`).
- **Expected Outcomes:** The streaming job will not fail on bad records.
- **Definition of Done:** When a malformed frame is sent from the GDS, the job continues to run, and the bad frame is saved to the quarantine location.
- **Dependencies:** A working Level 0 ingestion job.

### User Story: Develop Level 2 Cleansing and Calibration Job

**Description:** As a Data Engineer, I want to develop a batch or streaming job that processes raw Level 0 data, applies calibration files, and conforms it into a well-defined schema in the Level 2 layer of the Science Database.

**Acceptance Criteria:**
- The job reads new data from the Level 0 layer.
- Data is deduplicated based on a unique packet identifier.
- The packet APID is parsed into structured columns (Instrument, Measurement, etc.).
- Data types are enforced, and null values are handled.
- The cleaned and calibrated data is written to Delta tables in the Level 2 layer.

#### Task: Create Level 2 Processing Notebook
**Description:** Develop a science platform notebook to read from the Level 0 FITS files and write to Level 2 Delta tables.
- **Technical Details:** Use Spark SQL or DataFrame operations to read the Level 0 data. Apply transformations such as `split()` to parse the packet header and `withColumn()` to apply calibration curves. Use `dropDuplicates()` to handle duplicate packets. Write the transformed DataFrame to a Delta table.
- **Expected Outcomes:** A notebook that transforms Level 0 data into a clean, structured format.
- **Definition of Done:** The notebook runs successfully. The Level 2 Delta table is created and populated with clean data. The table schema matches the defined specification.
- **Dependencies:** Data present in the Level 0 layer.

### User Story: Develop Level 3 Aggregation Job for Analytics

**Description:** As a Science Analyst, I want to create a data pipeline that aggregates Level 2 layer data into science-centric models (e.g., daily plume activity maps, subsurface ice density) in the Level 3 layer, optimized for reporting.

**Acceptance Criteria:**
- A pipeline or notebook reads from Level 2 layer Delta tables.
- Data is aggregated into key metrics (e.g., averages, min/max values) over specific time windows.
- The aggregated data is stored in Level 3 layer Delta tables with a user-friendly schema.
- The resulting tables are performant for queries from analytics tools.

#### Task: Create Level 3 Aggregation Notebook
**Description:** Develop a notebook to create aggregated tables for science reporting.
- **Technical Details:** Read data from the Level 2 Delta tables. Use `groupBy()` and windowing functions (`window()`) in Spark to aggregate data over time. Select and rename columns to create a user-friendly, denormalized table.
- **Expected Outcomes:** A notebook that produces aggregated Level 3 tables.
- **Definition of Done:** The notebook runs successfully. The Level 3 tables are created and contain the correct aggregated data.
- **Dependencies:** Clean data in the Level 2 layer.

## Feature 3: Autonomous Science Model Development and Training

**Description:** This feature encompasses the process of developing, training, and validating the autonomous science ML model within the Science Operations Center (SOC). This includes an initial port of the vendor's model, followed by in-house iteration and improvement using MLOps best practices.

### User Story: Perform Exploratory Data Analysis (EDA) on Level 3 Layer Data

**Description:** As a Data Scientist, I want to perform a thorough exploratory data analysis on the curated Level 3 layer data to understand data distributions, identify correlations, and validate assumptions before model development.

**Acceptance Criteria:**
- A notebook is created for EDA.
- Visualizations (histograms, scatter plots, etc.) are generated to understand key features.
- A summary report of findings, including data quality issues or interesting patterns, is produced.

#### Task: Create EDA Notebook
**Description:** Develop a notebook to load Level 3 layer data and perform exploratory analysis.
- **Technical Details:** Load data from a Level 3 Delta table into a Spark DataFrame. Convert to a Pandas DataFrame for use with libraries like Matplotlib and Seaborn. Generate plots to visualize feature distributions, correlations, and relationships with the target variable.
- **Expected Outcomes:** A notebook with detailed data analysis and visualizations.
- **Definition of Done:** The notebook is complete and has been reviewed by the data science team. Key findings are documented.
- **Dependencies:** Aggregated data in the Level 3 layer.

### User Story: Translate and Adapt SwRI's Feature Engineering to SOC

**Description:** As a Data Scientist, I want to replicate the feature engineering logic from SwRI's model in a SOC notebook, using the Level 3 layer data as the source, so we can own and understand the data preparation process.

**Acceptance Criteria:**
- A SOC notebook is created that contains the PySpark code for all feature engineering steps.
- The output of the notebook is a feature set that is structurally identical to the one used by the vendor.
- The notebook is parameterized and can be run as part of a larger pipeline.

#### Task: Implement Feature Engineering Logic
**Description:** Write the PySpark code in a SOC notebook to implement the feature engineering transformations.
- **Technical Details:** Based on the logic from SwRI, implement transformations such as creating lag features, rolling averages, and other domain-specific features. Use Spark UDFs if necessary for complex transformations. The final output should be a single DataFrame ready for model training.
- **Expected Outcomes:** A feature engineering notebook.
- **Definition of Done:** The notebook runs and produces the expected feature set. The code is commented and follows project coding standards.
- **Dependencies:** EDA is complete; logic from SwRI is available.

### User Story: Integrate SOC Notebooks with MLflow for Automated Logging

**Description:** As a Data Scientist, I want to integrate our training notebooks with the MLflow tracking service in the SOC to automatically log all relevant experiment information, so we have a reproducible and auditable history of our model development.

**Acceptance Criteria:**
- The training notebook uses `mlflow.autolog()` or manual logging calls.
- For each experiment run, MLflow captures the Git commit hash, parameters, performance metrics, and the model artifact itself.
- Experiment results are visible and comparable in the SOC UI.

#### Task: Add MLflow Logging to Training Notebook
**Description:** Modify the model training notebook to include MLflow logging.
- **Technical Details:** Add `import mlflow` to the notebook. Use `mlflow.start_run()` to begin a new run. Log parameters using `mlflow.log_param()`, metrics with `mlflow.log_metric()`, and the trained model with `mlflow.spark.log_model()`.
- **Expected Outcomes:** The training notebook will log all experiment details to MLflow.
- **Definition of Done:** After running the notebook, a new experiment run appears in the MLflow UI in the SOC with all the specified information.
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

### User Story: Validate and Register Flight-Candidate Model in MLflow

**Description:** As a Data Scientist, I want to validate the final trained model against a hold-out test set and, if it meets performance criteria, register it in the MLflow Model Registry and transition its stage to "Flight-Ready".

**Acceptance Criteria:**
- The model's performance is evaluated on an unseen test dataset.
- The performance meets or exceeds the pre-defined mission requirements (e.g., >95% precision).
- The model is saved to MLflow and registered in the Model Registry.
- The model version is transitioned to the "Flight-Ready" stage, triggering the CI/CD pipeline.

#### Task: Create Model Validation and Registration Notebook
**Description:** Develop a notebook that loads the trained model, evaluates it on a test set, and registers it in the MLflow Model Registry.
- **Technical Details:** Load the model from an MLflow run. Score it against a previously unseen test dataset. Calculate performance metrics. If the metrics meet the required threshold, use `mlflow.register_model()` to create a new version in the registry.
- **Expected Outcomes:** A new, validated model version in the MLflow Model Registry.
- **Definition of Done:** The notebook runs successfully. A new model version is created in the registry and is visible in the SOC UI.
- **Dependencies:** A trained model from the previous step.

## Feature 4: FSW MLOps and Uplink Pipeline

**Description:** This feature focuses on creating a robust, automated CI/CD pipeline using GitHub Actions. This pipeline will be responsible for cross-compiling the trained ML model into an FSW module, pushing it to a secure repository, and triggering the uplink to the on-board flight computer.

### User Story: Develop a Production-Ready FSW Inference Module

**Description:** As an FSW Engineer, I want to develop a lightweight C/C++ module that loads the trained model and exposes an FSW API for inference, so the model can be easily consumed by the flight software.

**Acceptance Criteria:**
- A C/C++ module for the FSW is created.
- The module includes a `predict()` function that accepts input data and returns a model prediction.
- The module includes a `health_check()` function for health checks.
- The module is stored in the model's GitHub repository.

#### Task: Write RAD750 Inference Service
**Description:** Create a C/C++ module to serve the ML model on the RAD750.
- **Technical Details:** Use C/C++ to create the FSW module. Implement a `health_check()` function that returns a nominal status. Implement a `predict()` function that accepts a data structure, uses the loaded ML model to make a prediction, and returns the result.
- **Expected Outcomes:** A C/C++ module for the inference service.
- **Definition of Done:** The module cross-compiles and responds to function calls correctly in the testbed.
- **Dependencies:** A trained model in a format that can be loaded (e.g., C-header or binary).

### User Story: Author a Build Script for FSW Module

**Description:** As an FSW Engineer, I want to author a build script that efficiently packages the inference module, the trained model, and all necessary dependencies into a minimal, flight-ready binary.

**Acceptance Criteria:**
- A `build.sh` script is created in the GitHub repository.
- The script uses a specific, versioned cross-compiler (e.g., `rad750-gcc`).
- The resulting FSW binary is small and free of unnecessary build dependencies.
- The binary can be built and run successfully on a local emulator.

#### Task: Create Cross-Compilation Build Script
**Description:** Write a build script to create an optimized production binary.
- **Technical Details:** Use a first stage to download dependencies. In a second stage, use the `rad750-gcc` toolchain to compile the C/C++ code and link the model binary. Set the `CMD` to produce the final `.bin` or `.so` module.
- **Expected Outcomes:** A `build.sh` script in the project repository.
- **Definition of Done:** The `build.sh` command completes successfully. The resulting binary is smaller than a standard build would be.
- **Dependencies:** The C/C++ inference module and a `requirements.txt` file (or equivalent).

### User Story: Provision FSW Artifact Repository for FSW Binaries

**Description:** As a Ground Segment Engineer, I want to provision a secure FSW Artifact Repository to store our production model FSW binaries, so that we have a private, managed repository for our uplinkable modules.

**Acceptance Criteria:**
- An Artifactory or similar instance is provisioned using IaC.
- The repository is configured with private access, accessible only from our CI/CD agents and the DSN uplink systems.
- A repository is created for the autonomous science model binaries.

#### Task: Write Bicep/Terraform for Artifactory
**Description:** Author an IaC template to define the Artifact Repository.
- **Technical Details:** Create a Bicep or Terraform file that defines the `Microsoft.ContainerRegistry/registries` (or equivalent Artifactory) resource. Set the SKU to `Premium` to allow for private endpoints. Configure network rules to restrict access.
- **Expected Outcomes:** A reusable IaC template for deploying Artifactory.
- **Definition of Done:** The template is checked into GitHub and successfully deployed.
- **Dependencies:** A mission subscription and resource group.

### User Story: Implement CI Pipeline to Build and Archive FSW Module

**Description:** As an MLOps Engineer, I want to create a GitHub Actions CI workflow that automatically cross-compiles, tags, and pushes the model's FSW binary to Artifactory whenever a new model is tagged for release in the Git repository.

**Acceptance Criteria:**
- A GitHub Actions workflow file is created.
- The workflow is triggered by a new Git tag (e.g., `v1.2.0`).
- The workflow authenticates to Artifactory using a service principal or managed identity.
- The FSW binary is built and pushed to Artifactory with a tag corresponding to the Git tag.

#### Task: Create GitHub Actions CI Workflow
**Description:** Create the YAML file for the GitHub Actions workflow.
- **Technical Details:** Create a `.github/workflows/ci.yml` file. Use the `on: push: tags:` trigger. Use the `azure/login` action to authenticate. Use a build script and an `artifactory-upload` action to build the binary and push it.
- **Expected Outcomes:** A CI workflow file in the repository.
- **Definition of Done:** When a new tag is pushed to GitHub, the workflow runs successfully and the new FSW binary appears in Artifactory.
- **Dependencies:** An Artifactory instance; a build script.

### User Story: Implement CD Pipeline to Uplink Module via AMO

**Description:** As an MLOps Engineer, I want to extend the GitHub Actions workflow to act as a CD pipeline that, after a successful binary push, updates the Autonomous Mission Operations (AMO) sequence file to roll out the new FSW module to the on-board flight computer.

**Acceptance Criteria:**
- The workflow retrieves the AMO sequence file from the repository.
- It updates the module version in the sequence.
- It uses the CLI or a dedicated GitHub Action to apply the updated sequence to the target Mission Ops uplink queue.
- The pipeline includes a manual approval step from Mission Ops before uplink.

#### Task: Create GitHub Actions CD Workflow
**Description:** Add a deployment job to the GitHub Actions workflow.
- **Technical Details:** Add a new job to `ci.yml` that depends on the build job. Use a tool like `sed` or `yq` to update the version in the `sequence.xml` file. Use a custom action (`mission-ops/uplink`) to apply the sequence.
- **Expected Outcomes:** A CD workflow that automates uplink preparations.
- **Definition of Done:** When the workflow runs, the new module version is queued for uplink, which can be verified in the MOC portal.
- **Dependencies:** A CI workflow; an AMO service registered in the MOC.

## Feature 5: Flight Segment Model Deployment and Inference

**Description:** This feature covers the setup of the on-board flight computer to host the ML model and run inference locally. This is the most critical component for ensuring the spacecraft can continue to operate and benefit from model predictions even during a complete comms blackout.

### User Story: Install and Configure Flight Executive (cFE/CFS)

**Description:** As an FSW Engineer, I want to install and configure the Core Flight Executive (cFE) on our RAD750 flight computer, so we have a resilient and scalable platform for running our FSW modules.

**Acceptance Criteria:**
- A multi-module cFE/CFS instance is installed and configured.
- The instance has role-based access control (RBAC) enabled for commands.
- The ground system is configured as a trusted source for new FSW modules.

#### Task: Install cFE/CFS on RAD750
**Description:** Install the cFE/CFS distribution on the provisioned flight computers.
- **Technical Details:** Choose a cFE/CFS distribution suitable for the mission. Follow the official documentation to install the core services on the computer. Verify service status with telemetry.
- **Expected Outcomes:** A running cFE/G-FSW instance.
- **Definition of Done:** All core services are in the `Ready` state. Telemetry can be received from the instance.
- **Dependencies:** Provisioned flight computers.

### User Story: Configure FSW Software Bus (CFS-SB) Policies for Secure Communication

**Description:** As a Security Engineer, I want to implement CFS-SB filters to strictly control data flow between the ML model module and the on-board data bus, ensuring the model can only communicate with the intended services.

**AcceptanceCriteria:**
- A CFS-SB filter is defined that allows egress packets from the model module only to the C&DH APID.
- A filter is defined that allows ingress packets to the model module only from trusted instrument APIDs.
- All other ingress and egress traffic from the model module is denied by default.

#### Task: Define and Apply Packet Filters
**Description:** Create configuration tables for the CFS-SB filters and apply them to the FSW load.
- **Technical Details:** Create a `NetworkPolicy` manifest. Define a `podSelector` to target the ML model pods. Create an `egress` rule that specifies the IP address and port of the HiveMQ broker. Apply the policy using `kubectl apply -f <policy-file>.yaml`.
- **Expected Outcomes:** A Network Policy resource created in the cluster.
- **Definition of Done:** Test connectivity from the model pod to an unauthorized endpoint (should fail). Test connectivity to the HiveMQ broker (should succeed).
- **Dependencies:** A running K8s cluster with a CNI that supports Network Policies (e.g., Calico).

### User Story: Install and Register AMO on the Flight Computer

**Description:** As an FSW Engineer, I want to install the Autonomous Mission Operations (AMO) service on our on-board flight computer and register it with our MOC, so that it can be managed and receive deployment instructions from the ground.

**Acceptance Criteria:**
- The AMO FSW module is successfully deployed to the flight computer.
- The AMO service is successfully registered in the MOC and shows a connected state.
- The `amo` command-line tool can be used to check the status of the runtime and modules.

#### Task: Deploy AMO to Flight Computer
**Description:** Deploy the AMO runtime to the on-board flight computer using FSW build scripts.
- **Technical Details:** Add the AMO module to the FSW build. Use the build script to install the `amo` module, providing the device connection string from the MOC as a parameter. Verify that the `amoAgent` and `amoHub` services are created and running.
- **Expected Outcomes:** AMO runtime installed on the flight computer.
- **Definition of Done:** The `amoAgent` and `amoHub` tasks are in the `Running` state. The device appears as "Connected" in the MOC portal.
- **Dependencies:** A running cFE/CFS instance; a MOC instance.

### User Story: Deploy and Validate the ML Model Module on-Flight

**Description:** As an MLOps Engineer, I want to perform the first deployment of the ML model module to the on-board computer via the CD pipeline and run a smoke test to validate that it's running correctly.

**Acceptance Criteria:**
- The CD pipeline successfully deploys the model module to the flight computer.
- The model module starts and is in a `Running` state.
- The module telemetry shows a successful subscription to the data bus.
- A test message sent to the input APID results in a valid prediction on the output APID.

#### Task: Run Initial Uplink Pipeline
**Description:** Trigger the CD pipeline to deploy the model to the on-board computer for the first time.
- **Technical Details:** Manually trigger the GitHub Actions CD workflow. Monitor the pipeline execution to ensure all steps pass. After the pipeline completes, use telemetry to verify that the model module has been created.
- **Expected Outcomes:** The model module is deployed and running on the flight computer.
- **Definition of Done:** The module is running. Telemetry from the module shows no startup errors.
- **Dependencies:** A completed CD pipeline; a running AMO on cFE/CFS setup.

### User Story: Implement On-Board Telemetry Buffering

**Description:** As an OT Engineer, I want to ensure that telemetry from the running model module is collected and available locally on the Solid-State Data Recorder (SSDR), so that we can monitor the model's health and troubleshoot issues even if the DSN connection is down.

**Acceptance Criteria:**
- The cFE/CFS instance is configured to collect telemetry from all modules (e.g., using the Telemetry Output service).
- Telemetry is forwarded to the on-board SSDR.
- Operators have a documented procedure for accessing and searching these local logs during an outage.

#### Task: Deploy a Local Telemetry Aggregator
**Description:** Deploy a telemetry aggregation tool like the CFS Telemetry Output (TO) service on the flight computer.
- **Technical Details:** Use a FSW build script to deploy TO to the computer. Configure TO to collect telemetry from all modules. Configure the output plugin to forward telemetry to the SSDR.
- **Expected Outcomes:** A telemetry aggregation pipeline running on the flight computer.
- **Definition of Done:** TO is running on all nodes. Telemetry from the model module is appearing in the configured SSDR location.
- **Dependencies:** A running cFE/CFS instance.

## Feature 6: Mission Data Governance, Security, and Identity Management

**Description:** This feature ensures that the entire system is secure, compliant, and well-governed, with proper access controls and monitoring in place across both ground and flight environments. It implements a Zero Trust approach wherever possible.

### User Story: Implement Least-Privilege Access with NASA Identity Services

**Description:** As a Security Administrator, I want to use NASA Identity Services (PIM) for Groups to provide just-in-time (JIT) access to privileged roles in the GDS, so that we minimize standing permissions and reduce our security exposure.

**Acceptance Criteria:**
- NASA Identity Service groups are created for different privilege levels (e.g., `SOC-Admins`, `PDS-Contributors`).
- These groups are enabled in PIM, requiring users to request and justify temporary access.
- Access reviews are scheduled quarterly to recertify the need for membership in these groups.

#### Task: Configure PIM for Groups
**Description:** Onboard the necessary NASA Identity Service groups to PIM and configure the access settings.
- **Technical Details:** In the NASA Identity portal, navigate to PIM -> Groups. Select the target groups and bring them under PIM management. Configure the role settings to require approval for activation and set a maximum duration.
- **Expected Outcomes:** Privileged access to resources will be JIT.
- **Definition of Done:** A user successfully requests, is granted, and uses temporary access to a privileged role.
- **Dependencies:** NASA Identity P2 license.

### User Story: Secure PDS Node with Network Controls

**Description:** As a Cloud Security Engineer, I want to secure our PDS Node account by disabling public access and using private endpoints and VNet service endpoints, so that data can only be accessed from trusted services and networks.

**Acceptance Criteria:**
- The PDS account's public network access is disabled.
- A private endpoint for the PDS account is created in the main VNet.
- The SOC is configured to access the PDS through the trusted workspace access mechanism.
- All access attempts are logged and can be audited.

#### Task: Configure PDS Networking
**Description:** Modify the PDS account's networking settings to disable public access and create a private endpoint.
- **Technical Details:** In the PDS networking blade, set "Public network access" to "Disabled". Create a new private endpoint, connecting it to the PDS `dfs` sub-resource and placing it in the appropriate VNet/subnet.
- **Expected Outcomes:** The PDS account is no longer accessible from the public internet.
- **Definition of Done:** An attempt to access the PDS account from a public IP address fails. A VM within the VNet can access the account via the private endpoint.
- **Dependencies:** A PDS account and a VNet.

### User Story: Centralize Threat Detection with Mission SOC

**Description:** As a Security Administrator, I want to onboard the SOC subscription to the central Mission SOC workspace and enable relevant data connectors, so that we have a single pane ofglass for threat detection and incident response.

**Acceptance Criteria:**
- The Activity, NASA Identity, and Microsoft Defender for Cloud data connectors are enabled for the SOC subscription.
- Logs are confirmed to be flowing into the Mission SOC Log Analytics workspace.
- At least one custom alert rule is created based on a potential threat scenario relevant to this project (e.g., anomalous data egress from PDS).

#### Task: Configure Sentinel Data Connectors
**Description:** Connect the SOC subscription's log sources to the Mission SOC workspace.
- **Technical Details:** From the Mission SOC workspace, navigate to Data Connectors. Open the connector page for Activity Log, NASA Identity, etc. Follow the instructions to configure the connection to the SOC tenant and subscription.
- **Expected Outcomes:** Logs from the SOC tenant are ingested into the Mission SOC.
- **Definition of Done:** Data is visible in the connector dashboards and can be queried in Log Analytics.
- **Dependencies:** Appropriate cross-tenant permissions (Lighthouse).

## Feature 7: Mission Reporting and Analytics

**Description:** This feature focuses on creating Power BI and Grafana reports and dashboards to provide insights to various stakeholders, from flight controllers to mission leadership, by combining FSW data with ground data. The goal is to move from simple operational reporting to strategic, data-driven decision-making.

### User Story: Integrate Mission Scheduling Data into SOC

**Description:** As a BI Developer, I want to use a connector in the SOC to ingest DSN scheduling data (e.g., comms windows, data rates) into the Level 3 layer of the PDS, so it can be joined with our FSW data.

**Acceptance Criteria:**
- A connection to the MOC's DSN scheduling system is established in the SOC.
- A dataflow or pipeline is created to incrementally load relevant schedule tables into the Level 3 layer.
- The data is available as Delta tables and can be queried.

#### Task: Create SOC Dataflow for DSN Schedule
**Description:** Use a Dataflow Gen2 in the SOC to connect to and ingest data from the DSN.
- **Technical Details:** Create a new Dataflow Gen2. Use the built-in DSN connector. Authenticate using an organizational account with permissions to the MOC. Select the required tables and configure incremental refresh if possible.
- **Expected Outcomes:** A dataflow that copies DSN data to the Level 3 layer.
- **Definition of Done:** The dataflow runs successfully and the target Delta tables in the Level 3 layer are populated.
- **Dependencies:** Access credentials for the DSN scheduling environment.

### User Story: Develop Mission Leadership KPI Dashboard in Power BI

**Description:** As a BI Developer, I want to create a Power BI dashboard for mission leadership that marries FSW data (e.g., events detected, data buffered) with DSN data to provide high-level mission KPIs.

**Acceptance Criteria:**
- A Power BI semantic model is created in DirectLake mode over the Level 3 layer tables.
- The dashboard includes KPIs for Science Data Return (SDR) with associated data volume.
- A "what-if" analysis feature is included to estimate science return from preventing data loss.
- The report is published as a Power BI App and shared with the mission leadership user group.

#### Task: Create Mission Ops Semantic Model
**Description:** Create a new Power BI semantic model in the SOC workspace that connects to the Level 3 layer tables.
- **Technical Details:** Create a new semantic model. Add the Level 3 layer Delta tables as sources. Use the modeling view to define relationships between the FSW and DSN data tables. Create DAX measures for key KPIs.
- **Expected Outcomes:** A well-structured Power BI semantic model.
- **Definition of Done:** The model is created, relationships are correct, and key measures are defined.
- **Dependencies:** Level 3 layer tables are available.

### User Story: Create Power BI Report for FSW Model Performance Monitoring

**Description:** As a Data Analyst, I want to create a Power BI report that connects to MLflow and our PDS to visualize the ongoing performance of the production FSW model, so we can detect data or concept drift.

**Acceptance Criteria:**
- The Power BI report connects to the Level 3 layer and the MLflow tracking server (via API or exported data).
- The report visualizes the distribution of input features over time to detect data drift.
- It compares model prediction distributions to actual outcomes to detect concept drift.
- The report is published and made available to the data science team.

#### Task: Connect Power BI to MLflow
**Description:** Develop a method to get MLflow experiment data into Power BI.
- **Technical Details:** Write a SOC notebook to periodically query the MLflow API (using `mlflow.search_runs()`), extract relevant metrics and parameters into a DataFrame, and save it as a Delta table in the Level 3 layer. This table can then be easily imported into Power BI.
- **Expected Outcomes:** A table in the Level 3 layer containing MLflow experiment data.
- **Definition of Done:** The notebook runs and successfully populates the MLflow data table.
- **Dependencies:** MLflow experiments have been run.
