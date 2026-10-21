# mp-dance-lora

**Low-latency dance VJ: live human video in, continuous motion and projection out.**

Capture a real person with a camera. The system should keep producing dance visuals even when the person stops moving. Rendering is replaceable: use any renderer that can display fullscreen to a projector.

## Runtime

```text
Camera
  -> MediaPipe Pose (live landmarks)
  -> motion engine
       ├─ follow live movement when detected
       └─ continue / replay / transform recorded dance when movement stops
  -> real-time renderer
  -> fullscreen projector output
```

The output must not freeze when live movement disappears. The motion engine and renderer run independently from the camera capture, so a slow inference frame does not stall projection.

## Initial constraints

- Target machine: 16 GB RAM; CPU-first and modest-GPU friendly.
- Priority: low end-to-end latency, stable frame pacing, no growing frame queue.
- Input: live raw human video capture, not just prerecorded VTuber clips.
- Output: projector-friendly fullscreen window; renderer can be replaced.
- First visual target: pose lines / silhouette / trails / repeated motion. Add a 3D avatar or generative video only if needed later.
- Rust is an option for the real-time motion/rendering layer.

## Suggested implementation

- **Capture and pose:** MediaPipe Pose Landmarker in live-stream mode; process the newest frame and drop stale frames rather than building latency.
- **Motion continuity:** maintain a small ring buffer of recent pose frames. When the person becomes still or detection is lost, loop, blend, mirror, or transform a captured motion phrase.
- **Rendering:** start with a lightweight 2D renderer. Rust + `wgpu` is a possible cross-platform GPU route; the renderer is not part of the data contract.
- **Training:** keep offline model training separate from the live performance process. Use recorded poses for analysis and motion-model experiments; do not require LoRA inference for the first live prototype.

MediaPipe supports live-stream processing and may drop input frames when needed to reduce latency. See [Pose Landmarker](https://developers.google.com/edge/mediapipe/solutions/vision/pose_landmarker). Rust's [wgpu](https://wgpu.rs/) provides cross-platform GPU rendering.

## File extensions

- `.mp4` — optional recording of the camera input
- `.jsonl` — timestamped pose frames for export and replay
- `.sqlite` — local pose/session store, e.g. `dance.poses.sqlite`
- `.ipynb` — offline Kaggle analysis and experiments
- `.safetensors` — optional trained model/LoRA weights; not required for the first live VJ
- `.vrma`, `.vmd`, or `.fbx` — optional avatar-motion interchange formats, depending on the target avatar/runtime

## Low-latency rules

1. Capture, pose inference, motion logic, and rendering must not block one another.
2. Prefer a latest-frame slot or bounded queue; drop stale frames instead of accumulating delay.
3. Measure capture-to-projection latency and displayed FPS.
4. Keep inference resolution and model complexity configurable.
5. Keep the first prototype independent of large generative models.
6. When live movement stops, continue output from the motion buffer rather than waiting for new camera motion.

## Development sequence

1. Display live pose landmarks fullscreen.
2. Record timestamped landmarks to `dance.poses.sqlite` and export JSONL.
3. Add a ring buffer and continuous playback while the performer is still.
4. Add trails, duplication, color, and projection-friendly controls.
5. Measure latency and frame pacing on the 16 GB target machine.
6. Explore pose-sequence generation or LoRA only after the live VJ loop works.

See [Issue #1](https://github.com/bonsai/mp-dance-lora/issues/1) for the end-to-end project and [Issues #2–#5](https://github.com/bonsai/mp-dance-lora/issues) for capture, data, evaluation, and model experiments.
