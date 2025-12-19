- [ ] **Restreamer Hardening**
  - [ ] Create/Download a placeholder "Station ID" image (1920x1080 PNG).
  - [ ] Configure Restreamer to use this image as `Fallback`.
  - [ ] Update Restreamer process options to increase `Analysis Duration` and `Probesize` (ffmpeg buffers).
  - [ ] Verify Output Transcoding is set to `H.264` (not Copy) and `AAC` (not Copy).

- [ ] **Owncast Optimization**
  - [ ] Create `s3_config_template.yaml` in `config/owncast` as a reference for future scalability.
  - [ ] Tune `hls_segment_length` to 2s or 3s (lower latency vs stability tradeoff).

- [ ] **Integration Test**
  - [ ] Simulate "Emergency Interrupt": Start OBS stream -> Switch Restreamer Source -> Verify Client Player continuity.
