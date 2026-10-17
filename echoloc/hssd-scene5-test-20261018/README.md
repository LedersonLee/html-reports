# HSSD scene5: matched vision / acoustic / fusion review

488 official v4 test RGB observations, 61 exact camera positions × 8 headings.
This acoustic evaluation covers **one held-out scene**, not all 17 test scenes.
The fixed vision model is the completed-epoch20 UnLoc checkpoint. Predictions,
recall tables and case classifications are unchanged by this visualization update.

The likelihood selector changes only the background:

- Vision-only: maximum visual log score over yaw at each XY.
- Acoustic-only: negative minimum acoustic distance over all 36 yaw candidates.
- Fusion: negative acoustic distance **inside visual Top-K only**, following the
  selected cell-rerank or per-candidate visual-yaw-prior rule. Other indoor cells
  are grey. This is hard candidate reranking, not a probabilistic weighted sum
  or a calibrated joint posterior.

Colours use per-query/per-layer min–max normalization and the same no-white
midpoint palette. Red is higher/better. Colour values cannot be interpreted as
cross-method calibrated probabilities. Four independently togglable arrows show
GT (blue), vision (green), acoustic (orange), and fusion (purple), with native
pose/yaw positions and no display offsets.

The shared XY support has 7,108 locations. Downloadable report score fields are
gzip-compressed little-endian float64, one query per file. They are derived
analysis artifacts, not raw RIR waveforms. `review.json` describes the field
layout, coordinates, method order, and relative file paths; RGB assets and all
runtime dependencies are contained in this report directory. A current Firefox
or Chrome supporting `DecompressionStream` is required.

## Additional simple 1:1 XY fusion

The fourth background option is full-grid **simple 1:1 fusion**, separate from
the original sparse Top-K reranking view. Both scores are independently
min–max normalized across the same full common XY grid per query:

`F = 0.5 * minmax(visual max-yaw log score) + 0.5 * minmax(-acoustic min-yaw distance)`.

There is no Top-K restriction or GT-based weight fitting. The selected XY uses
the visual maximum-score yaw at that location, matching the previous fusion
heading convention. This is an arithmetic normalized-score baseline, not
calibrated posterior addition or joint XY/yaw fusion. Choosing this background
also selects its matching purple pose arrow and failure-case classification.
K is disabled for this mode; existing Top-K predictions remain unchanged.

| Method | R@0.1m % | R@0.5m % | R@1m % | R@1m30° % |
|---|---:|---:|---:|---:|
| Simple 1:1 + STFT-L1 | 8.61 | 35.66 | 43.03 | 41.60 |
| Simple 1:1 + C 0.375–3kHz | 5.12 | 36.48 | 49.39 | 46.72 |
| Simple 1:1 + C 0.5–1.5kHz | 3.69 | 33.61 | 42.42 | 40.98 |
| Simple 1:1 + C 0.5–4kHz | 4.92 | 35.66 | 48.98 | 45.70 |
| Simple 1:1 + E+C 0.5–1.5kHz | 14.14 | 41.39 | 47.54 | 46.52 |
| Simple 1:1 + E+C 0.5–4kHz | 10.66 | 36.89 | 41.80 | 40.57 |

Vision-only common-support R@1m is 64.75%; this simple fusion baseline does
not improve it. No method/weight was selected on test to claim improvement.
The extra protocol, per-query CSV, summary and **2,928** browser-field/MAP
checks are published in `equal_weight_*.json/csv`. Original binary score files,
original evaluation CSV and original vision/acoustic/Top-K metrics are unchanged.

`likelihood_verification.json` records **26,840** checks matching displayed
score maxima/reranking choices to the immutable evaluated poses. The source
CSV/array hashes and every compressed field hash are included. Browser QA checked
all six methods, both fusion policies, four K values, three background modes,
query persistence and a 360-pixel viewport.

Important existing geometry limitation: these test results still reuse the
original object-free HSSD **native stage** FP bank, with candidate source Y about
1.70–1.90 m versus observation Y=1.50 m. They do NOT use the new canonical-map
extrusion training positives being generated separately. No new model evaluation
or improvement claim is implied by the selector.
