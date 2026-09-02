---
layout: post
title: "Building an αVβ3 Conformer Library by Steered MD"
---

The AFMFold framework needs an ensemble of αVβ3 ectodomain structures spanning the bent and extended states, because every downstream step (forward-rendering simulated AFM, template-matching real AFM) consumes a conformer library as its input contract. There is no extended αVβ3 crystal structure, so the ensemble has to be generated. This report covers how it was generated, what the resulting library covers, and the one region of conformational space it does not reach.

## Structure prediction returns only the bent state

Two co-folding predictors were scored against a steered-MD pull trajectory (19 frames, frame 0 bent through frame 300 over-stretched) by Kabsch Cα-RMSD and TM-score.

Protenix was run at full MSA and at 5 % subsampled MSA on PACE A100. The two depths differ by less than 0.001 TM at every frame. AF2 2.3.2 multimer with `reduced_dbs` produced 25 ranked models whose pairwise RMSD spans only 0.1 to 2.9 Å, with confidence 0.856 to 0.890 iptm+ptm. Both methods give TM = 0.96 to 0.99 against the bent frame and 0.06 against frame 300, and their TM profiles agree to within 0.03 at every intermediate frame.

![AF2 and Protenix TM-score and RMSD against pulled frames](/images/2026-09-02/af2_vs_protenix_validation.png)
*Best TM-score (top) and best Cα-RMSD (bottom) of AF2 and Protenix predictions against each pulled MD frame. The two curves are superimposed across the full range.*

MSA subsampling, which diversifies predicted conformations in other systems, produces no conformational diversity here. The bent state dominates the PDB training data for αVβ3 strongly enough that neither method samples an extended pose. Physics-based generation is therefore the only available route.

AF2 does carry usable information about which regions move. Per-residue pLDDT on the bent prediction is 96.8 in the head and thigh but 82.8 in the αV legs and tail and 83.6 in the β3 EGF and tail, and those low-confidence regions coincide with the largest per-residue displacement under pulling.

<p align="center">
  <img src="/images/2026-09-02/avb3_pulling_confidence.gif" alt="Pulled αVβ3 frames colored by AF2 per-residue confidence" width="480px" />
</p>
*Steered-MD pull frames colored by AF2 per-residue pLDDT from the bent prediction. The regions that displace most are the regions AF2 is least confident about.*

## Distance biasing is the only steering method that opens the leg

Four domain-preserving steering presets were run on αVβ3 in OpenMM on an RTX A5000, 1 ns production each at 27.9 ns/day.

| preset | mechanism | Δ α-leg angle | Δ head-tail distance |
|---|---|---|---|
| gentle_open | angle torque, k = 50 | −4.5° | |
| moderate_open | angle torque, k = 200 | −0.2° | |
| restrained_pull | position restraints plus centroid pulling | numerically unstable (NaN) | |
| cv_distance_extend | flat-bottom distance bias, k = 200 | +88.2° | +112.7 Å |

![Comparison of four domain-steering presets](/images/2026-09-02/domain_steering_comparison.png)
*Inter-domain hinge angles and centroid distances before and after each steering preset, with production energy and temperature traces. Only the flat-bottom distance bias (green) moves the geometry.*

Angle torques at k = 50 to 200 produce changes smaller than thermal noise. Restrained pulling is numerically unstable. The flat-bottom distance bias between domain centroids opens the α-leg by 88° and extends the head-tail distance by 113 Å in 1 ns, and it is the method used for every library run reported below.

## Library coverage

The library is the union of two steering runs in opposite directions from a common seed.

The extend run contributes 309 frames spanning CV0 (αV head-thigh to αV calf centroid distance) from 52.9 to 85.0 Å. The bent run, added later with targets [4.0, 3.5, 2.0] nm and k = 200, contributes 306 frames and reaches CV0 = 47.3 Å, below the 51.4 Å of the bent crystal 1JV2. The combined library is 615 frames.

![Steered MD CV trajectories and combined library manifold](/images/2026-09-02/steering_cv_trajectory.png)
*Extend-steering CV0 trajectory (top left), bent-steering CV0 trajectory (top right), β-leg extension for both runs (bottom left), and the combined 615-frame library on the CV0-CV1 plane with the 1JV2 reference marked (bottom right).*

The bent run overshoots the crystal rather than falling short of it, so the library brackets 1JV2 on the compact side. On the extended side it stops at 85.0 Å.

## The headpiece does not open under classical SMD

Extension and headpiece opening are separate transitions. CV2, the αV head to β3 head separation, distinguishes the extended-closed (EC) state from the extended-open (EO) state at a threshold of 50 Å. Three attempts were made to drive it.

The first used a preset named `cv_distance_headopen` that silently inherited the default head-tail pair list, which contains no head-head pair. CV2 stayed at 34 ± 0.4 Å over 790 ps, so the preset was not applying the bias its name described. After adding an explicit (αV head-thigh, β3 head) pair, two further runs were made. At k = 250 with a 6 nm target, CV2 moved 34.6 to 34.9 Å over 620 ps. At k = 1000 with a 0.5 nm flat bottom and the leg targets set to their current values so they do not compete, CV2 moved 35.7 to 36.6 Å over 620 ps.

![CV2 under headpiece-opening steering at k=1000](/images/2026-09-02/headopen_v5_cv2_failed.png)
*Head-head separation CV2 under the corrected headpiece-opening bias. The trace is flat on the scale of the 60 Å target.*

The observed rate is 0.07 Å/ps, which extrapolates to roughly 320 ns to reach a 60 Å target. The αVβ3 headpiece interface is too stable to be opened by a classical distance bias on a 3 ns budget at any force constant tested.

## Published structures supply no extended-open endpoint

An alternative to sampling EO is to import it. All five published full-ectodomain αVβ3 crystal structures (1JV2, 1L5G, 4G1E, 4G1M, 4MMX) were scored on the same CVs.

![Library coverage against five published αVβ3 ectodomain structures](/images/2026-09-02/library_coverage_v3.png)
*Library and fitted-trajectory CV distributions with the five published αVβ3 ectodomain crystal structures marked in red, on CV0 (top), CV1 (middle) and CV2 (bottom). The hatched region on CV2 is the EO band.*

All five sit at CV0 = 51 to 52 Å and CV2 = 36 to 37 Å. None reaches the EO threshold. Cilengitide-bound 1L5G, which has an open headpiece internally, still crystallizes bent overall. Importing EO endpoints from the PDB returns zero usable structures.

## What remains

The library covers the bent-to-extended-closed axis and does not cover extended-open. Two consequences follow for everything built on it. Any state assignment made from this library can label BC, Intermediate and EC but cannot label EO. Any free-energy or population estimate is undefined above CV0 = 85 Å.

Closing the gap requires enhanced sampling rather than a longer classical run: metadynamics, replica exchange, a string method seeded from αIIbβ3 structures, or a coarse-grained Gō-Martini model. The route-A string-method work reported separately is the current line of attack.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
