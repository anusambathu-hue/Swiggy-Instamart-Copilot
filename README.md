# Customer Intelligence & Support Copilot 

This project simulates a customer support and business intelligence system for an online grocery platform. It uses synthetic customer and order data to analyze customer behavior, predict customer inactivity, automate ticket routing, and answer policy-related questions.

The project is designed for learning purposes and does not use any real customer data or external AI services.

## Objectives

The project demonstrates how to:

Load and clean customer and order data.
Store cleaned data in a SQLite database.
Perform customer analytics using SQL and Python.
Segment customers based on purchasing behavior.
Predict customers who may stop ordering.
Protect customer privacy through data masking.
Automatically classify support tickets.
Retrieve company policies using a simple Retrieval-Augmented Generation (RAG) approach.
Process refund requests using predefined business rules.

## Setup and run
Python 3.11+ recommended.

```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# macOS/Linux:
# source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python generate_data.py
python run_project.py
```

Open `notebooks/Customer_Intelligence_Support_Copilot.ipynb` and run cells from top to bottom. The notebook calls the same modules used by the scripts.

## Repository map
- `generate_data.py`: deterministic fictional CSV and policy generation.
- `mock_llm.py`: offline stand-in for classification, policy answers and agent tool calls.
- `src/data_pipeline.py`: audit, clean, SQLite load, validation.
- `src/customer_models.py`: RFM, K-means, PCA, lapse model with train/test separation.
- `src/triage.py`: token guardrails, PII redaction, retries, response validation, routing.
- `src/policy_rag.py`: chunking, TF-IDF retrieval and grounded policy evaluation.
- `src/refund_agent.py`: order lookup, refund calculation, ledger and bounded tool loop.
- `sql/queries.sql`: seven SQL learning patterns.
- `outputs/`: generated artifacts; `state/`: agent checkpoint examples.

## How students should learn the data-fetching flow
1. `pd.read_csv("data/raw/orders.csv", keep_default_na=False)` loads local export rows. In a production project, a fetch function might download from an approved endpoint or query a warehouse, but this exercise intentionally uses local synthetic data and has no credentials.
2. Inspect shape, data types, null/blank counts, duplicate rows, keys and impossible values before cleaning.
3. Apply one documented cleaning rule at a time and compare row/value counts before and after.
4. Write clean CSVs for easy inspection and SQLite tables for repeatable SQL queries.
5. Run assertions before downstream modelling. A failed assertion should stop the pipeline rather than silently propagate bad data.

## Part A — Data foundation
The data pipeline cleans and standardizes the data by making city and gender values consistent, treating invalid ages as missing, removing duplicate orders while keeping the latest record, filling missing order amounts using product prices, and correcting delivery time information. Missing delivery times are estimated only for completed orders within the same product category. All cleaning rules should be checked carefully and validated using the capstone's expected results.

## Part B — Segments and lapse
RFM analysis is performed using completed orders and a fixed reference date. Customer segmentation is done using K-Means clustering, where different cluster values (K=2 to K=8) are evaluated, and K=4 is chosen for easier interpretation. For lapse prediction, only orders before September 1, 2025 are used as features, while customer activity during September is used as the target outcome. The model is built using a preprocessing pipeline, GridSearchCV for parameter tuning, and a saved Joblib model. The Premium customer retention list is provided for demonstration purposes only because the Premium label is calculated using the full RFM period and may include future information.

## Part C — Ticket triage and privacy
Support tickets that are empty or too long are ignored before processing. Sensitive information such as email addresses, phone numbers, and order IDs is removed to protect customer privacy. The system automatically retries if temporary errors occur. Ticket categories must match a predefined list of valid categories. If the output is unclear or invalid, the ticket is sent for manual review. Fraud-related tickets are routed to the security team, premium customer tickets go to the premium support desk, and all other tickets are sent to the general support queue.

## Part D — Policy retrieval and refund tools
Policy documents are divided into smaller sections and converted into searchable text using TF-IDF. The most relevant sections are identified using similarity scoring. The system then generates responses based on the retrieved policy information. Refund requests are checked against rules stored in a SQLite database. Orders delivered more than 45 minutes late qualify for a full refund, while refunds above INR 1000 require manual review. This project is a learning simulation and does not process real payments.

## GitHub suggestions
These public repositories/documentation projects are useful for learning concepts; do not copy a student's capstone solution. Adapt the patterns and cite the source in your notes:
- pandas documentation: https://github.com/pandas-dev/pandas — CSV loading, cleaning, groupby, pivot tables.
- scikit-learn: https://github.com/scikit-learn/scikit-learn — pipelines, preprocessing, clustering, model selection, metrics and TF-IDF.
- Python tutorial: https://github.com/python/cpython/tree/main/Doc/tutorial — Python fundamentals.
- Jupyter Notebook: https://github.com/jupyter/notebook — notebook workflow.
- SQLite: https://github.com/sqlite/sqlite — SQL database engine (official mirror).
- scikit-learn text feature extraction guide: https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction.

## Limitations and next steps
## Limitations
This project is designed for learning purposes and may require some adjustments to match all grading requirements.
The mock AI model follows predefined rules and does not behave like a real Large Language Model (LLM).
Refund records are stored locally and do not connect to a real payment system.

## Future Improvements

A production-ready system could include:

User authentication and access control
Secure storage of passwords and API keys
System monitoring and logging
Data quality and schema validation
Data tracking and audit history
Model performance monitoring
Human approval workflows for critical decisions
