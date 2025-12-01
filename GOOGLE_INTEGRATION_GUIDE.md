# AutoTask Agent - Google Integration Guide

## Overview
This guide shows how to integrate AutoTask Agent with Google Gmail and Google Calendar APIs to enable:
- Real-time email reading from Gmail
- Automatic calendar event creation in Google Calendar
- Real-time task synchronization

## Step 1: Setup Google Cloud Project

### 1.1 Create a Google Cloud Project
1. Go to https://console.cloud.google.com
2. Click "Select a Project" → "New Project"
3. Name it "AutoTask Agent"
4. Click Create

### 1.2 Enable Required APIs
1. In Cloud Console, search for "Gmail API"
2. Click on it and press "Enable"
3. Search for "Google Calendar API"
4. Click on it and press "Enable"

### 1.3 Create Service Account
1. Go to "APIs & Services" → "Credentials"
2. Click "+ Create Credentials" → "Service Account"
3. Fill in the details and click Create
4. Grant "Project Editor" role
5. Create a JSON key and download it
6. Save as `backend/credentials.json`

## Step 2: Install Google Libraries

Add to backend/requirements.txt:
```
google-auth-oauthlib
google-auth-httplib2
google-api-python-client
```

Install:
```bash
pip install google-api-python-client google-auth-oauthlib google-auth-httplib2
```

## Step 3: Backend Integration

Create `backend/app/integrations/google_gmail.py`:
```python
from google.auth.transport.requests import Request
from google.oauth2.service_account import Credentials
from google.auth.oauthlib.flow import InstalledAppFlow
from google_auth_httplib2 import AuthorizedHttp
from googleapiclient.discovery import build
import json
import base64
from email.mime.text import MIMEText

SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']
CALENDAR_SCOPES = ['https://www.googleapis.com/auth/calendar']

class GmailIntegration:
    def __init__(self):
        self.service = self._authenticate()
    
    def _authenticate(self):
        creds = Credentials.from_service_account_file(
            'credentials.json', scopes=SCOPES)
        return build('gmail', 'v1', credentials=creds)
    
    def get_recent_emails(self, max_results=5):
        try:
            results = self.service.users().messages().list(
                userId='me', q='is:unread', maxResults=max_results
            ).execute()
            messages = results.get('messages', [])
            emails = []
            for msg in messages:
                email_data = self.service.users().messages().get(
                    userId='me', id=msg['id']
                ).execute()
                emails.append(self._parse_email(email_data))
            return emails
        except Exception as e:
            print(f"Error fetching emails: {e}")
            return []
    
    def _parse_email(self, message):
        headers = message['payload']['headers']
        return {
            'id': message['id'],
            'from': next(h['value'] for h in headers if h['name'] == 'From'),
            'subject': next((h['value'] for h in headers if h['name'] == 'Subject'), 'No Subject'),
            'body': self._get_email_body(message)
        }
    
    def _get_email_body(self, message):
        if 'parts' in message['payload']:
            for part in message['payload']['parts']:
                if part['mimeType'] == 'text/plain':
                    data = part['body'].get('data', '')
                    return base64.urlsafe_b64decode(data).decode('utf-8')
        else:
            data = message['payload']['body'].get('data', '')
            return base64.urlsafe_b64decode(data).decode('utf-8') if data else ''

class GoogleCalendarIntegration:
    def __init__(self):
        self.service = self._authenticate()
    
    def _authenticate(self):
        creds = Credentials.from_service_account_file(
            'credentials.json', scopes=CALENDAR_SCOPES)
        return build('calendar', 'v3', credentials=creds)
    
    def create_event(self, task_data):
        try:
            event = {
                'summary': task_data['title'],
                'description': task_data['description'],
                'start': {
                    'dateTime': task_data['due_date'].isoformat(),
                    'timeZone': 'UTC'
                },
                'end': {
                    'dateTime': task_data['due_date'].isoformat(),
                    'timeZone': 'UTC'
                },
                'reminders': {
                    'useDefault': False,
                    'overrides': [
                        {'method': 'email', 'minutes': 24 * 60},
                        {'method': 'popup', 'minutes': 30}
                    ]
                }
            }
            result = self.service.events().insert(
                calendarId='primary', body=event
            ).execute()
            return result
        except Exception as e:
            print(f"Error creating calendar event: {e}")
            return None
```

## Step 4: Real-Time Integration Endpoint

Add to `backend/app/main.py`:
```python
from integrations.google_gmail import GmailIntegration, GoogleCalendarIntegration

@app.get("/sync/gmail")
def sync_gmail():
    gmail = GmailIntegration()
    emails = gmail.get_recent_emails(max_results=10)
    processed_tasks = []
    
    calendar = GoogleCalendarIntegration()
    for email in emails:
        # Process email with agents
        task = process_email_with_agents(email)
        if task['confidence'] > 0.8:
            calendar.create_event(task)
            processed_tasks.append(task)
    
    return {"status": "synced", "tasks_created": len(processed_tasks), "tasks": processed_tasks}
```

## Step 5: Frontend Real-Time Display

Frontend should poll `/sync/gmail` endpoint every 30 seconds to show real-time updates.

## Testing

```bash
curl http://localhost:8000/sync/gmail
```

This will:
1. Read 10 most recent unread emails from Gmail
2. Process each with multi-agent pipeline
3. Auto-create high-confidence tasks
4. Add events to Google Calendar
5. Return results in real-time

## Output Example
```json
{
  "status": "synced",
  "tasks_created": 3,
  "tasks": [
    {
      "task_id": "task_abc123",
      "title": "Complete Q4 Deliverables",
      "source_email_id": "msg_xyz789",
      "confidence": 0.89,
      "calendar_event_id": "evt_def456",
      "calendar_link": "https://calendar.google.com/calendar/u/0/r/eventedit/evt_def456",
      "status": "auto_created"
    }
  ]
}
```

## Security Notes

1. Keep `credentials.json` in `.gitignore`
2. Use environment variables for sensitive data
3. Implement OAuth2 for user authentication
4. Add rate limiting to prevent API abuse
5. Use service account for server-to-server communication

## Troubleshooting

- "Permission denied": Ensure Gmail and Calendar APIs are enabled
- "Invalid credentials": Check `credentials.json` is valid
- "Rate limit exceeded": Implement exponential backoff
- "No unread emails": Check Gmail account has unread messages
