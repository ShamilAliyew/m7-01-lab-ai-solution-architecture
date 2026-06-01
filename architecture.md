# AI Solution Architecture: Scenario B (Weekly Churn Predictions)

## 1. Executive Summary & Serving Pattern Justification
For the B2B SaaS Churn Prediction system, we have selected a **Batch (Offline) Serving Pattern**. 

To justify this choice, we look at the core question: **"Who or what is waiting on the prediction, and for how long?"**
* **The Consumer:** The sales team reads the ranked churn risk list once a week on Monday morning to plan their outreach. No user or live system is waiting for a real-time response on a screen.
* **Data Stability:** Account history, billing cycles, and customer support tickets change on a daily/weekly cadence, not a millisecond cadence.
* **The Three Knobs Trade-off:** * **Latency:** Relaxed. Predictions can take hours to compute on Sunday night without impacting the product experience.
  * **Throughput:** ~120,000 accounts processed concurrently once a week.
  * **Cost:** Minimised. By choosing batch, we treat Cost as our primary efficiency budget. We completely avoid the need for expensive, 24/7 idle GPU/CPU online inference servers.

---

## 2. System Architecture Diagram

```mermaid
graph TD
    %% Boundaries
    subgraph Offline_Data_Layer [1. Data & Storage Layer]
        A[Data Sources: App Logs, Stripe, Zendesk] -->|Raw ETL Log| B[(Data Warehouse: Snowflake/BigQuery)]
        B -->|Weekly Aggregation via dbt| C[(Offline Feature Store)]
    end

    subgraph ML_Lifecycle_Layer [2. ML Lifecycle & Registry]
        C -->|Historical Features| D[Training Pipeline]
        D -->|Register Model vX| E[Model Registry: MLflow]
    end

    subgraph Serving_Layer [3. Batch Inference Serving]
        C -->|Current Week Features| F[Batch Inference Job <br> Every Sunday Night]
        E -->|Pull Active Production Model| F
        F -->|Write Predictions| G[(DWH: Prediction Table)]
    end

    subgraph Consumer_Monitoring_Layer [4. Consumers & Feedback Loop]
        G -->|Reverse ETL: Hightouch/Census| H[CRM Dashboard: Salesforce]
        H -->|Weekly Outreach Playbooks| I[Sales Team]
        H -->|Actual Churn Events| J[Monitoring & Feedback Loop]
        J -->|Ground Truth Labels| B
    end

    %% Styles
    style F fill:#bbf,stroke:#333,stroke-width:2px
    style E fill:#f9f,stroke:#333,stroke-width:2px
    style H fill:#bfb,stroke:#333,stroke-width:2px