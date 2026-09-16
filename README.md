# AI File Processing Automation with Make.com

> **Working Proof of Concept | AI Automation Portfolio Project**

An AI-powered file processing workflow built with **Make.com** that automatically receives email attachments, validates supported file types, processes documents and images with the **OpenAI API**, generates structured metadata, stores processed files in **Google Drive**, records information in **Google Sheets**, and sends automated email notifications.

The project demonstrates practical **workflow automation, AI/API integration, file processing, data handling, routing, and error handling**.

---

## Overview

Many document-processing workflows involve repetitive manual tasks such as:

* Receiving files through email
* Identifying file types
* Renaming documents
* Extracting useful information
* Organizing files
* Recording document information
* Notifying users about processing results

This project automates that workflow using Make.com and AI.

### High-Level Workflow

```text
Gmail
  │
  ▼
Retrieve Attachments
  │
  ▼
Validate File Type
  │
  ├─────────────── Supported ──────────────┐
  │                                        │
  │                                        ▼
  │                                  OpenAI Processing
  │                                        │
  │                                        ▼
  │                                Structured Metadata
  │                                  │             │
  │                                  ▼             ▼
  │                            Google Drive   Google Sheets
  │                                  │
  │                                  ▼
  │                            Gmail Notification
  │
  └──────────── Unsupported ───────────────► Rejection Notification
```

---

## Problem

Processing incoming email attachments manually can require several repetitive steps:

1. Download the attachment.
2. Determine whether the file can be processed.
3. Identify the type and purpose of the document.
4. Rename the file consistently.
5. Store the file in the appropriate location.
6. Record useful metadata.
7. Notify the appropriate recipient.

When multiple attachments arrive in a single email, these tasks become even more repetitive.

---

## Solution

This automation creates an end-to-end file processing pipeline.

When an email containing attachments arrives, the workflow:

1. Detects the incoming email.
2. Retrieves its attachments.
3. Processes multiple attachments independently.
4. Validates the file type.
5. Routes supported and unsupported files separately.
6. Sends supported files to OpenAI for AI processing.
7. Generates structured document metadata.
8. Creates a standardized filename.
9. Stores the processed file in Google Drive.
10. Records processing information in Google Sheets.
11. Sends an automated processing result through Gmail.
12. Rejects unsupported files through a separate notification path.

---

## Key Capabilities

* 📧 Automated email attachment processing
* 📎 Multiple attachment handling
* 🔀 Supported/unsupported file routing
* 🤖 AI-powered document and image processing
* 🏷️ Automated file naming
* 📄 PDF and DOCX processing
* 🖼️ JPG and WEBP image processing
* ☁️ Google Drive file storage
* 📊 Google Sheets metadata logging
* 📬 Automated Gmail notifications
* ⚠️ Unsupported-file handling
* 🧪 Proof-of-concept testing

---

## Supported File Types

| File Type                 | Processing  |
| ------------------------- | ----------- |
| PDF                       | ✅ Supported |
| DOCX                      | ✅ Supported |
| JPG                       | ✅ Supported |
| WEBP                      | ✅ Supported |
| Other/Unsupported Formats | ❌ Rejected  |

The workflow validates files before sending them to the appropriate processing path.

---

## AI Processing

The OpenAI API is used to analyze supported files and return structured information.

The workflow extracts or generates fields including:

| Field                 | Purpose                                                |
| --------------------- | ------------------------------------------------------ |
| `document_type`       | Identifies the general type of document                |
| `subject`             | Identifies the document subject                        |
| `source_organization` | Identifies the originating organization when available |
| `document_date`       | Identifies the relevant document date                  |
| `new_filename`        | Generates a standardized filename                      |
| `summary`             | Provides a concise document summary                    |

### Example

A document such as a Babylist birth plan can produce metadata similar to:

```text
Document Type: Birth Plan
Source Organization: Babylist
New Filename: Birth_Plan_Babylist.pdf
Summary: ...
```

The structured output can then be passed to downstream automation steps.

---

## Automation Flow

### 1. Monitor Gmail

The workflow monitors Gmail for incoming messages containing attachments.

### 2. Retrieve Attachments

Attachments are retrieved from the incoming email.

### 3. Process Multiple Attachments

An Iterator allows attachments to be processed independently instead of treating the email as a single file-processing operation.

### 4. Validate and Route Files

The workflow determines whether each attachment belongs to a supported file type.

Supported files continue to AI processing.

Unsupported files follow a rejection/notification path.

### 5. Process with OpenAI

Supported documents and images are sent to the OpenAI processing stage.

The AI generates structured metadata for downstream modules.

### 6. Store the File

Processed files are stored in Google Drive using the generated filename.

### 7. Record Metadata

Important processing information is recorded in Google Sheets.

### 8. Send the Result

Gmail sends an automated notification containing the processing result.

---

## Architecture

The automation is organized into several logical stages:

```text
┌─────────────────────┐
│        Gmail        │
│   Incoming Email    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  List Attachments   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Iterator       │
│ Process Each File   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   File Validation   │
└──────────┬──────────┘
           │
       ┌───┴────┐
       │        │
       ▼        ▼
  Supported  Unsupported
       │        │
       ▼        ▼
   OpenAI     Gmail
       │      Rejection
       ▼
Structured Output
       │
   ┌───┴─────────┐
   ▼             ▼
Google Drive  Google Sheets
   │
   ▼
 Gmail Result
```

For a detailed explanation of the architecture, see:

**[Architecture Documentation](docs/architecture.md)**

---

## Proof of Concept

The automation was implemented and tested in Make.com.

![Workflow Overview](screenshots/workflow-overview.png)

The proof of concept successfully demonstrates:

- PDF processing
- DOCX processing
- JPG processing
- WEBP processing
- Multiple attachment handling
- Unsupported file handling
- AI-generated structured output
- Google Drive storage
- Google Sheets logging
- Gmail notifications

[View detailed testing documentation](docs/testing.md)

---

## Error and Unsupported File Handling

The workflow does not assume that every incoming attachment can be processed.

Unsupported files are routed away from the AI processing path and handled through a separate notification flow.

This prevents unsupported inputs from unnecessarily reaching downstream processing modules.

Additional implementation and design decisions are documented in:

**[Design Decisions](docs/decisions.md)**

---

## Technology Stack

| Technology        | Purpose                                        |
| ----------------- | ---------------------------------------------- |
| **Make.com**      | Workflow automation and orchestration          |
| **OpenAI API**    | AI-powered file analysis and structured output |
| **Gmail**         | Email input, attachments, and notifications    |
| **Google Drive**  | Processed file storage                         |
| **Google Sheets** | Metadata and processing records                |

---

## Project Structure

```text
make-ai-file-processing-automation/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── implementation.md
│   ├── testing.md
│   └── decisions.md
│
├── screenshots/
│
├── sample-data/
│   └── README.md
│
└── .gitignore
```

### Documentation

| Document                                 | Purpose                                    |
| ---------------------------------------- | ------------------------------------------ |
| [Architecture](docs/architecture.md)     | System architecture and data flow          |
| [Implementation](docs/implementation.md) | Implementation and configuration guide     |
| [Testing](docs/testing.md)               | Test cases and proof-of-concept results    |
| [Design Decisions](docs/decisions.md)    | Important technical and workflow decisions |

---

## Security & Privacy

This repository should contain only safe demonstration material.

Do not commit:

* API keys
* Passwords
* OAuth tokens
* Credentials
* Private documents
* Personal email content
* Personally identifiable information
* Confidential business information

Screenshots should be reviewed before publication to ensure sensitive information is not exposed.

---

## Lessons Learned

This project provided practical experience with:

* Designing multi-step automation workflows
* Connecting multiple SaaS platforms
* Integrating an AI API into an automation
* Processing different file types
* Handling multiple attachments
* Routing different processing conditions
* Working with structured AI output
* Passing data between automation modules
* Designing unsupported-input handling
* Testing automation behavior across multiple scenarios

---

## Future Improvements

Potential future improvements include:

* Additional document formats
* More advanced document classification
* Improved confidence/error reporting
* Human approval workflows
* Duplicate-file detection
* Enhanced logging
* Retry mechanisms
* Additional storage and notification integrations
* More sophisticated document routing

---

## Project Status

**Completed — Working Proof of Concept**

The automation has been built and tested in Make.com.

This repository documents the project architecture, implementation approach, testing results, and design decisions.

---

## Author

**Ariel Calangian**

AI Automation Portfolio Project
