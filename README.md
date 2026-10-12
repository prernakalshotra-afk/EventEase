# EventEase — AI Event Management Platform

AI-assisted event management automation built as an n8n capstone project.

## Overview
EventEase automates participant registration, QR-based attendance, event reminders, certificate generation/distribution, feedback collection, and AI-assisted event analytics.

## Workflows

| # | Workflow | Nodes |
|---|---|---:|
| 1 | Registration & Participant Management | 12 |
| 2 | QR Code Generation & Attendance | 15 |
| 3 | Event Notifications & Reminders | 10 |
| 4 | Certificate Generation & Distribution | 14 |
| 5 | Feedback & AI Event Analytics | 14 |
| **Total** | **5 workflows** | **65** |

## Architecture

```text
                         EVENTEASE
                  AI Event Management Platform

                           │
                           ▼
              ┌─────────────────────────┐
              │ 1. REGISTRATION         │
              │ Validation + Participant│
              │ Management               │
              └────────────┬────────────┘
                           │
                    Participant Data
                           ▼
              ┌─────────────────────────┐
              │ 2. QR & ATTENDANCE      │
              │ QR Generation + Scanning │
              └────────────┬────────────┘
                           │
                    Attendance Data
                     ┌─────┴─────┐
                     ▼           ▼
          ┌────────────────┐  ┌──────────────────┐
          │ 3. NOTIFICATION│  │ 4. CERTIFICATE   │
          │ & REMINDERS    │  │ & DISTRIBUTION   │
          └───────┬────────┘  └────────┬─────────┘
                  │                    │
                  └─────────┬──────────┘
                            ▼
                 ┌─────────────────────┐
                 │ 5. FEEDBACK &       │
                 │ AI ANALYTICS        │
                 └──────────┬──────────┘
                            ▼
                       Gemini AI
                            │
                            ▼
                     Event Insights
```

## Main Features
- Registration validation and duplicate prevention
- Automatic Participant ID generation
- QR pass generation and email delivery
- Token-validated attendance
- Duplicate attendance prevention
- Scheduled reminder emails
- Human approval before certificates are issued
- Personalized certificate generation and Google Drive storage
- Feedback collection
- Gemini-based sentiment, satisfaction, issues, positives, suggestions, and summary
- AI fallback report
- Audit trail

## Technology
- n8n
- Google Sheets
- Gmail
- Google Drive
- Gemini API
- QR image service

## Repository Structure

```text
EventEase/
├── workflows/
│   ├── workflow_1_registration_participants.json
│   ├── workflow_2_qr_attendance.json
│   ├── workflow_3_reminders.json
│   ├── workflow_4_certificates.json
│   └── workflow_5_feedback_ai_analytics.json
├── AI_Event_Platform_Sheets_Template.xlsx
├── PROJECT_GUIDE.md
├── EventEase_Documentation.md
└── README.md
```

## Google Sheets Data Model

Seven tabs are used:

`Participants`, `QR_Codes`, `Attendance`, `Certificates`, `Feedback`, `Notifications`, `AuditLogs`.

## Setup

### Prerequisites
- n8n 1.80+
- Google account
- Gemini API key

### Credentials
Attach:
- Google Sheets OAuth2
- Gmail OAuth2
- Google Drive OAuth2
- Gemini Header Auth using `x-goog-api-key`

No API keys are embedded in the workflow JSON.

### Import Order
Import and configure in this order:

1. W2 — QR & Attendance
2. W1 — Registration
3. W3 — Reminders
4. W4 — Certificates
5. W5 — Feedback & AI Analytics

W2 is imported first because W1 triggers its QR-generation webhook.

### Configuration
Update each workflow's Config node with:
- Google Sheet ID
- Event ID/name/date
- Organizer email
- Public webhook URLs where required

For phone-based QR scanning, localhost webhook URLs must be replaced with a reachable public URL.

## AI Output
The AI analysis is structured into:

```json
{
  "sentiment": "positive | neutral | negative",
  "satisfaction_score": 1,
  "main_issues": [],
  "positive_points": [],
  "suggestions": [],
  "summary": ""
}
```

If AI parsing/API processing fails, W5 produces a rating-based fallback report.

## Reliability
The workflows include retry settings, conditional rejection paths, duplicate checks, audit logging, and an AI fallback.

## Testing Note
The source package was programmatically checked for JSON validity, node references, connections, expressions, Code-node syntax, and logic tests. It was not live-executed against n8n/Google/Gemini during source preparation. Complete live testing should be performed after credentials and configuration are added.

## Limitations
- Certificates are HTML files that can be printed/saved as PDF.
- Google Sheets is used as the project database.
- Current configuration is designed around one event per deployment.
- Phone QR scanning needs a public webhook URL.
- Communication is primarily email-based.

## Future Scope
- Native PDF certificates
- PostgreSQL/database backend
- Multi-event support
- WhatsApp/SMS notifications
- Organizer dashboard
- Role-based administration
- Advanced analytics

## License
MIT
