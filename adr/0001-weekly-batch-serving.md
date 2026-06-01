# ADR 0001: Use Weekly Batch Serving via Data Warehouse for Churn Predictions

## Context
The B2B SaaS sales team requires a ranked list of 120,000 accounts at risk of churning, which they consume exclusively once a week on Monday mornings via a CRM dashboard. The features driving the model (e.g., billing cycles, historical support tickets, weekly app usage) are aggregated logs that change slowly over days rather than seconds, removing any requirement for instant, real-time prediction updates.

## Decision
We decided to implement a scheduled **Weekly Batch Inference Job** that executes every Sunday night. The system will pull the latest customer states from the Data Warehouse, compute predictions in bulk, and write the output back to a centralized table, which is then pushed to Salesforce using a Reverse ETL tool.

## Alternatives rejected
* **Online Request-Response API:** Considered and rejected because there is no interactive user interface or live client application requiring synchronous, low-latency scoring. Maintaining a 24/7 active API cluster would result in massive infrastructure waste with zero business utility.
* **Daily Batch Refresh Pattern:** Considered and rejected because the sales team only reviews the data on a weekly cadence. Running the aggregation and scoring pipelines daily would increase compute costs sevenfold without providing any actionable value to the end-users.

## Consequences
* **Positive:** Infrastructure and operational expenses are minimized, as we only pay for ephemeral compute resources during the Sunday night processing window.
* **Positive:** Operational complexity is drastically reduced; we completely bypass the need for API endpoints, active load balancers, auto-scaling groups, and sub-second latency monitoring.
* **Negative:** Data staleness is introduced into the workflow. If an account experiences a critical drop in usage on a Tuesday, the sales team will not see this update until the following Monday.

## Revisit if
We will reopen this decision if the product organization launches an automated, real-time in-app customer retention widget that requires instant risk scores to trigger immediate in-product interventions.

