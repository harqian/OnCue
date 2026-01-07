# OnCue

Your AI-powered interview wingman. OnCue listens to interview questions in real-time and surfaces relevant memories from your experience vault exactly when you need them.

[![DEMO](https://img.youtube.com/vi/4UaQEB1b84E/maxresdefault.jpg)](https://www.youtube.com/watch?v=4UaQEB1b84E)

## What It Does

OnCue uses live audio transcription to detect interview questions and automatically pulls up the most relevant information from your personal knowledge base using vector search. Think of it as having all your best stories, projects, and accomplishments on standby—appearing right on cue.

**The Magic Flow:**
1. Start recording during your interview
2. Interviewer asks: "Tell me about a challenging project..."
3. OnCue detects the question and searches your memory vault
4. Relevant experiences appear instantly with similarity scores
5. Answer confidently with context at your fingertips

## Tech Stack

- **Frontend**: React 19 + TypeScript + Vite + Tailwind CSS
- **AI/ML**: Google Gemini (audio transcription, embeddings, LLM detection)
- **Backend**: Node.js + Express
- **Database**: MongoDB Atlas with Vector Search
- **Models Used**:
  - `gemini-2.5-flash-native-audio-preview-09-2025` (live audio transcription)
  - `text-embedding-004` (vector embeddings)
  - `gemini-2.0-flash-exp` (optional LLM-based question detection)

## Features

### Real-Time Audio Transcription
- Captures microphone input and streams to Google Gemini
- Live, sentence-level transcription updates
- Distinguishes between final and intermediate results

### Smart Question Detection (Dual Mode)
- **Keyword Mode** (default): Lightning-fast pattern matching
  - Detects question marks and interview keywords
  - Zero latency, no API calls
- **LLM Mode**: AI-powered semantic analysis
  - Understands context and nuance
  - Enable by setting `DETECTION_MODE = 'LLM'` in `App.tsx:18`

### Vector-Powered Memory Search
- Automatically queries MongoDB when questions are detected
- Uses semantic similarity search via embeddings
- Returns top 3-5 most relevant memories with:
  - Classification (category)
  - Description (the actual memory)
  - Source file
  - Similarity score (0-1)
  - Creation date

### Clean UI/UX
- Questions highlighted in yellow
- Expandable memory panels
- Real-time connection status indicators
- One-click start/stop recording

## Quick Start

### Prerequisites

- Node.js 18+
- MongoDB Atlas account with vector search enabled
- Google Gemini API key

### Environment Setup

Create a `.env` file in the project root:

```bash
MONGO_DB_URI="your_mongodb_atlas_connection_string"
GEMINI_API_KEY="your_gemini_api_key"
```

### Installation & Running

**Start the backend server** (runs on port 5001):
```bash
cd mongodb_backend
export MONGO_DB_URI="your_mongodb_uri"
export GEMINI_API_KEY="your_api_key"
node server.mjs
```

**Start the frontend** (runs on port 3000):
```bash
cd interview_app
npm install
export API_KEY="your_gemini_api_key"
npm run dev
```

Open http://localhost:3000 in your browser.

## Project Structure

```
gemini-multimodal-hack/
├── interview_app/              # React frontend
│   ├── App.tsx                # Main app component
│   ├── services/
│   │   ├── geminiService.ts   # Audio transcription logic
│   │   └── vectorSearchService.ts  # MongoDB search integration
│   └── ...
├── mongodb_backend/           # Node.js backend
│   ├── server.mjs            # Express REST API
│   ├── mcp-server.mjs        # MCP tool server
│   └── ...
└── ACCESS_DATABASE.md        # Database setup guide
```

## MongoDB Setup

**Database**: `context`
**Collection**: `test1`
**Vector Index**: `vector_index`

**Document Schema**:
```javascript
{
  classification: String,    // e.g., "project", "skill", "achievement"
  description: String,       // The actual memory/experience
  sourceFile: String,        // Origin of the data
  createdAt: ISODate,       // Timestamp
  embedding: Array[768]     // Vector embedding
}
```

### Creating the Vector Index

In MongoDB Atlas:
1. Go to your cluster → Database → Browse Collections
2. Select `context` database → `test1` collection
3. Go to "Search Indexes" → "Create Index"
4. Use JSON Editor and paste:

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 768,
      "similarity": "cosine"
    }
  ]
}
```

5. Name it `vector_index`

## API Endpoints

### Backend REST API (Port 5001)

**POST /api/search-memory**
```javascript
// Request
{
  "query": "challenging projects",
  "limit": 5  // optional, defaults to 5
}

// Response
[
  {
    "classification": "project",
    "description": "Built a real-time ML pipeline...",
    "sourceFile": "resume.json",
    "createdAt": "2024-01-15T...",
    "score": 0.89
  },
  ...
]
```

**POST /api/save-memory**
```javascript
// Request
{
  "memory": {
    "classification": "project",
    "description": "Developed OnCue interview assistant"
  },
  "sourceFile": "projects.json"
}

// Response
{
  "success": true,
  "id": "..."
}
```

### MCP Server

The Model Context Protocol server exposes a `vector_search` tool for AI agents (Claude, etc.):

```bash
node mongodb_backend/mcp-server.mjs
```

## Configuration Options

### Question Detection Mode

In `interview_app/App.tsx:18`:

```typescript
const DETECTION_MODE: 'keyword' | 'LLM' = 'keyword';  // Change to 'LLM' for AI detection
```

- **keyword**: Fast, pattern-based (recommended for low latency)
- **LLM**: AI-powered, more accurate (uses Gemini API calls)

### Search Result Limit

In `interview_app/services/vectorSearchService.ts:7`:

```typescript
limit: 5  // Adjust number of results returned
```

## How It Works

### Audio Processing Pipeline
1. Browser captures microphone input via Web Audio API
2. Audio converted from Float32Array to Int16Array PCM
3. Streamed to Gemini Live API in real-time
4. Transcription results streamed back sentence by sentence

### Question Detection Logic
- Monitors transcript for question indicators
- Groups consecutive question phrases together
- Prevents duplicate searches for the same question group
- Triggers vector search only for new questions

### Vector Search Flow
1. Question text → Gemini embeddings API
2. 768-dimensional vector created
3. MongoDB vector search finds similar documents
4. Top N results ranked by cosine similarity
5. Results displayed with confidence scores

## Development Notes

- Frontend uses Vite for fast HMR
- Backend supports CORS for cross-origin requests
- Audio context properly cleaned up on component unmount
- LLM mode includes caching to reduce API costs (max 100 items)
- Error handling for microphone permissions and API failures

## Future Ideas

- Support for multiple languages
- Custom memory categories/tags
- Voice-activated recording
- Export interview transcripts
- Integration with resume parsers
- Mobile app version

## License

MIT

## Credits

Built during the Gemini Multimodal Hackathon. Combines Google Gemini's multimodal AI capabilities with MongoDB's vector search for an intelligent interview preparation tool.
