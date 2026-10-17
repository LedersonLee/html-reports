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
