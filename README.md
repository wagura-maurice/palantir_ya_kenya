# Palantir ya Kenya

**Palantir ya Kenya** is an AI-powered surveillance system designed to monitor protest activities in Nairobi’s Central Business District (CBD). The system leverages advanced AI to process live and uploaded videos, still images (JPEG, PNG), audio recordings, and drone feeds to track police officers, paid goons, undercover operatives, and criminals infiltrating protests. By integrating with a web platform, it enables business owners to contribute CCTV footage and supports real-time and retrospective analysis to promote accountability and transparency while ensuring compliance with Kenya’s **Data Protection Act, 2019**.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Technologies](#technologies)
- [Installation](#installation)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Features

- **Multi-Media Processing**:
  - Processes live video streams (RTSP/WebRTC) from CCTV and drones.
  - Handles uploaded videos (MP4, AVI), still images (JPEG, PNG), and audio recordings (MP3, WAV) from phones and other devices.
  - Supports real-time segmentation for live video and audio streams.
- **AI-Powered Analysis**:
  - **Facial Recognition**: Identifies individuals involved in violent or unlawful actions using FaceNet (Rust ONNX).
  - **Object Detection**: Detects police uniforms and badge numbers with YOLOv8 and OCR (tesseract-rs).
  - **Audio Analysis**: Detects gunshots, chants, or screams using YAMNet (tch-rs).
  - **Event Classification**: Flags violence, suspicious behavior, or looting across videos, images, and audio.
  - **Telemetry Analysis**: Tracks movement patterns, generates heatmaps, and predicts trajectories using opencv-rust.
- **Drone Integration**:
  - Provides aerial surveillance with real-time video and audio streaming via Cloudflare Stream.
  - Supports command and control, telemetry collection, and geofencing via Cloudflare Workers Pub/Sub.
- **Web Platform**:
  - Allows business owners to upload media and view flagged events, heatmaps, and telemetry data.
  - Real-time alerts for incidents via Cloudflare Durable Objects (WebSockets).
  - Generates exportable incident reports (PDF/CSV).
- **Role-Based Access Control (RBAC)**:
  - Roles: Admin, Analyst, Contributor, Observer, Field Officer.
  - Secured with JWT authentication and Cloudflare Access.
- **Compliance & Security**:
  - Adheres to Kenya’s **Data Protection Act, 2019** with data encryption (TLS, R2) and compliance logging in D1.
  - Protected by Cloudflare WAF and IP whitelisting.
- **Future Expansions**:
  - Multilingual NLP for chant transcription.
  - Whistleblower portals with Cloudflare Access.
  - Federated identity (OAuth2) and crowd sentiment analysis.

## Architecture

The backend architecture is modular, built with Rust, and deployed on Cloudflare’s serverless infrastructure. Below is a high-level overview (see [Mermaid.js graph](#mermaid-architecture) for details):

- **Backend Core API Layer**:
  - Framework: Axum (Rust).
  - RESTful APIs for authentication, media upload, live streams, events, telemetry, users, and reports.
  - WebSocket channels for live alerts, drone feeds, and AI-detected events via Cloudflare Durable Objects.
- **AI Services**:
  - Microservices for face recognition, object detection, audio analysis, event classification, and telemetry analysis.
  - Orchestrated with Cloudflare Queues and Workers, outputs stored in D1.
- **Media Processing & Storage**:
  - Ingestion workers process videos (rust-ffmpeg), images (image-rs), and audio (hound-rs).
  - Stored in Cloudflare R2, streamed via Cloudflare Stream, metadata in D1.
- **Drone Communication**:
  - Telemetry via Cloudflare Workers Pub/Sub, video/audio streaming via Cloudflare Stream.
- **RBAC, Logging, Security**:
  - RBAC enforced via D1 and Cloudflare Access.
  - Logging with Cloudflare Workers Analytics Engine, compliance logs in D1.
  - Secured with JWT, TLS, and Cloudflare WAF.
- **Cloud Infrastructure**:
  - Fully serverless with Cloudflare Workers, Pages, Load Balancer, and CI/CD via GitHub Actions + Wrangler.

### Mermaid Architecture

```mermaid
graph TD
    A[AI-Powered Surveillance System - Backend Architecture] --> B[Backend Core API Layer]
    A --> C[AI Services]
    A --> D[Media Processing & Storage Service]
    A --> E[Drone Communication Module]
    A --> F[Role-Based Access Control]
    A --> G[Logging & Monitoring]
    A --> H[Cloud Infrastructure]
    A --> I[Security Modules]
    A --> J[Optional Future Expansions]

    B --> B1[Framework: Axum Rust]
    B --> B2[API Versioning]
    B --> B3[RESTful Endpoints & WebSockets]
    B --> B4[Middleware]
    B3 --> B3a[/api/v1/auth/]
    B3 --> B3b[/api/v1/media/upload/]
    B3 --> B3c[/api/v1/media/live/]
    B3 --> B3d[/api/v1/events/]
    B3 --> B3e[/api/v1/telemetry/]
    B3 --> B3f[/api/v1/users/]
    B3 --> B3g[/api/v1/reports/]
    B3 --> B3h[WebSocket Service: Cloudflare Durable Objects]
    B3h --> B3h1[Live Alerts]
    B3h --> B3h2[Drone Feed: Video & Audio]
    B3h --> B3h3[AI-Detected Events]
    B4 --> B4a[JWT Auth: jsonwebtoken]
    B4 --> B4b[Rate Limiting: Cloudflare Workers]
    B4 --> B4c[Role-Based Access Guard]
    B4 --> B4d[Request Validation & Sanitization: serde]

    C --> C1[Face Recognition Service]
    C --> C2[Object Detection Service]
    C --> C3[Audio Analysis Service]
    C --> C4[Event Classification Service]
    C --> C5[Telemetry Analysis Service]
    C --> C6[ML Orchestration]
    C1 --> C1a[Rust ONNX: FaceNet]
    C1 --> C1b[Video & Image Processing]
    C2 --> C2a[YOLOv8: tch-rs]
    C2 --> C2b[OCR: tesseract-rs]
    C2 --> C2c[Uniform & Badge Detection: Video/Image]
    C3 --> C3a[YAMNet: tch-rs]
    C3 --> C3b[Gunshot & Chant Detection]
    C3 --> C3c[Live & Uploaded Audio]
    C4 --> C4a[Violence Detection: Video]
    C4 --> C4b[Suspicious Behavior: Video/Image]
    C4 --> C4c[Audio Event Integration]
    C5 --> C5a[Trajectory Prediction: opencv-rust]
    C5 --> C5b[Heatmaps & Pattern Recognition]
    C5 --> C5c[Video & Drone Telemetry]
    C6 --> C6a[Cloudflare Queues]
    C6 --> C6b[Task Queue: Workers]
    C6 --> C6c[Output to D1/Channels]

    D --> D1[Ingestion Workers]
    D --> D2[Upload Queue Handling]
    D --> D3[CDN Integration]
    D --> D4[Metadata Extractors]
    D1 --> D1a[Video: FFmpeg rust-ffmpeg]
    D1 --> D1b[Image: image-rs]
    D1 --> D1c[Audio: hound-rs]
    D1 --> D1d[Real-Time Segmentation: Video/Audio]
    D2 --> D2a[Cloudflare Queues: Video/Image/Audio]
    D3 --> D3a[Cloudflare R2: Storage]
    D3 --> D3b[Cloudflare Stream: Live Video/Audio]
    D4 --> D4a[Frame Hashes: Video/Image]
    D4 --> D4b[Timestamps & EXIF]
    D4 --> D4c[Audio Metadata: Duration, Spectrograms]

    E --> E1[MQTT: Workers Pub/Sub]
    E --> E2[Command & Control: Rust]
    E --> E3[Streaming Relay: Cloudflare Stream]
    E3 --> E3a[Video Streaming]
    E3 --> E3b[Audio Streaming]

    F --> F1[Admin: Full Access]
    F --> F2[Analyst: Review Flagged Media/Reports]
    F --> F3[Contributor: Upload & View Submissions]
    F --> F4[Observer: Read-Only Public Incidents]
    F --> F5[Field Officer: Live Monitor & Drone Pilot]

    G --> G1[Log Aggregation: Workers Analytics Engine]
    G --> G2[Alerting: Workers Email/SMS]
    G --> G3[Performance Metrics: Cloudflare Metrics]
    G --> G4[Audit Trail: D1]

    H --> H1[Cloudflare Workers: Compute]
    H --> H2[Cloudflare Pages: Frontend]
    H --> H3[Load Balancing: Cloudflare]
    H --> H4[CI/CD: GitHub Actions + Wrangler]
    H --> H5[Serverless Orchestration: Workers + Durable Objects]

    I --> I1[JWT + Refresh Tokens]
    I --> I2[IP Whitelisting: Cloudflare Access]
    I --> I3[Data Encryption: TLS, R2]
    I --> I4[API Gateway + WAF: Cloudflare Gateway]
    I --> I5[Compliance Logging: D1]

    J --> J1[Multilingual NLP: Chant Transcription]
    J --> J2[Whistleblower Portals: Cloudflare Access]
    J --> J3[Federated Identity: OAuth2]
    J --> J4[Crowd Sentiment Analysis: Group Movement]
