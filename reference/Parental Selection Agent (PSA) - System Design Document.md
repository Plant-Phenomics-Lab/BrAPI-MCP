# **Parental Selection Agent (PSA) \- System Design Document**

Date: October 16, 2025

## 

**![][image1]**

I Fig. 1\. To alleviate the problem of fragmented workflows and data overload in plant breeding programs we propose two key infrastructure implementations. a) An ontology aligned image to database phenotyping pipeline; b) A natural language user interface that allows easy data discovery and download in addition to expert decision support for plant breeding.

## 

## **1\. Introduction**

This document outlines the system design for the Parental Selection Agent (PSA), an advanced AI system designed to provide decision support for plant breeders engaged in Marker-Assisted Parental Selection. The core mission of PSA is to transform this complex, multi-step process into a streamlined, conversational workflow.  
The system will function as a collaborative partner, guiding the breeder from the initial strategic planning of crosses to the final data-driven ranking of candidate parents. By leveraging a multi-agent architecture, PSA will separate distinct responsibilities—planning, data retrieval, and analysis—to create a robust, verifiable, and modular solution.

## **2\. System Overview and Architecture**

PSA will be implemented as a **multi-agent system**. This architecture is chosen over a monolithic design to ensure modularity, specialization, and clarity of roles. Each agent has a focused set of responsibilities and tools, which enhances robustness and simplifies development and maintenance.  
The system consists of three specialized agents operating under a central orchestrator:

* **CoordinatorAgent:** The user-facing orchestrator that manages the overall workflow.  
* **DataRetrieverAgent:** A specialist for all data gathering operations.  
* **DataAnalystAgent:** A specialist for all computational analysis and data processing tasks.

### **High-Level Workflow**

The interaction follows a structured, three-phase process managed by the CoordinatorAgent, with verification handshakes built in at the end of each phase to ensure accuracy and user control.

## **3\. Agent Roles and Responsibilities**

### **3.1. CoordinatorAgent (The Orchestrator)**

* **Role:** The primary interface for the plant breeder. It manages the state of the selection task and delegates work to the specialized agents.  
* **Responsibilities:**  
  * **Interactive Planning:** Engage the breeder in a dialogue to define the goals, traits, constraints, and desired number of crossing pairs.  
  * **Plan Formulation:** Synthesize the dialogue into a structured, machine-readable **Selection Plan**.  
  * **Workflow Management:** Orchestrate the sequence of tasks between the DataRetrieverAgent and DataAnalystAgent.  
  * **Communication Hub:** Act as the sole intermediary between the breeder and the backend agents, presenting summaries, verification checks, and final results.  
* **Tools:** None. This agent's function is restricted to reasoning, planning, and delegation.

### **3.2. DataRetrieverAgent (The Librarian)**

* **Role:** The data gathering specialist responsible for acquiring all necessary phenotypic and genotypic data.  
* **Responsibilities:**  
  * Execute the data collection section of the approved **Selection Plan**.  
  * Interface with any BrAPI-compliant database using the standardized Model Context Protocol (MCP) toolset.  
  * Process and read data from user-uploaded files (e.g., CSV, JSON).  
  * Consolidate, merge, and stage the collected data in a clean format for the DataAnalystAgent.  
* **Tools:**  
  * BrAPI/MCP Toolset  
  * File System Toolset (list\_files, read\_file)

### **3.3. DataAnalystAgent (The Scientist)**

* **Role:** The computational engine responsible for all data processing and analysis.  
* **Responsibilities:**  
  * Execute the analysis section of the approved **Selection Plan**.  
  * Perform successive filtering of male and female candidates based on phenotypic and marker data.  
  * Rank potential crosses based on a selection index or other criteria defined in the plan.  
  * Generate the final output files, including ranked lists of recommended crosses and summary statistics.  
* **Tools:**  
  * python\_interpreter  
  * File System Toolset (read\_file, write\_file, list\_files, delete\_file)

## **4\. The "Check-Confirm" Verification Framework**

A core principle of the PSA design is to ensure transparency and accuracy through a multi-layered verification process. The workflow is explicitly gated by **Verification Handshakes**, where the agent performs an automated self-check and then requires explicit user confirmation before proceeding.

1. **Planning Verification:**  
   * **Self-Check:** The CoordinatorAgent validates its generated Selection Plan against the conversation history.  
   * **Human Confirmation:** The plan is presented to the breeder for explicit approval. **The system will not proceed without a "Yes" from the user.**  
2. **Data Assembly Verification:**  
   * **Self-Check:** The DataRetrieverAgent runs a data quality profile on the collected data, checking for schema correctness, record counts, and missing values.  
   * **Human Confirmation:** The CoordinatorAgent presents a summary of the data quality check to the breeder, confirming the number of candidates found before proceeding to the computationally intensive analysis phase.  
3. **Analysis Verification:**  
   * **Self-Check:** The DataAnalystAgent verifies that its code executed without errors, logs the attrition rate at each filtering step, and checks if the final number of crosses meets the user's target.  
   * **Human Confirmation (Final Report):** The CoordinatorAgent presents a summary of the final results, confirming the outcome of the analysis.

## **5\. Tool and Environment Specification**

### **5.1. Code Interpreter Sandbox**

* **Technology:** The python\_interpreter tool will be implemented using **Docker containers** to ensure a secure, isolated, and reproducible execution environment.  
* **Security Model:** A strict security protocol will be enforced. A unique, temporary workspace directory will be created on the host for each session. This directory will be made available to the container via a **bind mount** (e.g., mapping host /tmp/psa\_session\_XYZ to container /workspace). The agent will have no access to any file or process outside this sandboxed workspace.  
* **Pre-installed Libraries:** The Docker image will contain a standard scientific Python stack, including pandas, numpy, scipy, matplotlib, seaborn, and scikit-learn.

### **5.2. Tool Definitions**

| Tool Name | Category | Agent(s) with Access | Description |
| :---- | :---- | :---- | :---- |
| python\_interpreter | Code Execution | DataAnalystAgent | Executes Python code in a secure sandbox for data analysis, filtering, and visualization. |
| list\_files | File System | Both DataRetriever & Analyst | Lists files in the current workspace. |
| read\_file | File System | Both DataRetriever & Analyst | Reads the content of a specified file. |
| write\_file | File System | DataAnalystAgent | Writes content to a specified file. |
| delete\_file | File System | DataAnalystAgent | Deletes a file from the workspace. |
| search\_studies | BrAPI/MCP | DataRetrieverAgent | Searches for studies/trials based on specified criteria. |
| get\_study\_details | BrAPI/MCP | DataRetrieverAgent | Retrieves all detailed information for a single study, given a studyDbId. |
| get\_observations | BrAPI/MCP | DataRetrieverAgent | Retrieves phenotypic observation data for a given study or observation unit. |
| search\_germplasm | BrAPI/MCP | DataRetrieverAgent | Searches for germplasm based on names or other attributes. |
| get\_germplasm\_by\_id | BrAPI/MCP | DataRetrieverAgent | Retrieves all detailed attributes for a single germplasm entry, given a germplasmDbId. |
| get\_observation\_variables | BrAPI/MCP | DataRetrieverAgent | Lists the phenotypic traits (observation variables) available in the database. |
| get\_pedigree | BrAPI/MCP | DataRetrieverAgent | Retrieves the pedigree information (parent-child relationships) for a specific germplasm. |

## **6\. Project Longevity and Future-Proofing**

The PSA system is designed to produce durable assets for the plant breeding community that will outlast any specific LLM.

* **Swappable Core:** The multi-agent architecture allows the LLM core of each agent to be upgraded as more powerful models become available.  
* **Durable Context:** The MCP files created for BrAPI databases will form a standardized, open toolkit that can be used by any future AI system.  
* **Data Flywheel:** All verified interactions (plan \-\> data \-\> analysis \-\> result) will be curated to create a high-quality dataset for fine-tuning future, domain-specialized agents.  
* **Reusable Workflow Repository:** The system will allow users to save and share successful selection plans and analysis workflows, creating a community-curated library of best practices that will grow in value over time.

