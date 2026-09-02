---
layout: post
title: "A Forward Model for Simulated High-Speed AFM"
---

<p align="center">
  <img src="/images/2026-09-02/sim_afm_video2.gif" alt="Simulated HS-AFM video of αVβ3" width="260px" />
</p>
*Simulated HS-AFM video generated from the 615-frame αVβ3 conformer library, subsampled from the v11 render. The z-blur and blur-isotropy corrections described below were applied after this version.*


The forward arm of AFMFold converts a conformer library into a simulated HS-AFM video. It exists to test the imaging model independently of the fitting problem: if the same library and the same tip parameters that fit a real recording also reproduce its visual character, the imaging model is not the limiting error. This report covers the render pipeline, the parameters that were calibrated against real data, and the measured agreement between simulated and real frames.

## Pipeline

The renderer takes a folder of PDBs plus a `library.json` giving per-frame CVs and trajectory order, the same input contract the template-matching arm consumes, and runs four stages.

1. Ingest the conformer library in trajectory order.
2. Stabilize orientation.
3. Forward-render through the afmfold hard-sphere dilation imaging model with z-blur and z-noise.
4. Stylize on a single canvas: zoom, substrate noise, blur, slant, row jitter, flash streaks, soft-clip, copper colormap.

## Orientation stabilization

A surface-bound molecule imaged by HS-AFM lies on a consistent face and reorients in discrete steps. A per-frame render taken straight from fitted coordinates does neither: it rolls between faces and spins smoothly.

The stabilization stage applies a five-step lock. PCA-flatten places the smallest principal axis vertical, so the molecule lies flat. A side-lock keeps the chain A face down. Long-axis sign alignment between consecutive frames removes PCA eigenvector sign flips. Stepwise yaw with a 50° threshold, a 20-frame minimum dwell and a 30° per-step cap replaces smooth rotation with discrete reorientation. Head xy positions are Gaussian smoothed at σ = 8 frames. Over the 1266 frames of video 2 this commits three discrete 30° yaw steps.

## Canvas and z-axis calibration

Three corrections were made against real Linz HS-AFM data.

The original 35 px canvas covers 34 nm at 0.98 nm/px, and the stabilized coordinates reach an xy span of 26 nm at full extension, so extended frames were clipping at the canvas edge. The canvas was enlarged to 60 px, covering 58.8 nm.

The stylizer originally generated substrate noise on a large canvas and pasted the molecule render into a smaller inset, which left a visible rectangular border whenever the molecule extended past the inset edge. All eight stylization operations now run on one 280 × 280 canvas at one resolution.

The z-axis was too sharp. The renderer had xy blur of about 0.1 nm and z-noise of 0.05 nm, against real HS-AFM which is limited by cantilever-oscillation averaging and carries 0.3 to 0.5 nm RMS z-noise. A z-blur of 0.8 nm applied to the dilated height map before per-frame normalization, with z-noise of 0.35 nm, brings the simulated z-axis into that range.

A fourth correction removed a horizontal artifact. The post-dilation blur had been tuned anisotropically at σx = 1.2, σy = 0.7 for frames whose orientation was unconstrained. After the side-lock and stepwise-yaw stage the long axis is consistently along x, and the anisotropic blur visibly squashed the molecule horizontally. The default is now isotropic at σx = σy = 1.0.

![Anisotropic versus isotropic post-dilation blur](/images/2026-09-02/sim_afm_v16_isotropic_blur.png)
*Four frames of the simulated video rendered with anisotropic blur (top row, σx = 1.2, σy = 0.7) and with isotropic blur (bottom row, σx = σy = 1.0). The anisotropic setting compresses the molecule along its locked long axis.*

## Measured agreement with real frames

The fitted PDBs from the template-matching arm were forward-rendered at tip radius 1.5 nm, tip angle 20°, noise 0.1 nm and resolution 0.98 nm/px, then scored frame by frame against the real Linz recordings by cosine similarity.

| | sim vs real | random baseline |
|---|---|---|
| video 1 | 0.824 | 0.65 |
| video 2 | 0.722 | 0.43 |

The library and the imaging model jointly reproduce the real data above the random baseline in both recordings, with a larger margin in video 2.

A tip-sharpness sweep at 1, 2 and 5 nm gives sim/real correlations of 0.648, 0.724 and 0.748.

![Simulated frames at three tip radii](/images/2026-09-02/sim_afm_tipsweep.png)
*The same conformer rendered at tip radius 1, 2 and 5 nm, with the mean sim/real correlation for each. The blunter tip reproduces the real scan more closely.*

The blunter 5 nm tip gives the higher agreement with real frames, while the 2 nm tip is the validated default for template matching, where a sharp tip is needed to discriminate between conformers. The two criteria select opposite ends of the range, which places an upper bound on how literally the hard-sphere dilation model can be read.

## Contact mechanics

Hard-sphere dilation ignores indentation. Hertzian contact was computed across the HS-AFM imaging-force range with R_eff = 0.857 nm and E* = 1.09 GPa.

| applied force | indentation | contact radius | peak pressure |
|---|---|---|---|
| 50 pN | 0.111 nm | 0.309 nm | 250 MPa |
| 100 pN | 0.177 nm | 0.389 nm | 315 MPa |
| 200 pN | 0.281 nm | 0.490 nm | 397 MPa |

Indentation reaches 5 to 14 % of the 2 nm probe-sphere height, and every value sits below the roughly 1 nm vertical noise floor of HS-AFM. Hard-sphere dilation is therefore a defensible first-order imaging model in this regime, and the Hertzian correction is a quantifiable systematic rather than a confounder.

## Consistency between the forward and inverse arms

The forward renderer and the overlay renderer consume different post-processing variants of the same fitted coordinates. The renderer reads `fitted_coords_stable.npy`, produced by the orientation-stabilization stage, while the overlay renderer defaulted to `fitted_coords_smooth.npy`, which has had temporal smoothing but no PCA-flatten or side-lock. At frame 100 the head-to-tail xy axis differs by 66° between the two variants, which appeared as a rotational misalignment in the combined figure. Both paths now project the same coordinate variant.

## What remains

Three items are open. The 0.72 to 0.82 sim/real ceiling has not been decomposed into imaging-model error and library-coverage error. A contact-mechanics correction flag on the renderer would let the Hertzian offset be applied rather than only bounded. All calibration to date uses one instrument and two recordings, so the tip and noise parameters are fitted to a single source.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
