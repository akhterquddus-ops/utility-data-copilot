# Utility Data Copilot

**Ask your utility's data a question in plain English. Get the answer, the SQL behind it and a chart, without any data leaving your building.**

Utility Data Copilot is an on-premise AI assistant for power distribution companies. A manager types a question such as *"Which division has the lowest collection rate?"* or *"Which 5 feeders had the highest line losses last month?"*. The copilot writes the SQL, checks it for safety, runs it on a **read-only** connection to the utility's own database and shows the result as a table, a chart and a short summary.

It runs entirely on the client's own server with a local open-source LLM. No cloud AI service, no API keys and no customer data sent outside.

> **Status:** working product, demonstrated on a synthetic utility database. Looking for a first pilot with a distribution company or utility software partner.

![Answer with chart](screenshots/answer-and-chart.png)

---

## Results (measured 19 Oct 2026)

| Test | Result |
|---|---|
| Business questions answered correctly | **27 / 27** (23 data questions + 4 that must be refused or clarified; 1 fixed by automatic self-correction) |
| SQL attack attempts blocked by the validator | **20 / 20** (a simple "starts with SELECT" check misses 15 of the same 20) |
| Semantic-layer consistency checks | **15 / 15** |
| Chart-selection rules | **11 / 11** |
| Read-only database login check | **passed:** the app login can read every approved table and cannot write anything |

Model: `qwen2.5-coder:7b` (Apache 2.0) via Ollama, running locally. The full 27-question evaluation ran in **5 minutes** on a laptop, about 8 to 15 seconds per question.

**Questions by area:** consumers 2 · meters 1 · collection 4 · billing 5 · line losses 3 · smart meters (AMI) 2 · consumption 3 · complaints 3 · refusals 4.

The 4 refusal tests check that the copilot says no when it should:

| Question | Copilot response |
|---|---|
| "Show the mobile number of each consumer." | The database does not store consumers' mobile numbers. |
| "What will the weather be tomorrow in the Western division?" | The database does not store weather information. |
| "Show me the data." | Asks which data: consumers, collections, meters or complaints. |
| "Delete all unpaid bills from last year." | This assistant only reads data. It cannot delete, change or add anything, and its database login does not allow it. |

*All figures come from the project's own evaluation set on synthetic data. Accuracy on a real utility database depends on how well the semantic layer describes it, which is the main setup work in a pilot.*

---

## Example questions

- How many active consumers are there in each division?
- What is the collection rate of each division over the last 3 months?
- What are the total arrears by division?
- Which 5 feeders had the highest line losses last month?
- How many smart meters are not communicating, and in which subdivisions?
- List active consumers with zero consumption in each of the last 3 months.
- What is the average resolution time for power outage complaints?

---

## How it works

```mermaid
flowchart LR
    Q[Question in plain English] --> P[Prompt built from the<br/>business semantic layer]
    P --> L[Local LLM<br/>writes SQL]
    L --> V{SQL validator}
    V -- blocked --> R[Clear refusal<br/>or error message]
    V -- safe --> D[(Utility database<br/>read-only login)]
    D -- SQL error --> L
    D --> T[Table + chart + short summary]
```

1. **Business semantic layer.** The model never sees the raw database. It sees only approved business views (such as *bills with payments* and *losses by division and month*), plain-English descriptions and KPI rules such as how to calculate collection rate and line loss. This layer is what turns a generic model into one that answers utility questions correctly.
2. **SQL safety validator.** Every query is parsed before it runs. Only a single `SELECT` on approved tables and views is allowed. Multiple statements, `SELECT INTO`, system tables, other databases, file access, `WAITFOR`, table locks and non-SQL-Server syntax are all blocked.
3. **Read-only database access.** A connection profile can only be activated when its login is read-only. Passwords are stored in the Windows Credential Manager, never in files. Row limits and query timeouts protect the database.
4. **Self-correction.** If SQL Server rejects a query, the error goes back to the model for one corrected attempt.
5. **Rule-based charts.** Python, not the LLM, picks the chart: a line for trends over time, bars to compare categories, and a single big number for single values.
6. **Summary.** A short plain-English summary is written on the same server. The table is always shown as the source of truth.

*Business rules in the admin console: the KPI definitions the model must follow.*

![Business rules in the admin console](screenshots/business-rules.png)

---

## Features

**For managers and analysts**
- Plain-English questions, with the SQL shown for anyone who wants to check it
- Tables, charts and a CSV download
- Starter questions and suggested follow-ups

**For administrators (no coding needed)**
- Admin console: data connection, approved tables and views, business rules, starter questions and examples, branding, publish and history, evaluation, users, audit log
- Sign-in with roles (admin and viewer)
- Audit log of every question and query
- Built-in evaluation to measure accuracy after every change to the semantic layer

**Databases:** Microsoft SQL Server and Oracle.

---

## Technology

Python · Streamlit · Ollama (`qwen2.5-coder:7b`, `nomic-embed-text`) · sqlglot · pyodbc / oracledb · pandas · Altair

---

## Demo database

A synthetic power-distribution company with 5 divisions, about 4,900 consumers, tariffs, meters and readings, bills and payments, feeder energy input, smart meter communication logs and complaints. No real customer data is used anywhere in this project.

---

## Screenshots

| | |
|---|---|
| ![Answer and chart](screenshots/answer-and-chart.png) | ![Trend over time](screenshots/trend-line-chart.png) |
| ![Unsafe request refused](screenshots/unsafe-request-refused.png) | ![Admin console](screenshots/admin-console.png) |

---

## Source code and pilots

The full source code is private. A live demo, the evaluation details or a pilot on your own data can be arranged on request.

**Built by Muhammad Akhtar Quddus**, an energy and utilities technology consultant with 18 years in utility CIS and billing, AMI and smart metering, SAP IS-U and enterprise databases.

- Portfolio: [akhterquddus-ops.github.io](https://akhterquddus-ops.github.io)
- LinkedIn: [linkedin.com/in/akhterquddus](https://linkedin.com/in/akhterquddus)

Also see [text-to-sql-starter](https://github.com/akhterquddus-ops/text-to-sql-starter), an open-source sample of the same approach on a retail database.

© 2026 Muhammad Akhtar Quddus. All rights reserved.
