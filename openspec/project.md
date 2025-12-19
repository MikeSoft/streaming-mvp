# Project Context

## Purpose
The goal of this project is to create a robust, online TV channel prototype capable of 24/7 broadcasting. It is designed to handle both scheduled programming (like a traditional TV station) and real-time live interruptions (Breaking News/Live Events). The system is built to be self-hosted on a Linux Server with NVIDIA GPU acceleration, but developed and tested locally on Windows via Docker.

## Tech Stack
- **Infrastructure:** Docker & Docker Compose (Container Orchestration).
- **Media Management:** Jellyfin (Organization of video assets).
- **Scheduling/Playout:** ErsatzTV (Linear TV channel generation from media library).
- **Master Control/Mixer:** Restreamer (Switching between scheduled feed and live RTMP inputs).
- **Distribution/Frontend:** Owncast (Public-facing web player and chat).
- **Hardware Acceleration:** NVIDIA NVENC (via NVIDIA Container Toolkit).
- **Live Input Source:** OBS Studio (for pushing live streams to Restreamer).

## Project Conventions

### Architecture Patterns
- **"All-in-Docker" Architecture:** All core services run as isolated containers within a shared network (`tv-network`).
- **Master Control Workflow:**
    - **Primary Feed:** ErsatzTV generates an HLS stream.
    - **Mixing:** Restreamer ingests the primary feed and accepts secondary RTMP inputs (Live).
    - **Output:** Restreamer transcodes (normalizing Audio/Video codecs) and pushes a single RTMP stream to Owncast.
- **Transcoding Pipeline:** Heavy transcoding tasks (ErsatzTV playout, Restreamer mixing) are offloaded to the NVIDIA GPU to preserve CPU for logic and file I/O.

### Configuration Standards
- **Video Standard:** 1080p (1920x1080) at 30fps or 60fps.
- **Distribution Codecs:**
    - **Video:** H.264 (High Profile, 4500-6000 kbps).
    - **Audio:** AAC (128-160 kbps, 44.1/48kHz).
- **Network:** Services communicate via internal Docker DNS names (`jellyfin`, `ersatztv`, `owncast`) instead of IP addresses.

### Testing Strategy
- **Local Development:** Verify streaming flow on Windows + Docker Desktop (WSL 2).
- **Integration Tests:** Manually verify the "handshake" between components:
    - Jellyfin -> ErsatzTV (API Connection).
    - ErsatzTV -> Restreamer (HLS Stability).
    - Restreamer -> Owncast (RTMP Push & Codec Compatibility).
- **Stress Testing:** Validate high-motion content handling and seamless switching between source inputs without stream drops.

## Domain Context
- **HLS/RTMP Streaming:** Understanding of streaming protocols, buffering latencies, and codec negotiation (ffmpeg).
- **Linear TV:** Concepts of "Playout", "Schedule", "Fillers", and "Guide".
- **Master Control:** The role of switching sources in real-time while maintaining a continuous output signal.

## Important Constraints
- **Hardware Dependency:** The production and dev environments strictly require NVIDIA GPUs with NVENC support.
- **Audio Consistency:** ErsatzTV streams must have their audio transcoded in Restreamer to prevent "Invalid Argument" errors in Owncast due to codec mismatches (e.g., switching from AAC to AC3 between files).
- **Buffering:** Restreamer requires a significant input buffer (3-5s) to handle the gap when ErsatzTV switches files.

## External Dependencies
- **NVIDIA Drivers & Container Toolkit:** Essential for transcoders.
- **Docker Hub Images:**
    - `jellyfin/jellyfin`
    - `jasongdove/ersatztv:latest-nvidia`
    - `datarhei/restreamer`
    - `owncast/owncast`
