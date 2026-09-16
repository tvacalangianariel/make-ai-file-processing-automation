# Design Decisions

## AI File Processing Automation

This document records the key design decisions made while building the automation and the reasoning behind them.

The goal was to create a practical proof of concept that could reliably process incoming files while keeping the workflow understandable, modular, and extensible.

---

## 1. Use Make.com as the Orchestration Layer

### Decision

Use Make.com to coordinate the workflow between email, AI processing, file storage, and metadata logging.

### Reason

The project involves multiple services that need to exchange information.

Make.com provides a visual workflow layer that allows these services to be connected without building a custom backend for the proof of concept.

### Result

The automation can be represented as a clear sequence of:

```text
Input
  ↓
Validation
  ↓
Processing
  ↓
Storage
  ↓
Logging
  ↓
Notification
```

---

## 2. Retrieve Attachments Separately from the Email

### Decision

Use Gmail attachment retrieval rather than treating the incoming email itself as the processing object.

### Reason

The primary processing targets are the files attached to the email.

Separating the email from its attachments makes it easier to process each file independently.

---

## 3. Use an Iterator for Multiple Attachments

### Decision

Use an Iterator to process attachments individually.

### Reason

An email may contain more than one attachment.

For example:

```text
Email
├── document.pdf
├── contract.docx
└── image.jpg
```

Each file needs to be evaluated and processed independently.

### Result

The workflow can process multiple attachments without requiring a separate scenario for each file.

---

## 4. Validate File Types Before AI Processing

### Decision

Validate supported file types before sending files to the AI processing stage.

### Reason

Not every attachment is appropriate for the current workflow.

The proof of concept supports:

```text
PDF
DOCX
JPG
WEBP
```

Unsupported formats are routed separately.

### Result

Unsupported inputs do not unnecessarily enter the AI processing path.

---

## 5. Use a Router for Conditional Processing

### Decision

Use a Make.com Router to separate supported and unsupported files.

### Reason

The workflow has different behavior depending on the file type.

```text
                 Router
                /      \
               /        \
      Supported        Unsupported
          │                 │
          ▼                 ▼
        OpenAI             Gmail
```

This keeps the processing logic explicit and makes future routes easier to add.

---

## 6. Use OpenAI for AI-Powered File Analysis

### Decision

Use the OpenAI API to analyze supported documents and images.

### Reason

Traditional automation rules can identify file properties, but they cannot easily understand the content of an arbitrary document or image.

AI provides the ability to extract meaningful information from the file and return structured metadata.

### Result

The automation can produce information such as:

```text
document_type
subject
source_organization
document_date
new_filename
summary
```

---

## 7. Generate a Standardized Filename

### Decision

Use AI-generated metadata to create a standardized filename.

### Example

```text
Original:
attachment_12345.pdf

Generated:
Birth_Plan_Babylist.pdf
```

### Reason

Consistent filenames make stored documents easier for humans and downstream systems to identify.

---

## 8. Use Structured AI Output

### Decision

Use structured fields instead of relying on a free-form AI response.

### Reason

The output needs to be consumed by other automation modules.

Structured fields can be mapped directly into:

* Google Drive
* Google Sheets
* Gmail

### Result

The AI output becomes usable automation data rather than simply a text response.

---

## 9. Use Google Drive for Processed File Storage

### Decision

Store processed files in Google Drive.

### Reason

The proof of concept requires a destination for the resulting documents.

Google Drive provides a simple cloud-based storage destination that integrates directly with the automation.

---

## 10. Use Google Sheets for Metadata Logging

### Decision

Record structured processing information in Google Sheets.

### Reason

The project needs a simple way to retain metadata about processed files.

Google Sheets provides a lightweight, human-readable log that can also be extended for reporting or monitoring.

### Potential Logged Information

```text
Document Type
Subject
Source Organization
Document Date
New Filename
Summary
```

---

## 11. Use Separate Success and Unsupported Notifications

### Decision

Use different Gmail notification paths for successful and unsupported processing.

### Reason

Users should receive clear feedback about what happened to their file.

A successful processing message and an unsupported-file message represent different outcomes and should not be treated identically.

---

## 12. Add Error Handling

### Decision

Include error-handling mechanisms for appropriate downstream processing stages.

### Reason

External services can fail for reasons unrelated to the workflow logic.

Examples include:

* Temporary service failures
* File-processing failures
* API errors
* Storage failures
* Connection problems

Error handling provides a mechanism for the scenario to respond to these conditions instead of relying only on the normal success path.

---

## 13. Keep Services Modular

### Decision

Assign each external service a defined responsibility.

```text
Gmail
→ Email and attachment handling

Make.com
→ Workflow orchestration

OpenAI
→ AI file analysis

Google Drive
→ File storage

Google Sheets
→ Metadata logging
```

### Reason

Modular responsibilities make the workflow easier to understand and potentially easier to modify.

For example, Google Drive could potentially be replaced with another storage platform without changing the fundamental AI processing concept.

---

## 14. Design for Extension

### Decision

Keep the workflow structured so additional capabilities can be added later.

Potential extensions include:

```text
Additional file types
       ↓
Additional processing routes
       ↓
Human approval
       ↓
Duplicate detection
       ↓
Advanced logging
       ↓
Monitoring
```

The current implementation intentionally remains focused on the proof-of-concept scope.

---

## 15. Proof-of-Concept Scope

### Decision

Treat the project as a working proof of concept rather than a production enterprise system.

### Reason

The primary purpose of the project is to demonstrate practical AI automation capabilities.

Production deployment would require additional engineering around areas such as:

* Authentication
* Security
* Monitoring
* Scalability
* Cost management
* Retry strategies
* Data retention
* Operational support
* More extensive testing

Keeping these considerations separate prevents the portfolio project from overstating its current maturity.

---

## Summary

The overall design prioritizes:

```text
Clarity
  +
Modularity
  +
AI Integration
  +
Input Validation
  +
Error Handling
  +
Structured Output
  +
Extensibility
```

These principles allow the automation to demonstrate a practical end-to-end AI workflow while keeping the implementation understandable and maintainable.
