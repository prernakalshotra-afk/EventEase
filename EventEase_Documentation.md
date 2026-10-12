# EventEase — AI Event Management Platform
## n8n Capstone Project Documentation

### 1. Project Overview
EventEase is an AI-assisted event management automation system designed to handle the event lifecycle from participant registration through attendance, notifications, certificates, feedback, and event analytics.

The implementation consists of five interconnected workflows using Google Sheets for structured data storage, Gmail for communication, Google Drive for certificate storage, and Gemini for feedback analysis.

### 2. Problem Analysis

#### Business Context
Events such as conferences, workshops, hackathons, and training programs may involve many participants. Manual handling of registration, attendance, communication, certificates, and feedback creates repetitive work and can introduce errors.

#### Stakeholders
- Event participants
- Event organizers
- Registration/operations team
- Event administrators

#### Pain Points
- Manual participant registration and duplicate checking
- Manual attendance tracking
- Repetitive reminder emails
- Manual certificate preparation and distribution
- Feedback data requiring manual analysis
- Limited audit visibility across event operations

#### Objectives
1. Automate participant registration and validation.
2. Generate QR-based passes and record attendance.
3. Send scheduled event reminders.
4. Generate and distribute certificates after approval.
5. Collect feedback and generate AI-assisted event insights.
6. Maintain an audit trail of important workflow actions.

### 3. System Architecture

EventEase is organized into five workflows:

**W1 Registration → W2 QR & Attendance → W4 Certificates**

**W1 → W3 Reminders** through shared participant data.

**W2 + W3 + W4 + W5 → shared Google Sheets data**

**W5 Feedback → Gemini AI → Organizer Event Report**

Google Sheets contains seven tabs:
- Participants
- QR_Codes
- Attendance
- Certificates
- Feedback
- Notifications
- AuditLogs

### 4. Workflow Details

#### W1 — Registration & Participant Management (12 nodes)
Participant submits the registration form. The workflow normalizes and validates the submitted name, email, and phone number. It checks for duplicate registration using email + event, generates a Participant ID for valid unique registrations, stores the participant, sends a confirmation email, triggers W2, and records success/rejection in AuditLogs.

**Main capabilities:** form trigger, validation, duplicate prevention, ID generation, Google Sheets storage, Gmail confirmation, workflow-to-workflow HTTP trigger, audit logging.

#### W2 — QR Code Generation & Attendance (15 nodes)
W2 has two entry paths.

**QR generation:** receives a POST request from W1, creates a token and QR payload, stores the QR record, emails the QR pass, and logs the operation.

**Attendance:** receives a GET request from the QR scan, validates the QR token, checks whether attendance was already recorded, records valid attendance, logs the scan, and returns an HTML result page.

**Scan outcomes:** valid attendance, already checked in, or invalid scan.

#### W3 — Event Notifications & Reminders (10 nodes)
A daily 09:00 schedule trigger checks whether the event is within the configured reminder window. If it is, participant records are read and previously reminded participants are skipped using the Notifications sheet. Reminder emails are sent and logged. The email node includes retry handling and continues regular output on failure.

#### W4 — Certificate Generation & Distribution (14 nodes)
After the event, the organizer triggers the certificate workflow. Attendance is checked for eligibility and already-issued certificates are skipped. A human approval email provides Approve/Reject choices. On approval, personalized HTML certificates are generated, stored in Google Drive, emailed to participants, and logged. Rejections are also recorded.

#### W5 — Feedback & AI Event Analytics (14 nodes)
Participants submit feedback through a form. Feedback is stored and logged. A report webhook collects participant, attendance, and feedback data, calculates event metrics, builds an AI prompt, and sends it to Gemini. The AI response is parsed into structured fields including sentiment, satisfaction score, main issues, positive points, suggestions, and summary. The organizer receives an HTML report by email.

If the Gemini response cannot be parsed or the AI call fails, the workflow has a rating-based fallback report.

### 5. Data Model

| Sheet | Main fields |
|---|---|
| Participants | participantId, eventId, fullName, email, phone, college, department, registrationDate, status |
| QR_Codes | participantId, eventId, fullName, email, token, qrPayload, qrImageUrl, generatedAt, status |
| Attendance | attendanceId, participantId, eventId, fullName, email, checkInTime, method, status |
| Certificates | certificateId, eventId, participantId, fullName, email, driveFileId, driveLink, issuedAt, approvedBy, emailStatus |
| Feedback | feedbackId, timestamp, eventId, participantId, rating, comments, suggestions |
| Notifications | notificationId, timestamp, eventId, participantId, email, type, subject, status, error |
| AuditLogs | timestamp, workflow, eventId, participantId, action, status, message |

### 6. Integrations and Credentials

- Google Sheets OAuth2 — data storage
- Gmail OAuth2 — confirmations, reminders, approvals, certificates, reports
- Google Drive OAuth2 — certificate storage
- Gemini API — feedback analysis
- QR image service — QR code image generation

API keys are not embedded in the exported workflow JSON files. Credentials are attached after import.

### 7. Reliability, Error Handling & Governance

EventEase includes:
- Retry settings on important Gmail, HTTP, Drive, and Sheets operations.
- Rejection branches for invalid registration, duplicate registration, invalid attendance, and certificate rejection.
- AI fallback reporting when the Gemini response is unavailable or invalid.
- AuditLogs for important actions.
- Notifications and Certificates records for operational traceability.
- Duplicate prevention for registrations, attendance, reminders, and successfully issued certificates.

### 8. Workflow Node Count

| Workflow | Nodes |
|---|---:|
| W1 Registration & Participant Management | 12 |
| W2 QR Code Generation & Attendance | 15 |
| W3 Event Notifications & Reminders | 10 |
| W4 Certificate Generation & Distribution | 14 |
| W5 Feedback & AI Event Analytics | 14 |
| **Total** | **65** |

### 9. Requirement Mapping

| Requirement | Implementation |
|---|---|
| 4–6 independent workflows | 5 workflows |
| 20–25+ total nodes | 65 nodes |
| AI functionality | Gemini analysis in W5 |
| Webhook/Form triggers | W1, W2, W4, W5 |
| Scheduled workflow | W3 daily 09:00 trigger |
| Conditional branching | Validation, duplicate, attendance, reminder, approval branches |
| Human approval | W4 Gmail Send-and-Wait approval |
| Error handling/retry | Retry settings, failure branches, AI fallback |
| Logging/audit trail | AuditLogs + workflow-specific logs |
| Data storage | Google Sheets + Google Drive |
| Communication | Gmail |

### 10. Setup Summary

1. Import the five workflow JSON files into n8n.
2. Create the Google Sheets workbook using the supplied template.
3. Attach Google Sheets, Gmail, Google Drive, and Gemini credentials.
4. Update each Config node with the Sheet ID, event details, organizer email, and webhook URLs.
5. Import/activate in this order: W2 → W1 → W3 → W4 → W5.
6. Test registration, duplicate registration, QR generation, attendance, reminders, certificate approval, feedback, and AI report generation.

### 11. Testing Status

The supplied workflow package was programmatically checked for JSON validity, node references, connections, expressions, Code-node syntax, and logic tests. The workflows were **not live-executed against n8n, Google, or Gemini in the source preparation environment**. Live credential setup and end-to-end execution should therefore be completed before the final demo.

### 12. Limitations
- Certificates are generated as personalized HTML files; they can be printed/saved as PDF.
- Google Sheets is used as the project database rather than a transactional database.
- The current configuration is designed around one event per deployment.
- Phone-based QR scanning requires a reachable public webhook URL; localhost URLs are not reachable from another device.
- Gemini availability depends on the configured API quota.
- The current system is primarily email-based for participant communication.

### 13. Future Enhancements
- Native PDF certificate generation
- Database backend such as PostgreSQL
- Multi-event management
- WhatsApp/SMS notifications
- Organizer dashboard and visual analytics
- Role-based administration
- More advanced attendance analytics

### 14. Demo Flow
1. Show architecture and Google Sheets tabs.
2. Register a participant and show the stored row, confirmation email, and audit log.
3. Repeat the registration to demonstrate duplicate rejection.
4. Show QR email and scan it; repeat the scan to demonstrate duplicate attendance prevention.
5. Demonstrate the scheduled reminder workflow.
6. Trigger certificate issuance and show the organizer approval step.
7. Show certificate delivery and Drive storage.
8. Submit feedback and generate the AI event report.
9. Finish with AuditLogs and the 65-node requirement mapping.

