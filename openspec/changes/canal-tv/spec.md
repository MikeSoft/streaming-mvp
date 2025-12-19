# Canal TV: Zero-Downtime Switching & S3 Scalability

## Problem Statement
The current prototype (Jellyfin -> ErsatzTV -> Restreamer -> Owncast) functions correctly as a headend but suffers from two critical limitations identified during testing:
1.  **Stream Drops:** Transitioning between ErsatzTV programs or switching to Live inputs causes Restreamer to momentarily stop the output stream, forcing Owncast clients to buffer or reload.
2.  **Scalability Bottleneck:** Serving HLS directly from the single Docker host limits concurrency to the server's uplink bandwidth (~50-100 users).

## Proposed Solution
We will harden the pipeline to achieve "Broadcast Grade" reliability and "Netflix Grade" scalability by implementing **Restreamer Fallback** and **Owncast S3 Offload**.

### 1. Zero-Downtime Switching (The "Glitchless" Switch)
To prevent stream drops during input transitions, we will configure Restreamer's **Internal Buffer** and **Fallback Image** mechanisms.
*   **Mechanism:** When the primary input (ErsatzTV) has a gap (file switch), Restreamer will hold the last valid frame or switch to a static "Station ID" image instantly, keeping the RTMP output connection to Owncast alive.
*   **Audio/Video Normalization:** We will enforce strict H.264/AAC transcoding on the Restreamer output to ensure the stream parameters never change, even if the source file codecs do.

### 2. Massive Scalability (S3 + CDN)
To support thousands of viewers, we will decouple the "Serving" layer from the "Generation" layer.
*   **Current:** Viewers -> Docker Host (Limited by Upload Speed).
*   **Proposed:** Viewers -> CDN -> S3 Bucket <- Owncast.
*   Owncast has built-in support for S3 Object Storage. We will configure it to write HLS segments directly to an S3-compatible bucket (e.g., AWS S3, DigitalOcean Spaces, Cloudflare R2).

## Implementation Plan

### Phase 1: Hardening Restreamer (Local)
1.  **Configure Fallback:** Set up a "Technical Difficulties" or "Station Logo" image in Restreamer.
2.  **Increase Buffers:** Tune `ffmpeg` input buffers in Restreamer to tolerate 3-5s of silence/latency.
3.  **Transcode Enforcement:** Lock output settings to `1080p60`, `6000kbps`, `AAC 160k` regardless of input.

### Phase 2: Scalability Prep (Configuration)
1.  **S3 Configuration:** Update `owncast` config to enable External Storage. (We will mock this or set up a placeholder config since we don't have S3 credentials provided yet, but the system will be "S3 Ready").
2.  **Test S3 Logic:** Verify Owncast attempts to push segments to the external endpoint.

## Verification
- **Test 1 (Gap Test):** Intentionally stop ErsatzTV for 5 seconds. Owncast should show the Fallback Image, not a loading spinner/error.
- **Test 2 (Switch Test):** Switch from ErsatzTV to OBS Live input. Transition should be seamless on the client side.
