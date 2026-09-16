# Implementation Guide

## AI File Processing Automation with Make.com

This document describes how to implement the AI File Processing Automation using Make.com, OpenAI, Gmail, Google Drive, and Google Sheets.

The guide focuses on the workflow architecture, important configuration, data mapping, and implementation considerations rather than documenting every individual UI click.

---

# 1. Prerequisites

The following services are required.

| Service       | Purpose                       | Required |
| ------------- | ----------------------------- | -------- |
| Make.com      | Workflow automation           | Yes      |
| Gmail         | Email input and notifications | Yes      |
| OpenAI API    | AI file processing            | Yes      |
| Google Drive  | Processed file storage        | Yes      |
| Google Sheets | Metadata logging              | Yes      |

You will also need:

* A Make.com scenario
* Authorized connections for the required services
* An OpenAI API key/connection with available API usage
* A Google Drive destination folder
* A Google Sheets destination spreadsheet

---

# 2. Scenario Structure

The scenario consists of the following logical stages:

```text
Gmail
  ↓
Attachment Retrieval
  ↓
Iterator
  ↓
Router
  ├── Supported Files
  │      ↓
  │   OpenAI Upload
  │      ↓
  │   OpenAI Generate Response
  │      ↓
  │   Google Drive
  │      ↓
  │   Google Sheets
  │      ↓
  │   Gmail Success
  │
  └── Unsupported Files
         ↓
      Gmail Rejection
```

---

# 3. Module Configuration

## Module 1 — Gmail: Watch Emails

### Purpose

Monitors the configured Gmail mailbox for incoming messages that may contain files for processing.

### Key configuration

Configure:

* Gmail connection
* Target mailbox/folder
* Search/filter criteria
* Processing frequency

### Output used downstream

The module provides information such as:

* Email ID
* Sender
* Subject
* Message information
* Attachment information

---

# 4. Module 2 — Gmail: List Attachments

### Purpose

Retrieves the attachments associated with the incoming email.

This separates the email itself from the files that need to be processed.

### Important output

Each attachment provides information such as:

```text
Filename
File extension
MIME type
File data
Attachment ID
```

The exact available fields depend on the Gmail module output.

---

# 5. Module 3 — Iterator

### Purpose

Processes multiple attachments independently.

For example:

```text
Incoming Email
│
├── invoice.pdf
├── application.docx
└── document.jpg
```

The Iterator converts these attachments into individual processing bundles.

Conceptually:

```text
invoice.pdf
     ↓
Process

application.docx
     ↓
Process

document.jpg
     ↓
Process
```

This allows each file to follow the appropriate processing path.

---

# 6. Module 4 — Router

### Purpose

Separates files based on whether they are supported by the automation.

The Router creates two primary paths:

```text
                 Router
                /      \
               /        \
      Supported        Unsupported
          │                 │
          ▼                 ▼
       OpenAI              Gmail
```

---

# 7. Supported File Filter

The supported route allows the file types handled by the current proof of concept:

```text
PDF
DOCX
JPG
WEBP
```

The filter should evaluate the file information produced by the preceding attachment/Iterator stages.

### Recommended logic

Conceptually:

```text
Extension = PDF
OR
Extension = DOCX
OR
Extension = JPG
OR
Extension = WEBP
```

The exact Make.com filter expression can depend on how the attachment filename/MIME type is exposed by the connected Gmail module.

---

# 8. Unsupported File Route

Files that do not satisfy the supported-file filter follow the unsupported route.

### Purpose

Prevent unsupported files from reaching the AI processing stage.

### Result

The workflow sends a Gmail notification informing the user that the file could not be processed because its format is unsupported.

This provides explicit feedback rather than allowing an unsupported input to fail later in the workflow.

---

# 9. OpenAI: Upload a File

### Purpose

Uploads the supported file to OpenAI for use by the subsequent AI processing stage.

### Important inputs

The module receives the attachment file produced by the Iterator.

Typical values include:

```text
File name
File data
```

The filename should retain the correct extension.

Correct:

```text
document.pdf
image.jpg
image.webp
document.docx
```

Avoid passing a file without an appropriate extension when the downstream processing depends on file identification.

---

# 10. OpenAI: Generate a Response

### Purpose

Analyzes the uploaded file and generates structured information.

The AI processing stage is responsible for identifying useful document information and producing a standardized filename and summary.

### Structured output

The workflow uses the following logical fields:

```text
document_type
subject
source_organization
document_date
new_filename
summary
```

### Example

Input:

```text
Birth plan document from Babylist
```

Potential structured result:

```text
document_type:
Birth Plan

source_organization:
Babylist

new_filename:
Birth_Plan_Babylist.pdf
```

The remaining fields provide additional metadata and a concise summary.

---

# 11. AI Output Requirements

The AI response should be structured so downstream Make.com modules can reliably use the values.

The intended output concept is:

```text
{
  "document_type": "...",
  "subject": "...",
  "source_organization": "...",
  "document_date": "...",
  "new_filename": "...",
  "summary": "..."
}
```

The exact implementation format depends on the OpenAI Make.com module configuration.

### Important consideration

The AI output should be treated as automation data rather than free-form conversational text.

Structured output makes it easier to map individual fields into:

* Google Drive
* Google Sheets
* Gmail

---

# 12. Google Drive

### Purpose

Stores the processed file.

The generated filename from the AI processing stage is used when creating the stored file.

Example:

```text
Original:
attachment_12345.pdf

AI-generated:
Birth_Plan_Babylist.pdf

Google Drive:
Birth_Plan_Babylist.pdf
```

### Important mapping

The Google Drive module should receive:

```text
File data → processed attachment/file data
File name → new_filename
```

The destination folder should be configured during implementation.

---

# 13. Google Sheets

### Purpose

Records structured processing information.

A spreadsheet can contain columns corresponding to the AI output.

Example:

| Field               | Mapping               |
| ------------------- | --------------------- |
| Document Type       | `document_type`       |
| Subject             | `subject`             |
| Source Organization | `source_organization` |
| Document Date       | `document_date`       |
| New Filename        | `new_filename`        |
| Summary             | `summary`             |

Additional fields can be added for operational tracking, such as:

```text
Processing Date
Original Filename
Sender
Processing Status
```

These are optional extensions rather than requirements of the current proof of concept.

---

# 14. Gmail: Success Notification

### Purpose

Provides confirmation after successful processing.

The notification can communicate:

* Original file
* Processed filename
* Document type
* Processing status
* Summary
* Storage/logging result

The final email should be designed for easy human interpretation rather than exposing raw automation data.

---

# 15. Gmail: Unsupported File Notification

The unsupported route sends a separate notification.

The notification should identify:

* Original filename
* Unsupported file type
* Processing status
* Reason for rejection

Example:

```text
File:
example.zip

Status:
Not Processed

Reason:
Unsupported file type
```

---

# 16. Multiple Attachment Processing

The Iterator is important when an email contains more than one attachment.

Example:

```text
Email
│
├── invoice.pdf
├── contract.docx
├── photo.jpg
└── archive.zip
```

The workflow can evaluate each attachment independently:

```text
invoice.pdf    → Supported → AI Processing
contract.docx  → Supported → AI Processing
photo.jpg      → Supported → AI Processing
archive.zip    → Unsupported → Rejection
```

This allows one email to contain a mixture of supported and unsupported files.

---

# 17. Error Handling

The workflow includes error-handling considerations for downstream processing.

The purpose of error handling is to prevent an individual module failure from silently breaking the intended workflow behavior.

Potential failure points include:

* File upload
* AI processing
* File storage
* Metadata logging
* Notification

Error handlers can be configured according to the desired behavior.

Possible Make.com error-handling strategies include:

```text
Retry
Resume
Skip
Rollback
Commit
```

The appropriate strategy depends on the type of failure and whether the operation is safe to repeat.

---

# 18. Scenario Configuration Considerations

The Make.com scenario should be reviewed for appropriate operational settings.

Important settings include:

### Process data in order

Determine whether attachments need to be processed sequentially.

### Keep data confidential

Enable this when appropriate for the data being processed and the requirements of the workflow.

### Store incomplete executions

Where available and appropriate, incomplete executions can assist with troubleshooting failed scenario runs.

These settings should be selected according to the sensitivity of the data and the desired operational behavior.

---

# 19. Security Considerations

Do not store credentials directly in the GitHub repository.

Never commit:

```text
API keys
Passwords
OAuth tokens
Access tokens
Credential files
Private documents
Private email content
```

Use Make.com's connection management and the respective service authentication mechanisms instead.

For portfolio screenshots, remove or obscure sensitive information before publishing.

---

# 20. Implementation Checklist

Use the following checklist when recreating the scenario.

### Accounts and Connections

* [ ] Make.com account
* [ ] Gmail connection
* [ ] OpenAI connection/API access
* [ ] Google Drive connection
* [ ] Google Sheets connection

### Make.com Scenario

* [ ] Gmail Watch Emails
* [ ] Gmail List Attachments
* [ ] Iterator
* [ ] Router
* [ ] Supported-file filter
* [ ] Unsupported-file route
* [ ] OpenAI Upload a File
* [ ] OpenAI Generate a Response
* [ ] Google Drive
* [ ] Google Sheets
* [ ] Gmail success notification
* [ ] Gmail unsupported-file notification
* [ ] Error handling where required

### Testing

* [ ] PDF
* [ ] DOCX
* [ ] JPG
* [ ] WEBP
* [ ] Multiple attachments
* [ ] Unsupported format
* [ ] AI structured output
* [ ] Google Drive result
* [ ] Google Sheets result
* [ ] Gmail notification

---

# 21. Implementation Summary

The implementation uses Make.com as the orchestration layer and separates the workflow into clear processing stages:

```text
INPUT
Gmail
  ↓
FILES
Attachment Retrieval
  ↓
ITERATION
Iterator
  ↓
DECISION
Router
  ↓
PROCESSING
OpenAI
  ↓
OUTPUT
Google Drive + Google Sheets
  ↓
NOTIFICATION
Gmail
```

This modular design allows individual parts of the workflow to be modified or extended without redesigning the entire automation.

---

## Current Scope

The current implementation demonstrates a working proof of concept for:

* Email attachment intake
* Multiple attachment processing
* PDF processing
* DOCX processing
* JPG processing
* WEBP processing
* AI-powered metadata generation
* Automated file naming
* Google Drive storage
* Google Sheets logging
* Gmail notifications
* Unsupported-file handling

The implementation can be extended with additional file types, integrations, validation rules, monitoring, and more advanced error-handling strategies.
