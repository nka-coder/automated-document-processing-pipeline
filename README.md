# Document Extraction Pipeline

## 1. Ingestion & Event Schema
* **Source:** Cloud Object Storage (Invoices, Receipts, Forms).
* **Mechanism:** Event-driven architecture utilizing a **Blob-Triggered** event pattern.
* **Technical Detail:** Upon file upload, an event notification is published to the Serverless Function, ensuring zero-latency processing without the need for resource-heavy polling of the storage container.

## 2. Orchestration & Compute (Python SDK)
* **Runtime:** Serverless Python Environment.
* **Process Flow:**
    * **Initialization:** The Function receives the event and initializes the `DocumentIntelligenceClient` via the Azure Python SDK.
    * **Async Operations:** Uses `begin_analyze_document` to start the extraction process, allowing the serverless worker to handle long-running tasks asynchronously.
    * **Lifecycle Management:** Manages the polling loop to retrieve the structured payload once the operation status reaches `Succeeded`.

## 3. Extraction & Logic Layer
* **Models:** Utilizes a mix of **Prebuilt Models** (for standard invoices/receipts) and **Customized Models** (for proprietary business forms).
* **Data Validation:**
    * **Confidence Scoring:** Implements a logic gate where any extraction with a confidence score $< 0.80$ is flagged.
    * **Business Rules:** Custom Python logic validates entity integrity (e.g., date formatting, mathematical verification of line items).

## 4. Persistence & Downstream Integration
* **Data Transformation:** Normalizes raw JSON output into a schema-validated format.
* **Integration:** Direct injection of structured data into **Backend Systems (ERP/CRM/NoSQL)**.
* **Error Handling:** Implements a **Dead Letter Queue (DLQ)** for documents that fail validation or schema requirements to ensure pipeline reliability.
