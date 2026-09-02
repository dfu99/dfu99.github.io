---
layout: post
title: "One Genu-Hinge Morph Recipe Across Five Integrins"
---

<p align="center">
  <img src="/images/2026-09-02/multi_integrin_morph.gif" alt="Bent to extended morph for five integrin heterodimers" width="640px" />
</p>
*Incremental genu-hinge morph applied to five integrin heterodimers from their deposited bent or half-bent structures. Blue is the headpiece, aqua the upper leg, orange the lower legs.*

The route-A work built an extended αVβ3 endpoint by swinging the lower legs about the genu hinge in small increments with a vacuum minimization after each step, because a one-shot rigid-body rotation leaves a knee clash no minimizer descends. That recipe was written for one heterodimer with hand-checked domain boundaries. This report covers its generalization to five integrins with boundaries transferred by structural alignment, the validation against the published αVβ3 endpoint and against a cryo-EM extended structure, and the two variants whose boundary transfer is not reliable.

## Method

Domain boundaries for each target were obtained by aligning its chains against the αVβ3 reference and transferring the route-A genu, upper-leg and lower-leg definitions across the alignment. Each morph then applies the same incremental protocol: swing the lower legs about the genu axis in steps of roughly 9°, vacuum-minimize under ff14SB after each step, and stop when the genu angle passes 150°. All runs are CPU-only and take 35 to 45 minutes each.

Acceptance requires four conditions: the genu angle reaches at least 150°, it increases monotonically to within 5°, long-axis extent grows by at least 20 %, and the potential energy stays finite at every step.

## Results

| variant | start PDB | boundary confidence | genu bent to extended | extent bent to extended | Rg bent to extended | steps |
|---|---|---|---|---|---|---|
| αVβ3 | 1JV2 | high | 41.2° to 176.2° | 130.0 to 209.9 Å | 39.1 to 66.6 Å | 16 |
| αIIbβ3 | 3FCS | high | 36.3° to 179.4° | 118.5 to 207.3 Å | 39.0 to 63.5 Å | 17 |
| α5β1 | 7NXD | high | 78.0° to 178.2° | 160.2 to 214.7 Å | 48.7 to 63.3 Å | 12 |
| αMβ2 | 7USM | low | 48.2° to 172.3° | 109.0 to 207.9 Å | 39.8 to 61.4 Å | 15 |
| αXβ2 | 5ES4 | low | 12.9° to 175.5° | 137.3 to 238.6 Å | 54.8 to 80.3 Å | 20 |

All five pass all four acceptance conditions.

![Bent to extended morph across five integrin variants](/images/2026-09-02/variant_extension_summary.png)
*(A) Genu angle against cumulative leg swing for each variant, with the 150° extended threshold marked. (B) Potential energy against cumulative leg swing. (C) Long-axis extent from bent start to morphed extended endpoint, with experimental extended cryo-EM references marked for αVβ3.*

Panel A shows the genu angle rising monotonically in every variant, with the offsets between curves set by each starting structure. Panel B shows the energy staying near 10⁴ kJ/mol for the whole protocol, with two transient excursions to 1.5 and 3.1 × 10⁶ kJ/mol in the αIIbβ3 run that the next minimization step removes rather than accumulating. Final energies are 47,000 to 75,000 kJ/mol.

The five starting structures span a wide range of initial compactness, and the morph absorbs that range by taking a different number of steps. α5β1 in cryo-EM starts already at a genu angle of 78° and an extent of 160 Å, so it needs 108° of total swing over 12 steps. αXβ2 starts at 12.9° and 137 Å and needs 180° over 20 steps. This is consistent with an earlier first-principles measurement on the same deposited structures, in which α5β1 buries 10,555 Å² of head-leg surface against 13,442 Å² for αVβ3 and 15,298 Å² for αIIbβ3, and has a head-tail centroid distance of 73.9 Å against 42.6 and 37.2 Å. The "half-bent" description in the 7NXD deposit is supported by the geometry rather than only by the annotation.

## Validation

The αVβ3 run is a positive control against the previously published route-A endpoint, which was built with hand-checked boundaries. The generalized pipeline gives a radius of gyration of 66.57 Å against 67.32 Å and a long-axis extent of 209.9 Å against 211.36 Å, deviations of 1.1 % and 0.7 %. Alignment-transferred boundaries reproduce the hand-checked result.

αVβ3 is also the only variant with a deposited extended experimental structure available for comparison. Against 8XEN, a cryo-EM structure of the αVβ3 complex in an extended conformation, the morph gives an extent of 209.9 Å against 214.5 Å, a difference of 4.6 Å, and a radius of gyration of 66.6 Å against 62.4 Å. The genu angles differ more: 176.2° for the morph against 145.3° for 8XEN. The morph produces a straighter leg than the experimental extended structure, which is expected because the protocol swings until the acceptance threshold is passed rather than to a target angle, and the vacuum model carries no solvent or ion term that would favor an intermediate angle.

## What remains

Three limitations bound these endpoints.

The morph is a vacuum minimization with no solvent, no ions and no dynamics. Each endpoint is a clash-free starting structure for a solvated simulation, not an equilibrated conformation. The αVβ3 endpoint has been carried through that step; the other four have not.

Boundary transfer is reliable for αVβ3, αIIbβ3 and α5β1, with leg-alignment scores of 1357, 663 and 353, and is not reliable for αMβ2 and αXβ2 at 137 and 146. The β2 integrins carry an inserted αI domain that the αV-based reference does not, and their boundaries need manual curation before their endpoints are used quantitatively. Their morphs pass the geometric acceptance conditions, which tests the protocol rather than the boundaries.

Only αVβ3 has an experimental extended reference. The 4.6 Å extent agreement against 8XEN is a single-variant check, and the 31° genu discrepancy in the same comparison indicates the morph endpoint is over-straightened relative to a real extended integrin.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
