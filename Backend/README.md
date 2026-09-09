# Automation Manager Backend

> 📖 **Xem tài liệu kiến trúc toàn diện:** [**BACKEND_SYSTEM_ARCHITECTURE.md**](file:///d:/Project/AutomationUi/Backend/BACKEND_SYSTEM_ARCHITECTURE.md) (Bao gồm Sơ đồ Mermaid, Phân rã Micro-Engine, Distributed Redis Task Queue, SSE Stream, RBAC Security và Database Schema).
>
> 🛡️ **Xem tài liệu Bản quyền & Đóng gói Bàn giao Khách hàng:** [**DEPLOYMENT_AND_LICENSE_GUIDE.md**](file:///d:/Project/AutomationUi/Backend/docs/DEPLOYMENT_AND_LICENSE_GUIDE.md) (Kiến trúc RSA-2048, HWID Fingerprint, Redis Cluster Sync, ProGuard Obfuscation & Customer Delivery Package).

This document outlines the database schema and API endpoints for the Automation Manager project.

## Database Structure

### 1. `projects`

This table stores information about the automation projects.

| Column      | Type          | Description                  |
|-------------|---------------|------------------------------|
| `id`        | `BIGINT` (PK) | Auto-incrementing primary key. |
| `name`      | `VARCHAR`     | The name of the project.     |
| `description` | `TEXT`        | A description of the project.|

### 2. `json_test_cases`

This table contains the test cases. Each test case is associated with a project and its steps are stored in a single JSON field.

| Column          | Type          | Description                                                  |
|-----------------|---------------|--------------------------------------------------------------|
| `id`            | `BIGINT` (PK) | Auto-incrementing primary key.                               |
| `project_id`    | `BIGINT` (FK) | Foreign key referencing the `projects` table.                |
| `name`          | `VARCHAR`     | The name of the test case.                                   |
| `raw_json_data` | `CLOB` / `TEXT` | A JSON string containing an array of all test steps.         |

### Relationship

A `Project` can have multiple `Test Cases`. The relationship is one-to-many.

```
+-----------+       +--------------------+
|  projects |       |  json_test_cases   |
+-----------+       +--------------------+
| id (PK)   |-------| id (PK)            |
| name      |       | project_id (FK)    |
| description|      | name               |
+-----------+       | raw_json_data      |
                    +--------------------+
```

### JSON Structure for `raw_json_data`

The `raw_json_data` column stores a JSON array where each object represents a single step in the test case. The structure of each object depends on the `platformType`.

#### Example: Web Step (`WebStepModel`)

```json
{
  "stepOrder": 1,
  "stepName": "Navigate to Google",
  "platformType": "WEB",
  "keyword": "NAVIGATE",
  "locatorType": null,
  "locatorValue": null,
  "testData": "https://www.google.com"
}
```

#### Example: API Step (`ApiStepModel`)

```json
{
  "stepOrder": 1,
  "stepName": "Get T24 Date",
  "platformType": "API",
  "keyword": "SEND_REQUEST",
  "apiMethod": "POST",
  "apiUrl": "http://localhost:8080/api/mock/t24",
  "apiHeaders": { "Content-Type": "application/json" },
  "apiBody": "{\\"serviceRef\\":\\"today_t24\\",...}",
  "extractions": [
    {
      "jsonPath": "$.data.T24CurrentDate",
      "contextKey": "T24CurrentDate"
    }
  ]
}
```

#### Example: Sub-Test Case Step (`SubTestCaseStepModel`)

```json
{
  "stepOrder": 2,
  "stepName": "Call sub-test case",
  "platformType": "TESTCASE",
  "keyword": "CALL_SUB_TESTCASE",
  "subTestCaseId": 1
}
```

## API Endpoints

### Trigger Automation

This endpoint triggers the execution of a specific test case by its ID.

* **URL:** `/api/automation/run/{id}`
* **Method:** `POST`
* **URL Params:** `id=[integer]` (the ID of the test case to run)
* **Success Response:**
  * **Code:** 200 OK
  * **Content:** `Thực thi kịch bản '[TestCaseName]' thành công!`
* **Error Response:**
  * **Code:** 500 Internal Server Error
  * **Content:** `Lỗi thực thi: [ErrorMessage]`

### Reload Database Configurations

This endpoint reloads the database connection configurations from the JSON file.

* **URL:** `/api/automation/db-connections/reload`
* **Method:** `POST`
* **Success Response:**
  * **Code:** 200 OK
  * **Content:** `✅ Đã nạp lại danh sách cấu hình kết nối mới từ file JSON thành công!`
