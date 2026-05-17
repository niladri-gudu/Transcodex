# 🎬 Transcodex

**Transcodex** is a full-stack video processing platform with distributed transcoding, adaptive streaming, and AI-powered video analysis.

## ✨ Features

### 📤 Multipart Uploads
Large video files are uploaded directly to AWS S3 using multipart uploads with chunked upload support, progress tracking, and presigned URLs.

### ⚙️ Distributed Video Processing
Asynchronous video processing pipeline using BullMQ workers and Redis queues:

1. Video uploaded to S3
2. Job added to processing queue
3. Worker downloads and validates video
4. FFmpeg transcodes to HLS format
5. Thumbnails generated
6. AI-powered captions/transcripts generated
7. Processed assets uploaded to S3
8. Real-time progress updates sent to frontend

### 🎥 HLS Adaptive Streaming
Multi-resolution transcoding (480p, 720p, 1080p) with:
- Adaptive bitrate streaming
- Manual quality switching
- Variable playback speed
- Subtitle support

### 🤖 AI Features

**AI Captions**
- Audio extraction from video
- Whisper-powered subtitle generation
- Automatic `.vtt` caption files

**AI Transcript Chat**
- Conversational AI interface for video content
- Ask questions like:
  - "Summarize this video"
  - "What technologies are discussed?"
  - "Explain the architecture"
- Powered by Gemini AI (called from Fastify backend)

### 🔄 Real-time Progress Updates
Socket.IO + Redis Pub/Sub architecture provides live transcoding updates without polling:
- Download progress
- Validation status
- Transcoding stage
- Completion/failure states

---

## 🏗️ Architecture

```text
Client
   │
   ▼
Next.js Frontend
   │
   ▼
Fastify API Server
   │
   ├── PostgreSQL (Prisma)
   ├── AWS S3
   ├── Redis
   ├── Gemini AI (transcript chat)
   └── BullMQ Queue
            │
            ▼
      Worker Service
            │
            ├── FFmpeg (transcoding)
            └── Whisper (captions)
```

---

## 🧰 Tech Stack

**Frontend**
- Next.js
- React
- Tailwind CSS
- Socket.IO Client
- hls.js

**Backend**
- Fastify
- Node.js
- Prisma ORM
- PostgreSQL
- BullMQ
- Redis
- Socket.IO
- Gemini API

**Media Processing**
- FFmpeg
- Whisper

**Cloud & Infrastructure**
- AWS S3
- Redis
- PostgreSQL

---

## 📂 Project Structure

```bash
frontend/    # Upload UI, video player, real-time progress, AI chat
backend/     # REST APIs, multipart uploads, queue management, WebSocket server, AI chat
worker/      # Video transcoding, thumbnail generation, caption generation
```

---

## ⚡ Processing Pipeline

```text
Upload
  ↓
S3 Storage
  ↓
BullMQ Queue
  ↓
Worker Picks Job
  ↓
FFmpeg Validation
  ↓
Thumbnail Generation
  ↓
HLS Transcoding (480p/720p/1080p)
  ↓
Caption Generation (Whisper)
  ↓
Transcript Generation
  ↓
Upload Processed Files to S3
  ↓
Real-time Progress Updates (Socket.IO)
  ↓
Video Ready to Stream
```

---

## 🚀 Local Setup

### 1. Clone Repository

```bash
git clone https://github.com/yourusername/transcodex.git
cd transcodex
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Setup Environment Variables

**Backend `.env`**
```env
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_BUCKET_NAME=your-bucket-name
PORT=8000
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://127.0.0.1:6379
GEMINI_API_KEY=your_gemini_api_key
```

**Frontend `.env`**
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

**Worker `.env`**
```env
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_BUCKET_NAME=your-bucket-name
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://127.0.0.1:6379
```

### 4. Run Database Migrations

```bash
cd backend
pnpm prisma migrate dev
```

### 5. Start Services

**Terminal 1 - Backend**
```bash
cd backend
pnpm dev
```

**Terminal 2 - Worker**
```bash
cd worker
pnpm dev
```

**Terminal 3 - Frontend**
```bash
cd frontend
pnpm dev
```

---

## 🎯 Challenges Solved

### Large File Uploads
Implemented multipart uploads to handle large video files efficiently without timeout issues.

### Memory Crashes During Transcoding
Replaced parallel transcoding with sequential processing to prevent memory exhaustion.

### Real-time Processing Updates
Integrated Redis Pub/Sub with Socket.IO for live progress tracking without polling overhead.

### Adaptive Streaming
Built complete HLS pipeline with multi-resolution support for adaptive bitrate streaming.

### AI Integration
Generated captions using Whisper and enabled conversational AI over transcripts using Gemini.

---

## 📈 Future Improvements

- [ ] Distributed transcoding workers with load balancing
- [ ] GPU-accelerated encoding
- [ ] CDN integration for global delivery
- [ ] RAG-based semantic search over transcripts
- [ ] Video recommendations engine
- [ ] Streaming AI responses
- [ ] Timeline-aware transcript search with timestamps

---

## 🌐 Deployment

**Frontend**: Vercel  
**Backend & Worker**: Railway / Render  
**Database**: Neon PostgreSQL  
**Redis**: Upstash Redis  
**Storage**: AWS S3

---

## 📜 License

MIT License

---

## 🙏 Acknowledgments

Built with modern streaming architecture patterns inspired by YouTube's processing pipeline.
