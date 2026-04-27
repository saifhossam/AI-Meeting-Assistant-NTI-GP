# AI Meeting Assistant

An intelligent, end-to-end meeting assistant that automates transcription, speaker identification, visual content analysis, and generates comprehensive meeting reports with built-in RAG (Retrieval-Augmented Generation) capabilities for querying past meetings.

## 🌟 Overview

AI Meeting Assistant is a production-ready solution that combines state-of-the-art AI models to transform meeting recordings into actionable insights. The system processes both audio and video content, identifies speakers, extracts visual information from presentations, and provides an intelligent chatbot interface for querying historical meeting data.

### Key Features

- **🎤 Real-time Speaker Diarization & Identification**: Automatically detects and identifies speakers using voice embeddings
- **📝 Automatic Transcription**: Converts speech to text with high accuracy using Whisper
- **👁️ Visual Content Analysis**: Extracts and analyzes slides, dashboards, and charts from meeting videos
- **📊 Intelligent Report Generation**: Creates structured meeting reports with summaries, action items, and decisions
- **🤖 RAG-Powered Chat**: Query past meetings using natural language with hybrid retrieval
- **📧 Integration Ready**: Automatic PDF upload to n8n webhooks for downstream workflows
- **🎯 Speaker Management**: Enroll and manage speaker voice profiles with department tracking

## 🏗️ Architecture

The system consists of two main components:

### 1. Meeting Processing Backend (`Meeting_to_pdf/`)
- **FastAPI Server** (`app.py`): Handles meeting uploads, speaker enrollment, and processing
- **Audio Processing Pipeline**: Voice activity detection, noise reduction, and preprocessing
- **Speaker Recognition**: ECAPA-TDNN embeddings with cosine similarity matching
- **Transcription Engine**: Pyannote diarization + Whisper ASR
- **Visual Analysis**: Frame extraction, similarity filtering, and LLM-based explanation
- **Report Generation**: Structured PDF reports with meeting insights

### 2. RAG Query System (`RAG/`)
- **LangGraph Agent**: Multi-step reasoning with tool calling
- **Hybrid Retrieval**: Dense (Gemini embeddings) + Sparse (BM25) search
- **Qdrant Vector Store**: Efficient similarity search with parent-child chunking
- **Auto-Indexing**: File watcher automatically indexes new meeting reports
- **Smart Query Rewriting**: Analyzes and rewrites ambiguous queries for better retrieval

## 📋 Prerequisites

- **Python**: 3.8 or higher
- **Node.js**: 16 or higher (for frontend)
- **FFmpeg**: Required for audio/video conversion
- **CUDA** (Optional): For GPU acceleration

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/Ahmed-Essam-Hammam/AI-Meeting-Assistant.git
cd AI-Meeting-Assistant
```

### 2. Backend Setup (Meeting Processing)

```bash
cd Meeting_to_pdf
pip install -r requirements.txt
```

**Required Python Packages:**
```
fastapi
uvicorn
torch
torchaudio
transformers
pyannote.audio
speechbrain
librosa
soundfile
noisereduce
opencv-python
pillow
imagehash
scenedetect
pymupdf
pymupdf4llm
reportlab
pymediainfo
requests
```

### 3. RAG System Setup

```bash
cd ../RAG
pip install -r requirements.txt
```

**Additional Packages:**
```
langchain
langchain-google-genai
langchain-qdrant
qdrant-client
langgraph
watchdog
pydantic
```

### 4. Frontend Setup

```bash
cd ../Frontend
npm install
```

## ⚙️ Configuration

### 1. Update API Keys in `config.py`

```python
# Meeting_to_pdf/config.py
HF_TOKEN = "your_huggingface_token"  # For Pyannote models
API_KEY = "your_openrouter_api_key"  # For vision model

# RAG/config.py
GEMINI_KEY = "your_google_gemini_api_key"  # For embeddings and LLM
```

### 2. Configure n8n Webhook (Optional)

```python
# Meeting_to_pdf/n8n.py
N8N_WEBHOOK_URL = "your_n8n_webhook_url"
```

### 3. Directory Structure

The system automatically creates necessary directories:
```
Meeting_to_pdf/
├── processed_audio_enrollment/  # Speaker voice profiles
├── processed_audio_test/         # Processed meeting audio
├── docs/                          # Generated PDF reports
└── rag_knowledge_base/           # PDFs for RAG indexing

RAG/
├── markdown/                      # Converted markdown from PDFs
├── parent_store/                  # Parent chunk storage
└── qdrant_db/                     # Vector database
```

## 🎯 Usage

### Starting the Backend Services

#### 1. Start Meeting Processing Server

```bash
cd Meeting_to_pdf
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

#### 2. Start RAG Server

```bash
cd RAG
uvicorn app1:app --reload --host 0.0.0.0 --port 8001
```

### Starting the Frontend

```bash
cd Frontend
npm run dev
```

Access the application at `http://localhost:3000`

## 📚 API Endpoints

### Meeting Processing API (Port 8000)

#### Enroll Speaker
```http
POST /enroll-speaker/
Content-Type: multipart/form-data

Parameters:
- file: Audio file (WAV, MP3, MP4, etc.)
- speaker_name: Name of the speaker
- department: Department/organization
```

#### Process Meeting
```http
POST /process-meeting/
Content-Type: multipart/form-data

Parameters:
- file: Meeting recording (audio or video)

Response:
{
  "transcript": "Full meeting transcript...",
  "report": "Formatted meeting report...",
  "pdf_path": "path/to/report.pdf",
  "summary": "Meeting summary..."
}
```

#### Dashboard Statistics
```http
GET /dashboard-stats

Response:
{
  "totalMeetings": 42,
  "voiceProfiles": 15,
  "hoursSaved": 42,
  "recentMeetings": [...]
}
```

#### List Speakers
```http
GET /list-speakers

Response:
[
  {
    "id": "John Doe",
    "name": "John Doe",
    "department": "Engineering",
    "status": "trained"
  }
]
```

#### Delete Speaker
```http
DELETE /delete-speaker/{speaker_name}
```

#### List Meetings
```http
GET /list-meetings
```

#### Download Meeting PDF
```http
GET /download-pdf/{filename}
```

### RAG Query API (Port 8001)

#### Chat with RAG
```http
POST /rag/chat
Content-Type: application/json

{
  "message": "What were the action items from last week's meeting?"
}

Response:
{
  "answer": "According to the meeting report from 2025-01-15..."
}
```

#### Health Check
```http
GET /health
```

## 🔧 How It Works

### Meeting Processing Pipeline

1. **Upload**: User uploads meeting recording (audio/video)
2. **Conversion**: Video converted to WAV using FFmpeg
3. **Preprocessing**: 
   - Noise reduction
   - Voice activity detection (VAD)
   - Bandpass filtering (80-7500 Hz)
4. **Diarization**: Pyannote identifies speaker segments
5. **Speaker Identification**: Matches segments to enrolled speakers using ECAPA embeddings
6. **Transcription**: Whisper generates text for each segment
7. **Visual Analysis** (for video):
   - Extract keyframes using scene detection
   - Filter similar frames using perceptual hashing
   - Remove unimportant frames (low content, dark screens)
   - Generate explanations using vision-language model
8. **Report Generation**: LLM synthesizes transcript + visuals into structured report
9. **PDF Creation**: ReportLab generates professional PDF
10. **Auto-Upload**: Sends to n8n webhook and indexes for RAG

### RAG Query Pipeline

1. **Query Received**: User asks question about past meetings
2. **Conversation Summarization**: Summarizes recent context (if multi-turn)
3. **Query Analysis**: Determines if query is clear; rewrites if needed
4. **Tool Selection**: Agent decides to use `search_child_chunks` or `retrieve_parent_chunks`
5. **Hybrid Search**: 
   - Qdrant retrieves relevant child chunks using dense + sparse embeddings
   - Optional file scoping (e.g., "in meeting_2025-01-15.pdf")
6. **Parent Retrieval**: Fetches full context for relevant chunks
7. **Answer Generation**: Gemini generates response citing sources
8. **Response**: Returns answer with source attribution

### Speaker Enrollment Process

1. **Upload Audio**: User provides voice sample
2. **Preprocessing**: Same pipeline as meeting audio
3. **Embedding Extraction**: Generate 192-dim ECAPA-TDNN embedding
4. **Storage**: 
   - WAV file saved to `processed_audio_enrollment/`
   - Embedding stored in `speaker_database.npy`
   - Metadata (name, department) saved as JSON sidecar

## 🧠 AI Models Used

| Component | Model | Purpose |
|-----------|-------|---------|
| **Speaker Diarization** | `pyannote/speaker-diarization-3.1` | Segment audio by speaker |
| **Speech Recognition** | `openai/whisper-small` | Transcribe speech to text |
| **Speaker Verification** | `speechbrain/spkrec-ecapa-voxceleb` | Extract voice embeddings |
| **Vision Analysis** | `meta-llama/llama-3.2-11b-vision-instruct` | Analyze meeting visuals |
| **Report Generation** | `meta-llama/Llama-3.1-8B-Instruct` | Generate structured reports |
| **RAG Embeddings** | `text-embedding-004` (Gemini) | Dense vector embeddings |
| **Sparse Embeddings** | `Qdrant/bm25` | Keyword-based retrieval |
| **RAG LLM** | `gemini-flash-latest` | Answer generation |

## 📁 Project Structure

```
AI-Meeting-Assistant/
├── Meeting_to_pdf/              # Main processing backend
│   ├── app.py                   # FastAPI application
│   ├── config.py                # Configuration and paths
│   ├── models.py                # AI model initialization
│   ├── preprocessing.py         # Audio/video preprocessing
│   ├── speaker_db.py            # Speaker database management
│   ├── speaker_identification.py # Speaker matching logic
│   ├── transcription.py         # Transcription pipeline
│   ├── visuals_extractor.py     # Visual content analysis
│   ├── report_generator.py      # Report generation
│   ├── pdf_utils.py             # PDF creation utilities
│   ├── video_to_wav.py          # Video conversion
│   ├── n8n.py                   # Webhook integration
│   └── requirements.txt
│
├── RAG/                         # Query system
│   ├── app1.py                  # RAG FastAPI server
│   ├── bootstrap.py             # Initialization
│   ├── config.py                # RAG configuration
│   ├── embeddings.py            # Embedding models
│   ├── vector_db.py             # Qdrant setup
│   ├── pdf_to_markdown.py       # PDF conversion
│   ├── indexing.py              # Document indexing
│   ├── tools.py                 # LangChain tools
│   ├── llm_setup.py             # LLM configuration
│   ├── graph.py                 # LangGraph workflow
│   ├── graph_nodes.py           # Agent nodes
│   ├── state.py                 # State definitions
│   ├── prompts.py               # System prompts
│   └── requirements.txt
│
└── Frontend/                    # React application
    ├── src/
    │   ├── components/          # UI components
    │   ├── pages/               # Application pages
    │   └── App.tsx
    ├── package.json
    └── tailwind.config.ts
```

## 🔍 Advanced Features

### Visual Content Filtering

The system employs multi-stage filtering to extract meaningful visual content:

1. **Scene Detection**: Identifies keyframes at scene boundaries
2. **Perceptual Hashing**: Removes near-duplicate frames using pHash, dHash, and aHash
3. **Content Quality Filtering**:
   - Edge density analysis (removes low-detail frames)
   - Color diversity check (filters avatar/name slides)
   - Brightness filtering (removes dark participant screens)
   - Text-area ratio (excludes pure text slides)

### Parent-Child Chunking Strategy

- **Parent Chunks**: 2,000-10,000 characters from markdown sections
- **Child Chunks**: 500 characters with 100-character overlap
- **Retrieval**: Search child chunks, retrieve parent context
- **Hybrid Search**: Combines semantic similarity with keyword matching

### Query Intelligence

- **Follow-up Detection**: Resolves pronouns using conversation history
- **Query Rewriting**: Corrects grammar and optimizes for retrieval
- **File Scoping**: Supports queries like "in meeting_2025-01-15.pdf"
- **Multi-query Splitting**: Breaks complex questions into sub-queries

## 🐛 Troubleshooting

### Common Issues

**Issue: "Speaker database not found"**
```bash
# Solution: The database is auto-created on first enrollment
# Ensure ENROLL_DIR exists and has write permissions
```

**Issue: "CUDA out of memory"**
```python
# Solution: Set DEVICE to "cpu" in config.py
DEVICE = "cpu"
```

**Issue: "FFmpeg not found"**
```bash
# Ubuntu/Debian
sudo apt-get install ffmpeg

# macOS
brew install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

**Issue: "Qdrant collection not found"**
```bash
# Solution: Delete qdrant_db folder and restart RAG server
rm -rf RAG/qdrant_db/
```

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- **Pyannote Audio** - Speaker diarization
- **OpenAI Whisper** - Speech recognition
- **SpeechBrain** - Speaker verification
- **LangChain** - RAG framework
- **Qdrant** - Vector database
- **Google Gemini** - Embeddings and LLM

## 🗺️ Roadmap

- [ ] Real-time meeting processing (streaming)
- [ ] Multi-language transcription support
- [ ] Video meeting platform integrations (Zoom, Teams, Meet)
- [ ] Sentiment analysis and emotion detection
- [ ] Meeting scheduling and calendar integration
- [ ] Mobile application (iOS/Android)
- [ ] Advanced analytics dashboard
- [ ] Export to multiple formats (DOCX, JSON, HTML)
- [ ] Custom report templates
- [ ] Role-based access control

---

