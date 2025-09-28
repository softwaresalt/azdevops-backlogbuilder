# Copilot Instructions for the ACME Predictive Maintenance Project

This document provides guidance for AI coding agents to assist with the ACME Predictive Maintenance project. Understanding these principles is crucial for generating relevant and correct code, infrastructure, and documentation.

## 1. The Big Picture: A Hybrid On-Prem and Cloud Architecture

The primary goal of this project is to develop and deploy a machine learning model that predicts equipment failure in a manufacturing plant. The single most important architectural principle is **resilience against internet outages**. The plant must remain operational, and the ML model must continue to function, even if disconnected from the cloud.

This leads to a hybrid architecture:

-   **Cloud (Microsoft Azure):** Used for data ingestion, storage, processing, and model training.
-   **On-Premises:** Used for data acquisition from SCADA systems and, critically, for **hosting the final, production-inference ML model**.

### Core Components & Data Flow:

1.  **On-Prem Data Acquisition:**
    -   SCADA systems (Ignition, Rockwell) collect raw equipment data.
    -   Data is published to a local **HiveMQ MQTT broker**.

2.  **Cloud Ingestion & Processing:**
    -   The on-prem HiveMQ broker bridges data to a **HiveMQ broker running on a Windows VM in Azure**.
    -   *(Future State: This VM will be replaced by managed **Event Hubs**).*
    -   A **Microsoft Fabric Spark Notebook** (PySpark) subscribes to the cloud broker, processes the streaming data, and lands it in two locations:
        -   **Bronze Layer:** Raw Parquet files in **Data Lake Storage (ADLS) Gen2**.
        -   **Silver/Gold Layers:** Processed and aggregated data in the **Fabric Lakehouse/Warehouse**.

3.  **Cloud Model Training (Fabric):**
    -   Data scientists use Fabric tools, including **MLflow**, to train, experiment, and register ML models.
    -   The initial model is being developed by a vendor (**ModelQ**) on their platform. The logic will be ported to Fabric for in-house ownership.

4.  **On-Prem Model Deployment (The Critical Path):**
    -   Once a model is validated in Fabric, it is **containerized (Docker)**.
    -   This container is deployed to an **on-prem Kubernetes cluster**.
    -   **IoT Edge** is the mechanism used to manage the deployment and lifecycle of the container on the on-prem cluster. This allows for centralized control from while ensuring local execution.

### Multi-Tenant Environment:

The environment is split across two tenants, and it's important to understand the separation of duties:

-   **Acme-Sub Tenant (Data Plane):**
    -   Hosts the Microsoft Fabric workspace.
    -   Hosts the ADLS Gen2 storage account for the data lake.
    -   This is where data engineers and data scientists primarily work.

-   **ACME Tenant (Control/Business Plane):**
    -   Hosts centralized security, networking (hub-spoke firewall), and identity (Entra ID).
    -   Manages access to the Acme-Sub tenant via Entra ID B2B guest accounts.
    -   Hosts business systems like EntERPSys_A and a related ODS in SQL.

## 2. Critical Constraints & Conventions

Adherence to these constraints is non-negotiable.

### **Constraint #1: On-Prem Model Execution is Paramount**

-   Any proposed change to the architecture must not compromise the ability of the on-prem Kubernetes cluster to run the ML model inference workload independently of the cloud.
-   Do not suggest solutions that require a live, real-time connection to for the model to perform a prediction.

## 3. Developer Workflows & Tooling

The project is transitioning from manual processes to automated DevOps.

-   **Source Code Repository:** GitHub Enterprise
-   **Work Item Tracking:** Azure DevOps
-   **CI/CD Pipelines:** The goal is to build automated CI/CD pipelines. When creating workflows, assume:
    -   A `build` pipeline will containerize the ML application.
    -   A `release` pipeline will use IoT Edge to deploy the new container version to the on-prem Kubernetes cluster.
-   **Infrastructure as Code (IaC):** The project will use IaC for all resources. Be prepared to generate Bicep or Terraform.

## 4. Key Roles & Responsibilities

-   **ACME Team:** Focuses on the infrastructure, security, networking, and the on-prem deployment via IoT Edge.
-   **Acme-Sub Team:** Data engineers and scientists who work inside Microsoft Fabric to build data pipelines and train models.
-   **ModelQ (Vendor):** Provides the initial ML model. The ACME/Acme-Sub team is responsible for taking ownership of this model and rebuilding it in-house. When dealing with model-related code, remember that the initial logic comes from this external source.