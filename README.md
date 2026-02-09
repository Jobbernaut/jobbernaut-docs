# Jobbernaut Ecosystem v1.0 Alpha - Proposed System Architecture

```mermaid
flowchart TD
    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b
    classDef gateway fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100
    classDef backend fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20
    classDef database fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c
    classDef director fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,stroke-dasharray: 5 5,color:#fbc02d

    subgraph ClientLayer [Client Layer]
        TabsFE[Jobbernaut Tabs Frontend]:::frontend
        Extract[Jobbernaut Extract]:::frontend
    end

    subgraph EntryLayer [Entry & Routing]
        ReDirector[Jobbernaut ReDirector]:::gateway
    end

    subgraph CoreMgmt [Core Management Sync]
        TabsBE[Jobbernaut Tabs Backend]:::backend
        TabsDB[(Jobbernaut Tabs Database)]:::database
    end

    subgraph AsyncWorkflow [Orchestration & Intelligence]
        Director[Jobbernaut Director]:::director
        TailorMain[Jobbernaut Tailor Main]:::backend
        TailorRender[Jobbernaut Tailor Render]:::backend
    end

    subgraph StorageLayer [Storage Layer]
        Warehouse[(Jobbernaut Warehouse)]:::database
    end

    %% Connections
    TabsFE -->|POST /jobs| ReDirector
    Extract -->|POST /jobs| ReDirector
    ReDirector -->|Route Request| TabsBE
    
    TabsBE -->|1. CRUD Operations| TabsDB
    TabsBE -->|2. Generate Presigned URL| Warehouse
    TabsBE -.->|3. Trigger Workflow| Director

    Director -->|State 1: Context & AI| TailorMain
    TailorMain -->|Fetch Context| TabsDB
    TailorMain -->|Return JSON| Director
    
    Director -->|State 2: Render PDF| TailorRender
    TailorRender -->|Upload Final PDF| Warehouse
    TailorRender -->|Return S3 Key| Director
    
    Director -->|State 3: Mark Complete| TabsDB
```
---

# Jobbernaut Ecosystem v1.0 Alpha - Stack Definition

| Component Name | Role | Technology Stack |
| :--- | :--- | :--- |
| **Jobbernaut Tabs Frontend** | User Interface (Web) | **Next.js** (React), Tailwind CSS. Hosted on Vercel or AWS S3 + CloudFront. |
| **Jobbernaut Extract** | Data Collection | **Chrome Extension** (Manifest V3), Vanilla JS / React. |
| **Jobbernaut ReDirector** | API Gateway | **AWS API Gateway** (HTTP API). Routes traffic and handles CORS. |
| **Jobbernaut Tabs Backend** | Core Logic (Sync) | **AWS Lambda** (Python 3.12). Handles CRUD, Validation, and Presigned URLs. |
| **Jobbernaut Tabs Database** | Data Persistence | **AWS DynamoDB** (On-Demand). Single-table design for Jobs and Users. |
| **Jobbernaut Warehouse** | File Storage | **AWS S3** (Standard). Stores Resumes, Cover Letters, and Source PDFs. |
| **Jobbernaut Director** | Orchestration | **AWS Step Functions**. Manages the async workflow state and retries. |
| **Jobbernaut Tailor Main** | Intelligence Microservice | **AWS Lambda** (Docker Image). Python 3.12, **LangGraph**, OpenAI API. |
| **Jobbernaut Tailor Render** | Render Microservice | **AWS Lambda** (Docker Image). Python 3.12, **TeXLive** (LaTeX), Jinja2. |
