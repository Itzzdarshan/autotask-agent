# AutoTask Agent - Complete Setup Guide

## Quick Start (Clone & Deploy)

### 1. Clone Repository
```bash
git clone https://github.com/Itzzdarshan/autotask-agent.git
cd autotask-agent
```

### 2. Backend Setup
```bash
cd backend
pip install -r requirements.txt
```

### 3. Frontend Setup
```bash
cd ../frontend
npm install
```

### 4. Run Application
Terminal 1 - Backend:
```bash
cd backend
uvicorn app.main:app --reload
```

Terminal 2 - Frontend:
```bash
cd frontend
npm run dev
```

## Environment Configuration
Create `.env` in backend folder:
DB_URL=mysql+pymysql://user:pass@localhost:3306/autotask
CORS_ORIGINS=["http://localhost:5173"]

## API Endpoints
- POST /ingest - Ingest email
- GET /tasks - List all tasks  
- GET /tasks/pending - Pending tasks

## File Structure
auotask-agent/
- backend/app/main.py
- backend/app/agents/
- frontend/src/components/
- README.md
