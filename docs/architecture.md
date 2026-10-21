# Architecture: Python + JavaScript + Rust

PlantUML source: [`docs/architecture.puml`](architecture.puml)

## Responsibilities

- **Python** — camera-frame ingestion, MediaPipe Pose inference, timestamped pose recording, SQLite/JSONL export, offline evaluation.
- **Rust** — bounded/latest-pose input, continuous motion playback when the performer is still, real-time rendering and projector output.
- **JavaScript** — control panel for start/stop, calibration, trail/color/loop parameters, and performance status.
- **PlantUML** — source-of-truth architecture diagram in `docs/architecture.puml`.

## Live data path

`Camera → Python/MediaPipe → latest-pose slot → Rust motion engine → Rust renderer → projector`

The pose slot must be bounded and overwrite stale data. Capture/inference must not block rendering. SQLite/JSONL writes and Kaggle training stay off the critical render path.

## Initial transport decision

Keep the boundary transport-agnostic in the diagram. For a first local prototype, use a small localhost WebSocket or OSC message carrying timestamp + normalized landmarks; benchmark before choosing a permanent protocol. Do not send full video frames from Python to Rust unless the renderer specifically needs them.

## Performance targets to measure

- Capture-to-projection latency (median and p95)
- Displayed FPS and frame-time variance
- Pose inference duration and dropped-frame count
- Memory usage during a long-running session
- Recovery behavior when pose detection disappears

The initial target is stable real-time output on a 16 GB RAM machine, not real-time diffusion/LoRA generation.
