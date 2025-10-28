# Copilot Instructions for the Flight Operations Manual for the JIMO-2 Autonomy Project

This document provides guidance for AI and flight software agents assisting with the Jovian Ice Moon Orbiter (JIMO-2) project. Understanding these principles is crucial for generating relevant and correct flight software (FSW), ground data system (GDS) infrastructure, and mission documentation.

## 1. The Big Picture: A Hybrid Ground and Flight Architecture

The primary goal of this mission is to autonomously detect and analyze transient phenomena (e.g., water plumes) on an ice moon. The single most important architectural principle is **resilience against communication latency and blackouts**. The spacecraft must remain operational, scientifically productive, and safe, even when out of contact with the Deep Space Network (DSN).

This leads to a hybrid architecture:

-   **Ground Segment (Earth):** Used for data reception, archiving, complex processing, and model training.
-   **Flight Segment (Spacecraft):** Used for data acquisition from instruments and, critically, for **hosting the final, production-inference "science-on-the-fly" ML models**.

### Core Components & Data Flow:

1.  **Flight Data Acquisition:**
    -   Instruments (Mass Spectrometer, Ice-Penetrating Radar) collect raw science data.
    -   Data is published to the on-board **SpaceWire data bus**.

2.  **Ground Ingestion & Processing:**
    -   The spacecraft's data buffer is downlinked via the **DSN** to the **Ground Data System (GDS) at JPL**.
    -   *(Future State: This GDS will be replaced by a cloud-native **Common Data Pipeline**).*
    -   A **NASA Science Data Processing Pipeline (SDPP)** subscribes to the GDS, processes the streaming telemetry, and lands it in two locations:
        -   **Level 0 Archive:** Raw, unprocessed files in the **Planetary Data System (PDS) Node**.
        -   **Level 2/3 Products:** Processed and calibrated data in the **Science Team's Operations Database**.

3.  **Ground Model Training (Science Ops Center):**
    -   Scientists use SOC tools, including **PyTorch and TensorFlow**, to train, experiment, and register ML models (e.g., plume detection).
    -   The initial model is being developed by an instrument vendor (**SwRI**) on their platform. The logic will be ported to the SOC's environment for in-house ownership.

4.  **Flight Model Deployment (The Critical Path):**
    -   Once a model is validated by the Science Team, it is **cross-compiled for the RAD750 FSW**.
    -   This new FSW module is uplinked to the **on-board flight computer**.
    -   **Autonomous Mission Operations (AMO)** is the FSW service used to manage the deployment and lifecycle of the new model on the flight computer. This allows for centralized control from Earth while ensuring local, autonomous execution.

### Multi-Center Environment:

The environment is split across two primary operations centers, and it's important to understand the separation of duties:

-   **Science Operations Center (SOC) (University Partner):**
    -   Hosts the Science Data Processing Pipeline.
    -   Hosts the PDS archives for science data.
    -   This is where scientists and data analysts primarily work.

-   **Mission Operations Center (MOC) (JPL):**
    -   Hosts centralized security, DSN scheduling, and spacecraft command/control.
    -   Manages access to the SOC via NASA Identity Services.
    -   Hosts flight dynamics systems and the primary telemetry database.

## 2. Critical Constraints & Conventions

Adherence to these constraints is non-negotiable.

### **Constraint #1: On-Board Autonomous Execution is Paramount**

-   Any proposed change to the FSW or GDS architecture must not compromise the ability of the on-board RAD750 computer to run the ML model inference workload independently of Earth.
-   Do not suggest solutions that require a live, real-time DSN connection for the model to perform a prediction or a critical safety check.

## 3. Developer Workflows & Tooling

The project is transitioning from legacy FORTRAN to modern, automated software development.

-   **Source Code Repository:** NASA GitHub Enterprise
-   **Work Item Tracking:** JIRA
-   **CI/CD Pipelines:** The goal is to build automated CI/CD pipelines. When creating workflows, assume:
    -   A `build` pipeline will cross-compile the FSW module for the RAD750.
    -   A `release` pipeline will deploy the new module to the "hardware-in-the-loop" testbed simulator.
-   **Infrastructure as Code (IaC):** The project will use IaC for all *ground* resources. Be prepared to generate Terraform or CloudFormation.

## 4. Key Roles & Responsibilities

-   **Mission Operations Team (JPL):** Focuses on the GDS infrastructure, DSN, security, and the on-board deployment via the AMO service.
-   **Science Team (University):** Data analysts and scientists who work inside the SOC to build data pipelines and train science-targeting models.
-   **SwRI (Instrument Vendor):** Provides the initial Ice-Penetrating Radar. The JIMO-2 team is responsible for taking ownership of this instrument's data processing and any associated autonomy models.
