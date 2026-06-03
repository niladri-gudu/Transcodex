This is the version I'd ship to GitHub. It's detailed enough to show engineering depth, but not so long that people stop reading.

# 🎬 Transcodex

> A full-stack video processing platform inspired by modern streaming systems like YouTube.
>
> Transcodex supports large video uploads, distributed transcoding, adaptive HLS streaming, AI-generated captions, transcript-based video chat, thumbnail generation, and real-time processing updates.

---

## 🚀 Highlights

* Multipart uploads directly to AWS S3
* Distributed video processing using BullMQ and Redis
* FFmpeg-powered HLS transcoding (480p / 720p / 1080p)
* Adaptive bitrate streaming with manual quality selection
* Automatic thumbnail generation
* AI-generated captions using Whisper
* Transcript generation and storage
* AI-powered chat over video transcripts using Gemini
* Real-time transcoding progress via Socket.IO
* AWS S3 storage for raw and processed assets

---

## 🌐 Live Demo

Frontend: `Coming Soon`

Backend API: `Coming Soon`

> **Note:** The media-processing worker currently runs locally because FFmpeg and Whisper workloads are resource-intensive and platform-dependent. The frontend and backend can be deployed independently while the worker remains isolated.

---

# 📖 Overview

Most video platforms hide the complexity of media processing behind a simple upload button.

The goal of this project was to understand and build the entire pipeline from scratch:

* Uploading large files efficiently
* Processing media asynchronously
* Generating adaptive streaming formats
* Creating subtitles and transcripts automatically
* Building AI-powered interactions on top of video content
* Delivering real-time updates to users

The result is a mini YouTube-style processing system built using modern web technologies and cloud infrastructure.

---

# 🏗️ System Architecture

```text
                           ┌───────────────┐
                           │   Frontend    │
                           │   Next.js     │
                           └───────┬───────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │      Fastify API        │
                      └───────┬────────┬────────┘
                              │        │
                              │        │
                              ▼        ▼
                        PostgreSQL   Redis
                         (Prisma)   (BullMQ)

                              │
                              ▼
                      ┌────────────────┐
                      │ Processing Queue│
                      └────────┬───────┘
                               │
                               ▼
                      ┌────────────────┐
                      │ Worker Service │
                      └───────┬────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
      FFmpeg             Whisper             Gemini
   Transcoding         Captions          Transcript Chat

                              │
                              ▼
                          AWS S3
```

---

# ⚡ End-to-End Processing Flow

```text
Browser Upload
      │
      ▼
Multipart Upload to S3
      │
      ▼
Video Record Created
      │
      ▼
BullMQ Job Created
      │
      ▼
Worker Picks Job
      │
      ▼
Download Raw Video
      │
      ▼
Validate Video
      │
      ▼
Generate Thumbnail
      │
      ▼
Generate Captions
      │
      ▼
Generate Transcript
      │
      ▼
Transcode to HLS
      │
      ▼
Upload Assets to S3
      │
      ▼
Update Database
      │
      ▼
Emit Realtime Events
      │
      ▼
Ready to Stream
```

---

# ✨ Features

## 📤 Multipart Uploads

Large videos are uploaded directly to AWS S3 using multipart uploads.

Benefits:

* Supports very large files
* Upload progress tracking
* Reduced backend load
* Reliable uploads over unstable connections
* Presigned URL-based architecture

---

## ⚙️ Distributed Processing Pipeline

Video processing is performed asynchronously using BullMQ workers.

This prevents long-running FFmpeg jobs from blocking API requests.

Pipeline includes:

* Video validation
* Thumbnail extraction
* Caption generation
* Transcript generation
* Multi-resolution transcoding
* Asset uploads
* Progress updates

---

## 🎥 Adaptive HLS Streaming

Videos are transcoded into multiple streaming variants:

* 480p
* 720p
* 1080p

Generated assets:

```text
master.m3u8

480p.m3u8
720p.m3u8
1080p.m3u8

480p_001.ts
480p_002.ts
...

720p_001.ts
720p_002.ts
...

1080p_001.ts
1080p_002.ts
...
```

Player features:

* Adaptive bitrate streaming
* Manual quality selection
* Playback speed controls
* Subtitle support
* HLS playback using hls.js

---

## 🖼️ Thumbnail Generation

A thumbnail is automatically extracted during processing.

Pipeline:

```text
Video
  ↓
FFmpeg
  ↓
Thumbnail
  ↓
AWS S3
  ↓
Database
```

The thumbnail generation process runs concurrently with transcoding to reduce overall processing time.

---

## 🤖 AI Captions

Automatic subtitle generation using Whisper.

Pipeline:

```text
Video
 ↓
Audio Extraction
 ↓
Whisper
 ↓
Transcript
 ↓
WebVTT File
 ↓
AWS S3
```

Generated captions are automatically loaded into the video player.

---

## 📝 Transcript Generation

A complete transcript is generated and stored alongside captions.

Benefits:

* Searchable content
* AI context generation
* Future semantic search support
* Better accessibility

Storage:

```text
processed/transcripts/video-id.txt
```

---

## 💬 AI Transcript Chat

Users can interact with video content using natural language.

The transcript is used as context for Gemini.

Example prompts:

```text
Summarize this video

What technologies are discussed?

Explain the architecture

What are the key takeaways?

What does the speaker say about Redis?
```

Pipeline:

```text
Transcript
     │
     ▼
Gemini Prompt
     │
     ▼
AI Response
```

---

## 🔄 Real-Time Progress Updates

Progress updates are streamed using:

* Socket.IO
* Redis Pub/Sub

This eliminates the need for polling.

Events include:

```text
Downloading Video
Validating File
Generating Thumbnail
Generating Captions
Generating Transcript
Transcoding 480p
Transcoding 720p
Transcoding 1080p
Uploading Assets
Completed
Failed
```

---

# 🗄️ Database Schema

## Video

```text
Video
├── id
├── title
├── rawS3Key
├── status
├── thumbnailS3Key
├── captionsS3Key
├── transcriptS3Key
├── createdAt
└── updatedAt
```

---

## VideoVariant

```text
VideoVariant
├── id
├── videoId
├── resolution
└── s3Key
```

---

# 🧰 Tech Stack

## Frontend

* Next.js
* React
* Tailwind CSS
* Socket.IO Client
* hls.js

## Backend

* Fastify
* Node.js
* Prisma ORM
* PostgreSQL
* BullMQ
* Redis
* Socket.IO

## Media Processing

* FFmpeg
* Whisper

## AI

* Gemini

## Infrastructure

* AWS S3
* Redis
* PostgreSQL

---

# 🎯 Engineering Challenges Solved

## Large File Uploads

Implemented multipart uploads to support large videos without request timeout issues.

---

## Memory Crashes During Transcoding

Initial implementation used parallel transcoding which caused memory exhaustion.

Solution:

```text
Promise.all
     ↓
Sequential Processing
```

Result:

* Lower memory usage
* Stable processing pipeline
* Predictable resource consumption

---

## Real-Time Progress Tracking

Integrated Redis Pub/Sub with Socket.IO to deliver processing updates instantly.

---

## Adaptive Streaming

Built a complete HLS transcoding and playback pipeline with multiple quality variants.

---

## Subtitle Delivery

Configured S3 policies and WebVTT support to enable browser-native subtitle playback.

---

## AI Integration

Combined Whisper and Gemini to transform uploaded videos into searchable and interactive content.

---

# 📂 Project Structure

```bash
transcodex/
│
├── frontend/
│   │
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   │
│   ├── public/
│   └── package.json
│
├── backend/
│   │
│   ├── prisma/
│   │
│   └── src/
│       ├── controllers/
│       ├── generated/
│       ├── lib/
│       ├── queue/
│       ├── routes/
│       ├── utils/
│       ├── workers/
│       └── index.ts
│
├── worker/
│   │
│   ├── prisma/
│   │
│   └── src/
│       ├── generated/
│       ├── lib/
│       ├── processors/
│       ├── utils/
│       ├── upload.ts
│       ├── worker.ts
│       └── index.ts
│
└── README.md
```

## Folder Responsibilities

### frontend/

Contains the user-facing application built with Next.js.

Responsibilities:

- Video upload UI
- Processing progress UI
- HLS video player
- Captions display
- AI transcript chat
- WebSocket client integration

---

### backend/

Acts as the API gateway for the platform.

Responsibilities:

- Multipart upload orchestration
- Video metadata management
- Queue job creation
- WebSocket server
- Transcript chat API
- Database access through Prisma

---

### worker/

Dedicated media processing service.

Responsibilities:

- Download raw videos from S3
- Generate thumbnails
- Generate captions
- Generate transcripts
- HLS transcoding
- Upload processed assets to S3
- Emit realtime progress events

---

# 🚀 Local Setup

## 1. Clone Repository

```bash
git clone https://github.com/yourusername/transcodex.git

cd transcodex
```

---

## 2. Install Dependencies

```bash
pnpm install
```

---

## 3. Configure Environment Variables

### Backend

```env
DATABASE_URL=

AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=

REDIS_URL=
REDIS_HOST=
REDIS_PORT=

PORT=8000

GEMINI_API_KEY=
```

### Frontend

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

### Worker

```env
DATABASE_URL=

AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=

REDIS_URL=
REDIS_HOST=
REDIS_PORT=
```

---

## 4. Run Migrations

```bash
cd backend

pnpm prisma migrate dev
```

---

## 5. Start Services

Backend:

```bash
cd backend

pnpm dev
```

Worker:

```bash
cd worker

pnpm dev
```

Frontend:

```bash
cd frontend

pnpm dev
```

---

# 📈 Future Improvements

* Distributed worker pools
* GPU accelerated encoding
* Semantic transcript search
* Timeline-aware AI responses
* Video recommendations
* CDN integration
* Automatic chapter generation
* AI-generated summaries

---

# 🌐 Deployment

| Service  | Provider         |
| -------- | ---------------- |
| Frontend | Vercel           |
| Backend  | Railway / Render |
| Database | Neon PostgreSQL  |
| Redis    | Upstash          |
| Storage  | AWS S3           |

---

# 📜 License

MIT License

---

# 🙏 Acknowledgments

Built to explore the engineering challenges behind modern video platforms, including media processing, adaptive streaming, distributed systems, real-time communication, and AI-powered content understanding.
