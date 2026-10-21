# mp-dance-lora

**Record human dance motion with MediaPipe, analyze it on Kaggle, and build models that can mimic motion.**

\`mp-dance-lora\` explores a reproducible pipeline from real-world movement to structured pose data and motion-learning experiments. The first goal is to record dance reliably; the next is to measure movement and prepare a dataset for a model that can imitate or generate similar motion.

> **Current status:** project plan and data contract. The pipeline below describes the intended workflow; it does not imply that capture, Kaggle analysis, or LoRA training is already implemented.

## Goal

Turn dance into data that can be stored, inspected, compared, and learned from:

1. Capture dance from a camera or video with MediaPipe Pose.
2. Save frame-level landmarks and timing to \`dance.poses.sqlite\`.
3. Export a portable JSONL dataset.
4. Check FPS, timestamps, missing frames, landmark confidence, and motion continuity.
5. Analyze sessions in Kaggle and export metrics.
6. Train and evaluate a motion imitation model.

## Pipeline

\`\`\`text
Camera / video
      |
      v
MediaPipe Pose
      |
      +----> dance.poses.sqlite  (local source of truth)
      |
      v
JSONL export (one record per frame)
      |
      v
Validation: timestamps / FPS / landmarks / gaps
      |
      v
Kaggle: quality analysis + motion features
      |
      v
Dataset preparation
      |
      +----> pose-to-motion / sequence model
      |
      +----> image or video LoRA, if paired media and captions are available
\`\`\`

## Data design

### SQLite

Use \`dance.poses.sqlite\` as the local source of truth. Keep sessions and frames identifiable so recordings can be resumed, audited, and exported consistently.

A frame record should preserve:

- \`session_id\`: stable identifier for one recording
- \`frame_index\`: original frame order
- \`timestamp_ms\`: capture or source-media timestamp
- \`source_fps\`: source or requested capture FPS
- \`width\`, \`height\`: source frame dimensions when available
- \`landmarks\`: MediaPipe landmark coordinates and available visibility/presence values
- \`schema_version\`: data-contract version

Store missing or failed detections explicitly rather than silently dropping frames. Keep original coordinates and any normalized/transformed representation distinguishable.

### JSONL

Export one frame per line. Example contract (illustrative, not yet a generated dataset):

\`\`\`json
{
  "schema_version": "1.0",
  "session_id": "dance-session-001",
  "frame_index": 0,
  "timestamp_ms": 0,
  "source_fps": 30.0,
  "image": { "width": 1280, "height": 720 },
  "landmarks": [
    { "name": "nose", "x": 0.51, "y": 0.18, "z": -0.02, "visibility": 0.99 }
  ]
}
\`\`\`

The example shows one landmark only; a real record should contain the available landmark set. Document whether coordinates are normalized, image-relative, or world coordinates.

## FPS and recording quality

Do not trust the requested FPS alone. Calculate observed frame intervals from timestamps:

\`\`\`text
observed_fps = 1000 / median(timestamp_ms[i] - timestamp_ms[i-1])
\`\`\`

Report at least:

- requested/source FPS and measured FPS
- timestamp interval distribution and gaps
- missing or duplicate frames
- landmark detection rate and confidence/visibility distribution
- implausible coordinate jumps and discontinuities
- usable duration and rejected-frame ratio

For variable-frame-rate video, preserve source timestamps instead of manufacturing evenly spaced timestamps. If resampling is needed for training, retain the original timeline and document the resampling policy.

## Kaggle analysis

Use Kaggle notebooks for repeatable, shareable analysis—not as the only copy of the raw data.

Suggested outputs:

- \`frame_quality.csv\`: frame-level quality checks
- \`session_metrics.csv\`: per-session FPS, duration, missingness, and motion summaries
- \`motion_features.csv\`: derived velocity, acceleration, joint angles, and sequence features
- plots for timestamp intervals, landmark confidence, and motion trajectories

Split training and evaluation by **recording/session or performer**, not randomly by individual frames. Adjacent frames are highly correlated and random frame splits can leak nearly identical poses into both sets. Upload only media and pose data that you have permission to use; remove identifying information when it is not needed.

## What does “mimic motion” mean?

Choose the learning path based on the intended output:

| Intended result | Data needed | Suitable direction |
| --- | --- | --- |
| Reproduce or generate a dance pose sequence | Time-ordered landmarks, timestamps, performer/session metadata | Pose-sequence or motion-generation model |
| Map a pose sequence to an avatar | Landmark sequences plus a rig/retargeting definition | Pose-to-avatar retargeting |
| Generate images or video in a dance style | Consented/licensed images or video, captions, and optionally paired pose/control data | Image/video model fine-tuning or LoRA |

**Important:** MediaPipe landmark JSONL alone is not a standard image-generation LoRA training dataset. A LoRA for image/video generation needs suitable media and captions, while motion imitation from landmarks is a separate modeling task. Keep these paths separate until the target model and evaluation criteria are decided.

## Implementation order

1. [Issue #2 — Record MediaPipe Pose to SQLite](https://github.com/bonsai/mp-dance-lora/issues/2)
2. [Issue #3 — Define SQLite/JSONL schema and exporter](https://github.com/bonsai/mp-dance-lora/issues/3)
3. [Issue #4 — Evaluate FPS, data quality, and motion features on Kaggle](https://github.com/bonsai/mp-dance-lora/issues/4)
4. [Issue #5 — Decide model target, training data, and evaluation](https://github.com/bonsai/mp-dance-lora/issues/5)
5. [Epic #1 — End-to-end capture-to-model pipeline](https://github.com/bonsai/mp-dance-lora/issues/1)

## Success criteria

- A recording produces a SQLite session with ordered, timestamped frames.
- JSONL can be exported and validated against the same schema.
- Reported FPS is measured from timestamps and recording gaps are visible.
- Kaggle analysis can reproduce session-level quality metrics.
- Training/evaluation splits avoid session leakage.
- The selected model can be evaluated against a documented baseline.

## Privacy and dataset responsibility

Record only with appropriate consent. Check licenses and permissions before uploading recordings to Kaggle or using them for training. Treat raw video as more identifying than pose landmarks, but do not assume landmark data is automatically anonymous.

## Related

- Repository: https://github.com/bonsai/mp-dance-lora
- MediaPipe Pose: https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker
- Kaggle: https://www.kaggle.com/
