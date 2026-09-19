# Frame-quality check: design for a collection-time surface

Draft design for the frozen/black-camera-frame detector, scoped to also warn the operator
during/immediately after recording — not just a post-hoc dataset-inspection script. Written
2026-09-19. Not started; this is the plan to build against, checked off in
`ROADMAP_CONTRIBUTION_OPPORTUNITIES.md`'s TODO list.

## Why this shape

Reviewed in the prior session: nothing in `lerobot`'s `datasets/` module inspects decoded frame
*content* at all. `#2936` (stale, unreviewed) scores dataset quality from parquet only, no video
decode. `#4533` (CarolinePascal, open) guards against dropped PNG writes at record time, but only
checks frame *count* per camera, not frame *content*. Nothing catches a camera that freezes or
goes black mid-session while still producing frames that pass every existing check.

For UMA specifically (see the roadmap doc's 2026-09-19 entries): they're running teleop sessions
at volume right now (hiring an Operator for Data Collection, a Robotic Teleoperator, Assembly
Technicians) with no one owning data quality (Data & ML Infra Lead req open 2 months, unfilled).
The expensive failure at that scale is a frozen/black camera silently poisoning an episode that
isn't inspected until training weeks later. The fix that matters operationally is catching it
**while the operator is still at the rig**, not just before the next training run.

## Two surfaces, one detector

Build one small, pure detector function, then two call sites. Don't build a third "analyzer
suite" — that's `#2936`'s job, not this PR's.

```python
def detect_frozen_or_black_frames(
    frames: Sequence[np.ndarray],
    *,
    freeze_threshold: float = 1.0,   # mean abs pixel diff below this = "identical enough"
    black_threshold: float = 2.0,    # mean pixel value below this (0-255 scale) = "black"
    min_run_length: int = 1,         # consecutive frames needed to flag, not a single blip
) -> list[FrameQualityIssue]:
    """Flag runs of frozen (near-identical to previous) or black (near-zero mean) frames."""
```

`FrameQualityIssue` — small dataclass: `camera_key`, `start_frame_index`, `run_length`, `kind`
(`"frozen"` / `"black"`).

Threshold values need real calibration against actual camera noise floor (sensor noise means
"identical" needs a nonzero tolerance) — do this against a couple of real SO-101 recordings
before picking defaults, don't guess constants blind.

## Surface 1 — episode-boundary check (build first, smallest, safest)

Hook into `DatasetWriter.save_episode()` in `src/lerobot/datasets/dataset_writer.py:271`,
immediately before the existing image-flush/verify logic `#4533` already added there (same
function, same timing — mirror her pattern exactly, don't invent a new lifecycle point).

- After the episode's frames are on disk (or in the streaming encoder buffer) but before the
  episode is finalized, decode/read back each camera's frame sequence for the just-recorded
  episode and run `detect_frozen_or_black_frames` per camera.
- On a hit: same treatment `#4533` gives a dropped-frame episode — log a clear, loud warning
  naming the camera, the frame range, and the issue kind; do **not** silently discard by default
  (unlike `#4533`'s dropped-PNG case, a frozen camera might still be an intentional static shot in
  some setups — flag, don't assume corrupt). Gate outright discard behind an opt-in flag.
- New `RecordConfig` field, e.g. `check_frame_quality: bool = True` (mirrors existing boolean
  config flags in `lerobot_record.py:171`'s `RecordConfig`), plus the two threshold params if
  the defaults need to be tunable per rig.

This is the PR to actually ship. Small diff, reuses `#4533`'s exact hook point and failure-
handling philosophy, testable the same way (`#4533`'s "deliberately missing PNG" test pattern ->
here, "deliberately inject a duplicated/blacked-out frame into a synthetic episode buffer").

## Surface 2 — mid-session live warning (stretch goal, not the PR to lead with)

The sharper "tell the operator while they're still at the rig" version means checking *during*
`record_loop` (`src/lerobot/scripts/lerobot_record.py:228`), not just at episode end. Concretely:
in the `"observe"` timer section (`record_loop.py:302`), after `obs = robot.get_observation()`,
keep a small rolling buffer (last N frames) per camera key and run a cheap incremental version of
the same detector every tick.

**Why this is v2, not v1**:
- `record_loop` is the hot path — every camera-side check here has a real per-tick cost budget
  (the existing `CycleTimer` already tracks and warns on slow loops; this must not become the
  next slow-loop complaint).
- Surfacing the warning to the operator needs a real UI decision: `display_mode` is already
  `"rerun"` or `"foxglove"` (`RecordConfig`) — does the warning show as a Rerun overlay, a
  terminal print alongside the existing timer warnings, both? Needs a decision, not a guess.
- Episode-boundary detection (Surface 1) already gets the operator the signal "while still at the
  rig" in practice — lerobot-record's existing re-record workflow means the operator sees the
  per-episode outcome immediately after stopping anyway. Surface 2 only wins if catching it
  *mid*-episode (so the operator can abort and redo *that* episode instead of finishing it first)
  matters enough to justify the added complexity — worth confirming that's actually the gap before
  building it.

Recommendation: ship Surface 1, mention Surface 2 as a natural follow-up in the PR description
(this is also good etiquette toward CarolinePascal/DATA-14 — shows the scope was deliberately kept
narrow, not that the harder half was skipped by accident).

## Testing plan

- Unit tests on `detect_frozen_or_black_frames` directly: synthetic frame sequences with an
  injected frozen run, an injected black run, and a clean sequence (no false positives) —
  same shape as `#2936`'s "synthetic corruption detection (4 types) — PASS" and `#4533`'s
  "episode with a deliberately missing PNG... discarded" test.
- Integration test on `DatasetWriter.save_episode()`: build an episode buffer with a real injected
  frozen/black run via the existing `lerobot_dataset_factory` test fixture pattern (used
  throughout `tests/datasets/`), confirm the warning fires and (if discard is enabled) the episode
  is dropped the same way `#4533`'s test confirms cleanup via `clear_episode_buffer`.
- Real-hardware smoke test: one real SO-101 recording session with a camera physically covered
  mid-episode, confirm detection — same "real repro, not just synthetic" discipline as the PRs
  reviewed in the last batch (`#4648`'s real rollout-reward-number repro, `#4620`'s real-hardware
  inference trace).

## Non-goals (say this explicitly in the PR)

- Not a new CLI, not a competing analyzer suite to `#2936`.
- Not solving `DATA-04` (timestamp drift) or `DATA-06` (unified feature validation) — orthogonal.
- Not claiming to replace `#4533` — this is a frame-*content* check, `#4533` is a frame-*count*
  check; both belong, neither subsumes the other.

## PR description checklist (once built)

- Reference `#4533` and CarolinePascal's DATA-14 gist explicitly, framed as filling an adjacent
  gap in her lane, not opening a competing one.
- No mention of UMA/Hebbian/hiring — that reasoning stays in
  `ROADMAP_CONTRIBUTION_OPPORTUNITIES.md`, not the public PR.
- State real calibration numbers for the thresholds (measured against real camera noise, not
  guessed), same evidentiary bar as the reviewed PR batch.
