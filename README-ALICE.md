# ALICE – Commercial Client Prospecting Agent

**AI-Powered B2B Prospecting and WhatsApp Automation**

![Project](https://img.shields.io/badge/Project-ALICE-red)
![Platform](https://img.shields.io/badge/Platform-n8n-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue)
![Database](https://img.shields.io/badge/Database-PostgreSQL-blue)
![Messaging](https://img.shields.io/badge/Messaging-WhatsApp-green)
![Architecture](https://img.shields.io/badge/Architecture-Multi--Agent-purple)

## 1. Project Overview

ALICE is an AI-powered commercial prospecting agent designed to process structured business lead databases and support B2B outreach through WhatsApp.

ALICE is part of a multi-agent architecture in which specialized agents perform distinct tasks:

- **LIZ – AI Data Prospecting Agent:** collects and organizes business prospect data, including data obtained through automated web research and scraping, subject to applicable laws, website terms, and data protection requirements.
- **ALICE – Commercial Client Prospecting Agent:** receives the structured lead database, validates contact information and authorization, prepares commercial messages, and manages the WhatsApp communication workflow.

The agents exchange data through Excel spreadsheets. LIZ prepares the spreadsheet; ALICE receives it through its XLSX upload form.

## 2. Multi-Agent Architecture

### LIZ – AI Data Prospecting Agent

LIZ is the upstream data acquisition and business intelligence agent.

Its intended responsibilities include:

- Identifying potential businesses and commercial prospects.
- Collecting and organizing relevant business information.
- Structuring lead records for downstream processing.
- Exporting prospecting data to Excel (XLSX).
- Delivering the prepared lead database to ALICE.

Data collection must follow applicable data protection laws, website terms, and the relevant legal basis for processing.

### ALICE – Commercial Client Prospecting Agent

ALICE is the commercial execution agent.

Its workflow includes:

- Receiving an Excel spreadsheet.
- Extracting spreadsheet rows.
- Validating Brazilian telephone numbers.
- Checking the explicit authorization field.
- Preparing commercial message variations.
- Using Google Gemini through an n8n AI Agent node.
- Sending image-and-text messages through Evolution API.
- Recording sending events in PostgreSQL.
- Applying configurable delays between contacts.
- Routing API errors to an error-handling branch.

ALICE is designed for unattended workflow execution after an operator uploads the spreadsheet and starts the process. Production operation still requires monitoring, operational safeguards, and recovery testing.

## 3. High-Level System Architecture

```text
       LIZ AI AGENT
             |
             v
   DATA RESEARCH / COLLECTION
             |
             v
   STRUCTURED BUSINESS LEADS
             |
             v
       EXCEL DATABASE
       (XLSX FILE)
             |
             v
       ALICE AI AGENT
             |
             v
     EXCEL EXTRACTION
             |
             v
    CONTACT VALIDATION
             |
             v
 AUTHORIZATION VERIFICATION
             |
             v
   COMMERCIAL MESSAGE FLOW
             |
             v
    IMAGE + TEXT PROCESSING
             |
             v
       EVOLUTION API
             |
             v
          WHATSAPP
             |
             v
        POSTGRESQL
             |
             v
     COMMUNICATION HISTORY
```

## 4. Large-Scale Prospecting

The architecture is intended to support large lead spreadsheets, with **20,000 contacts as a target workload**.

The current workflow processes contacts sequentially rather than sending the entire spreadsheet at once. This helps apply per-contact validation, pacing, and historical recording.

A 20,000-contact production operation must not be considered validated solely because the workflow accepts an XLSX file. Before operating at this scale, the deployment must be tested for:

- n8n execution duration and memory limits.
- Persistent checkpoints and restart recovery.
- Duplicate prevention and idempotency.
- API throughput, rate limits, and provider policies.
- Database performance and retention.
- Failed-message retry strategy.
- Monitoring, alerting, and operational intervention.
- Contact authorization and opt-out enforcement.

The workflow's configured delays are currently approximately 120–300 seconds, with additional scheduled pauses and operating-hour restrictions. At those intervals, 20,000 contacts would take many weeks. Actual throughput must be measured in a controlled test environment.

## 5. Technology Stack

| Technology | Responsibility |
|---|---|
| n8n | Workflow orchestration |
| Google Gemini | AI-assisted message generation |
| Evolution API | WhatsApp integration |
| PostgreSQL | Communication history |
| JavaScript | Contact validation and processing |
| Microsoft Excel (XLSX) | Lead database input |
| Docker | Infrastructure deployment |
| Google Drive / local storage | Campaign image source, depending on deployment configuration |

## 6. Excel Contact Database

ALICE receives a structured XLSX spreadsheet through its upload form.

Expected columns:

| Field | Description |
|---|---|
| `nome` | Business or contact name |
| `telefone` | Brazilian telephone number |
| `autorizado` | Explicit authorization indicator |
| `pais` | Country identifier; optional, defaults to BR |

The current workflow accepts Brazilian telephone numbers only. It checks the authorization field before preparing a contact for the sending path. Records without authorization or with invalid telephone numbers are excluded from sending.

**Important:** A telephone number being publicly available does not, by itself, establish permission to send marketing messages. The operator is responsible for documenting an appropriate legal basis and honoring opt-out requests.

## 7. Messaging and AI

The workflow contains a Google Gemini Chat Model connected to an n8n AI Agent. The agent is configured with the ALICE commercial persona and instructions for preparing concise, contextual B2B introductions.

The workflow also contains a JavaScript message-variation mechanism and an Evolution API request that sends an image with a caption.

**Integration validation required:** In the supplied workflow JSON, the Evolution API request uses the `mensagem` field prepared by the contact-validation code. The AI Agent output is not visibly mapped into that request. Confirm and correct this connection before claiming that AI-generated messages are used in live sends.

## 8. Image Handling

The workflow defines five campaign image filenames and includes image-rotation logic. The supplied JSON also contains Google Drive download URLs, while its sticky-note documentation describes a local Docker image library.

These two image-source approaches must be reconciled during deployment. Confirm which image source is active, verify that all five images are accessible, and test the rotation behavior before production use.

## 9. PostgreSQL History

The workflow includes a PostgreSQL node that inserts sending records into the `prospeccao_envios` table.

The recorded fields include:

- Contact name.
- Telephone number.
- Country.
- Image number.
- Message variation.
- Sending status.
- Sending timestamp.

The database schema must be created before the first execution. The PostgreSQL connection must be configured in n8n credentials; credentials must never be committed to this repository.

## 10. Installation Requirements

Required infrastructure and services:

1. A working n8n instance.
2. Google Gemini API credentials.
3. An Evolution API instance and WhatsApp connection.
4. A PostgreSQL database.
5. Docker, if using the documented container deployment.
6. Five campaign images in the configured storage location.
7. An XLSX lead spreadsheet.
8. n8n credentials for the form, Evolution API, Gemini, and PostgreSQL.

### Configuration checklist

- Import the ALICE workflow JSON into n8n.
- Configure the XLSX upload form authentication.
- Configure the Google Gemini credential.
- Configure Evolution API URL, instance, and header authentication.
- Configure the PostgreSQL credential.
- Create the required PostgreSQL table.
- Configure and verify the image storage location.
- Test with a small set of authorized contacts before any larger operation.
- Verify the AI Agent output mapping and error branches.
- Confirm opt-out handling and operational monitoring.

Do not include API keys, passwords, access tokens, private contact lists, or production credentials in Git.

## 11. Security, Privacy, and Responsible Outreach

The system must be operated in accordance with applicable data protection and electronic communications requirements, including Brazil's General Data Protection Law (LGPD – Law No. 13,709/2018), where applicable.

Operational principles:

- Establish and document an appropriate legal basis for personal-data processing.
- Record data provenance and collection purposes.
- Use contact authorization controls appropriate to the campaign and legal basis.
- Honor opt-out and do-not-contact requests.
- Protect personal information and restrict database access.
- Secure API credentials using n8n credentials or an appropriate secrets manager.
- Respect WhatsApp and provider policies, rate limits, and applicable marketing rules.
- Avoid excessive, deceptive, or unwanted communications.

## 12. Development and Validation Status

The supplied workflow JSON includes the following components:

- XLSX upload and extraction.
- Brazilian telephone validation.
- Authorization verification.
- Sequential contact processing.
- Google Gemini AI Agent and memory nodes.
- Evolution API image-and-text request.
- PostgreSQL history insertion.
- Configurable delays.
- An API error output branch.

The following items require validation or correction before production:

- AI-generated message mapping into the WhatsApp request.
- Consistency between local image-library documentation and Google Drive URL logic.
- Availability and correctness of all five campaign images.
- PostgreSQL schema and successful insertion tests.
- Error logging, retries, and restart recovery.
- Duplicate prevention and persistent checkpoints.
- Large-scale load and throughput testing.
- Opt-out enforcement and compliance review.

An HTTP success response from Evolution API indicates that the API accepted the request; it does not prove that the recipient received or read the message.

## 13. Project Vision

LIZ and ALICE form a modular multi-agent commercial intelligence workflow:

- **LIZ discovers and structures business opportunities.**
- **ALICE validates, prepares, and manages authorized commercial outreach.**

The long-term objective is to build a scalable, observable, and maintainable AI-assisted B2B prospecting operation, with clear separation of responsibilities between data acquisition and commercial communication.

## 14. Author

**Professor Thiago Rafael**  
AI Systems Architecture | Intelligent Agents | Workflow Automation | IT Infrastructure  
Brazil

---

**LIZ discovers. ALICE connects. Artificial Intelligence drives the operation.**
