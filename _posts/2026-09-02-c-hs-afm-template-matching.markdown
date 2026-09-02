---
layout: post
title: "Template Matching HS-AFM Video Against a Conformer Library"
---

The inverse arm of AFMFold takes a real high-speed AFM recording and a conformer library and returns, for every frame, the conformer and rigid-body orientation that best explain the observed image. The output is a per-frame structural trajectory extracted from a movie of a single molecule, with no network training and no per-dataset fitting parameters beyond a tip radius. This report covers the two recordings, the four errors that had to be removed before the fit was usable, and the experiment that shows the fit responds to the data rather than to the library.

## Data and the failure of the trained-network approach

Two HS-AFM recordings of surface-bound αVβ3 from Linz were used, of 409 and 1296 raw frames at 1 frame per second. The first 30 frames of each are discarded for scan-window stabilization, leaving 379 and 1266 analysed frames, referred to below as V1 and V2.

The first inference architecture was a C8-equivariant CNN trained on pseudo-AFM images rendered from the steering trajectory, predicting CVs directly from a frame. On real Linz data its CV predictions are near-constant regardless of input, which is a simulation-to-real domain gap rather than a training failure: the same network separates the simulated images it was trained on. Direct correlation matching against a pseudo-AFM rendering of the library reaches per-frame correlations up to 0.967 on the same data and is the method used throughout.

## Fitting

Each frame is matched against a pseudo-AFM rendering of every library conformer by cosine similarity, and the best conformer is then refined over 2048 uniformly sampled SO(3) rotations. On an A4500 this takes about 10 minutes per video; on CPU it takes about 4 hours.

Run in that form, the fit gives mean correlations of 0.691 on V1 and 0.797 on V2, and 37.8 % of frames show the head and tail interchanged relative to their neighbours. Four separate corrections raise this to a usable trajectory.

**Head anchoring.** Centering the fitted structure on its center of mass offsets it from the molecule by more than 2 nm, because the legs carry the center of mass away from the head. Tracking the head in the AFM frame by Gaussian smoothing followed by the centroid of the largest connected component, then aligning the PDB headpiece (αV residues 1 to 440, β3 residues 1 to 350) to that position, raises mean correlation to 0.969 on V1 and 0.941 on V2. The tracked head position is stable at 0.13 nm of drift per frame. Tip radius was swept at 1, 2, 3 and 5 nm; all values give correlations between 0.93 and 0.98, with 2 to 3 nm optimal, and 2 nm is the default.

**Library sampling.** The pseudo-AFM library was generated with `generate_images(batch_size=16, dataset_size=500)`, which processes only the first 31 of 309 conformers. The resulting label CV range was 78.6 to 80.8 Å instead of the library's true 52.9 to 85.0 Å, so 90 % of the library's conformational range was never presented to the matcher. Setting `batch_size=1`, which gives two steps per epoch over 309 conformers, covers the full library. This single parameter silently invalidated every fit made before it was found.

**Flip resolution.** The original flip-resolution routine compared each frame to the previous frame, which accumulates error: mean drift 1.01 nm, maximum 10.44 nm, with 399 of 1266 V2 frames displaced by more than 1 nm. Anchoring flip selection to the tracked AFM head position instead removes the drift entirely, and reveals that zero flips were actually required. The routine had been corrupting coordinates that were already correct. After head anchoring the one remaining discrete degree of freedom is a 180° rotation about the vertical axis through the head. Comparing each frame's tail direction to a rolling 21-frame median with 0.05 hysteresis corrects 108 V1 frames (28.5 %) and 229 V2 frames (18.1 %).

**Temporal smoothing.** The per-frame SO(3) fit over-rotates to match image noise. The first attempt at smoothing reassigned conformers by sliding-window mode while keeping each frame's own rotation, which made jitter worse by 11 % on V1 and 19 % on V2, because a conformer swapped under an old rotation is a discontinuity. Applying a rolling median of window 7 directly to the fitted coordinates, per atom and per dimension, reduces frame-to-frame mean atom displacement from 45.2 to 10.5 Å on V1 and from 38.8 to 9.9 Å on V2. A window of 7 frames is 0.7 s, shorter than the conformational transitions being measured. The median smears the head xy position by about 1 Å per frame, which is invisible in V1 and cumulative over V2's 1266 frames, so a per-frame head re-anchor follows the smoothing step.

![Frame-to-frame jitter before and after rolling-median smoothing](/images/2026-09-02/temporal_jitter_comparison.png)
*Frame-to-frame mean atom displacement over time (left) and its distribution (right) for V1 (top) and V2 (bottom), for the per-frame fit and after rolling-median smoothing.*

The final fits are V1 at 379 frames and mean correlation 0.965, and V2 at 1266 frames and mean correlation 0.939.

## The fit responds to the data, not to the library

A template-matching method returns whatever is in its library, so the question is whether the returned trajectory carries information about the recording. The test is to change the library and see whether the two recordings respond differently.

The v6 library contains 309 conformers from the extend-steering run and no bent conformers. The v7 library adds 306 bent-steering conformers, giving 615. Both videos were re-fitted against each. Using bands of CV0 < 50 Å for bent-closed, 50 to 70 Å for intermediate and above 70 Å for extended-closed, the results are as follows.

| | V1 bent fraction | V2 bent fraction |
|---|---|---|
| v6 library (309 frames, no bent conformers) | 12.4 % | 12.5 % |
| v7 library (615 frames) | 43.5 % | 18.6 % |

![Library coverage and matched-CV distributions for v6 and v7](/images/2026-09-02/library_coverage_v6_v7.png)
*Library CV0 coverage for v6 and v7 with the 1JV2 reference (top left), state occupancy for both videos under both libraries (top right), and matched-CV0 distributions for V1 (bottom left) and V2 (bottom right).*

The two recordings start from the same bent fraction under the biased library and diverge under the complete one: V1 shifts by 31 percentage points, V2 by 6. If the library were dictating the answer, both would shift by the same amount. The divergence is the discriminating evidence that the fit is reading the recordings.

The corollary is that the bent fraction reported under v6 was an artifact of missing templates, and any state fraction from this pipeline is only as complete as the library behind it. The same argument bounds what can be said about the extended-open state, for which the library contains no templates at all.

## What remains

The pre-registered falsification, in which conformers are assigned uniformly at random and the resulting correlation is expected to fall below 0.4, has not been run. It costs a few CPU hours and should be run before publication.

The full claim rests on two recordings from one instrument and one laboratory. A second independent HS-AFM dataset would test transferability directly.

An AlphaFold-Multimer ablation, estimated at roughly 50 GPU-hours, would establish what a prediction-only baseline recovers from the same recordings.

Finally, the rendered overlay of fitted structures on AFM frames is a pipeline-consistency demonstration and not fit-quality evidence, because the projection is constructed from the same coordinates the imaging model consumed. The correlations of 0.965 and 0.939 against the real frames are the only fit-quality numbers.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
