# Architecture

## AI File Processing Automation

This document describes the architecture and data flow of the AI File Processing Automation built with Make.com.

---

## 1. Architecture Overview

The automation uses Make.com as the orchestration layer between Gmail, OpenAI, Google Drive, and Google Sheets.

The workflow follows this general pattern:

```text
Email Input
    ↓
Attachment Retrieval
    ↓
Attachment Iteration
    ↓
File Validation / Routing
    ↓
AI Processing
    ↓
Structured Metadata
    ↓
Storage + Logging
    ↓
Notification
```

---

## 2. System Architecture

```text
                         ┌──────────────────┐
                         │      Gmail       │
                         │ Incoming Email   │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ List Attachments │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │     Iterator     │
                         │  Process Files   │
                         │  Independently   │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ File Validation  │
                         │    / Routing     │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             SUPPORTED FILES             UNSUPPORTED FILES
                    │                           │
                    ▼                           ▼
             ┌──────────────┐           ┌──────────────┐
             │    OpenAI    │           │    Gmail     │
             │ AI Processing│           │  Rejection   │
             └──────┬───────┘           └──────────────┘
                    │
                    ▼
             ┌──────────────┐
             │  Structured  │
             │    Output    │
             └──────┬───────┘
                    │
             ┌──────┴───────┐
             │              │
             ▼              ▼
      ┌─────────────┐ ┌─────────────┐
      │Google Drive │ │Google Sheets│
      │File Storage │ │   Logging   │
      └──────┬──────┘ └─────────────┘
             │
             ▼
      ┌─────────────┐
      │    Gmail    │
      │Notification │
      └─────────────┘
```

---

## 3. Component Responsibilities

### Gmail

Gmail provides both the workflow input and the final notification channel.

Responsibilities:

* Detect incoming emails
* Provide attachments
* Send processing results
* Send unsupported-file notifications

---

### Attachment Retrieval

The attachment retrieval stage obtains files from the incoming email so they can be processed individually.

---

### Iterator

The Iterator allows multiple attachments within the same email to be handled independently.

For example:

```text
Email
 ├── document1.pdf
 ├── document2.docx
 └── image1.jpg
```

becomes separate processing items:

```text
document1.pdf → Process
document2.docx → Process
image1.jpg    → Process
```

This makes the workflow capable of handling emails containing multiple files.

---

### File Validation and Routing

The routing stage determines how each attachment should be handled.

Supported file types:

```text
PDF
DOCX
JPG
WEBP
```

Unsupported formats are directed to a separate handling path.

This prevents unsupported files from entering the AI-processing workflow.

---

### OpenAI

OpenAI provides the AI processing capability.

The AI receives supported files and produces structured information that can be consumed by downstream modules.

The primary output fields are:

```text
document_type
subject
source_organization
document_date
new_filename
summary
```

---

### Google Drive

Google Drive provides persistent storage for processed files.

The workflow uses the AI-generated filename when creating the stored file.

Example:

```text
Birth_Plan_Babylist.pdf
```

---

### Google Sheets

Google Sheets acts as a lightweight processing log.

The structured AI output can be recorded for later reference, tracking, or reporting.

---

### Gmail Notification

The final Gmail stage communicates the processing result.

This provides confirmation that the automation completed its intended processing flow.

---

## 4. Data Flow

The primary data flow is:

```text
Email
  ↓
Attachment
  ↓
File Type
  ↓
OpenAI
  ↓
Structured Metadata
  ↓
Google Drive + Google Sheets
  ↓
Email Notification
```

The unsupported-file flow is:

```text
Email
  ↓
Attachment
  ↓
File Type
  ↓
Unsupported
  ↓
Rejection Notification
```

---

## 5. Supported Processing Paths

### Document Path

```text
PDF / DOCX
     ↓
Validation
     ↓
OpenAI
     ↓
Structured Metadata
     ↓
Google Drive
     ↓
Google Sheets
     ↓
Gmail
```

### Image Path

```text
JPG / WEBP
     ↓
Validation
     ↓
OpenAI
     ↓
Structured Metadata
     ↓
Google Drive
     ↓
Google Sheets
     ↓
Gmail
```

---

## 6. Design Principles

The workflow was designed around several principles:

### Separate Validation from Processing

Files are validated before entering AI processing.

### Process Attachments Independently

Each attachment is treated as an individual processing item.

### Use Structured AI Output

Structured fields allow AI results to be passed consistently into downstream modules.

### Separate Successful and Unsupported Processing

Unsupported files should not continue through the normal processing pipeline.

### Keep External Services Modular

Gmail, OpenAI, Google Drive, and Google Sheets each have a defined responsibility within the workflow.

---

## 7. Current Scope

The current proof of concept supports:

* Gmail attachment intake
* Multiple attachment processing
* PDF processing
* DOCX processing
* JPG processing
* WEBP processing
* AI metadata extraction
* Automated file naming
* Google Drive storage
* Google Sheets logging
* Gmail notifications
* Unsupported-file handling

---

## 8. Future Architecture Opportunities

The architecture can be extended with:

* Additional file formats
* Human approval steps
* Retry mechanisms
* Duplicate detection
* More advanced classification
* Additional storage providers
* Database logging
* Monitoring dashboards
* Confidence-based routing
* Human-in-the-loop review
