# autotask-agent
AI-powered email-to-task agent that transforms emails into actionable work using multi-agent architecture with FastAPI backend and React frontend

## 🚀 Real-Time Google Integration

AutoTask Agent now supports **real-time synchronization** with Google Gmail and Google Calendar:

### Live Demo
View the interactive real-time demo: [demo-google-realtime.html](demo-google-realtime.html)

### Features
- **Gmail API Integration**: Automatically reads emails from your Gmail inbox
- **Intelligent Processing**: Multi-agent pipeline analyzes emails and extracts actionable tasks
- **Google Calendar API**: Automatically creates calendar events for generated tasks
- **Real-Time Sync**: Continuous synchronization between email, tasks, and calendar events
- **Confidence Scoring**: AI-powered confidence scores for each generated task (85-92%)
- **Priority Classification**: Automatic task prioritization (HIGH, MEDIUM, LOW)

### Quick Start
1. Set up Google Cloud credentials ([GOOGLE_INTEGRATION_GUIDE.md](GOOGLE_INTEGRATION_GUIDE.md))
2. Install dependencies from [SETUP.md](SETUP.md)
3. Run the FastAPI server for real-time synchronization
4. View results in the interactive dashboard

### Architecture
The multi-agent pipeline processes emails through:
- **Gmail API**: Email Retrieval
- **NLU Agent**: Text Analysis
- **Task Composer**: Task Generation
- **Business Rules**: Validation
- **Calendar API**: Event Creation

For detailed setup instructions, see [GOOGLE_INTEGRATION_GUIDE.md](GOOGLE_INTEGRATION_GUIDE.md)
