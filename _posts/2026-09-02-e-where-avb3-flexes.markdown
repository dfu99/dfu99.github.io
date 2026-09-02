---
layout: post
title: "Where αVβ3 Flexes"
---

Two independent sources describe αVβ3 flexibility in this project: the 615-frame steered-MD conformer library, and the 1645 fitted frames recovered from two HS-AFM recordings. This report covers the per-residue flexibility map built from both, its agreement with published normal-mode analysis, how the map changes between conformational states, and a negative result on cryptic ligand-binding sites.

## Three metrics, and the rotation artifact that had to be removed

Three per-residue quantities were computed. Root-mean-square fluctuation is taken from the fitted HS-AFM trajectories. Cross-conformer Cα standard deviation is taken from the 615 × 615 Kabsch-aligned library. Cα-Cα-Cα bond-angle standard deviation over the library measures backbone hinge activity, and unlike the first two it is invariant to rigid-body motion.

Raw RMSF is dominated by whole-molecule rotation. Aligning each frame to the first on 790 headpiece Cα atoms (αV residues 1 to 440 and β3 residues 1 to 350) before computing RMSF drops the mean from 53.47 to 19.29 Å in V1 and from 71.24 to 19.30 Å in V2. Before correction the two recordings differed by 18 Å in mean RMSF; after correction they agree to three significant figures, because the difference was a trajectory-length bias in the rotational component.

After correction the headpiece sits at 7.4 to 9.7 Å RMSF, consistent with side-chain and loop motion around a rigid body. The αV calf reaches about 20 Å, the β3 tail about 26 Å, and the αV C-terminal coil 44 to 47 Å.

![Three-metric mechanical sensitivity composite](/images/2026-09-02/mechanical_sensitivity_composite_v2.png)
*Rotation-corrected RMSF (top), cross-conformer Cα standard deviation (second), Cα-Cα-Cα angular standard deviation (third), and the rectified z-score product of all three (bottom), with the ten highest triple-agreement residues circled and the twelve Matsumoto 2008 switch residues marked by classification.*

The triple-agreement composite is the rectified product of the three z-scores, which requires a residue to score highly on all three rather than on one. Its highest values are B:689 at 10.47, B:652 at 10.14 and A:842 at 10.03. C-terminal coils dominate the ranking. Classical normal-mode analysis works from a static reference and does not emphasize this surface-coupled coil motion.

Bootstrap resampling with 500 replicates over the fitted frames tests whether the ranking is stable. The per-residue RMSF profiles of V1 and V2 correlate at Pearson r = 0.998. The 19 most flexible residues hold their position in at least 97 % of resamples. The boundary between the 20th and 21st residue is not resolved, because the 20th residue's lower confidence bound of 87.12 Å falls below the 21st's upper bound of 90.84 Å, so the top-19 is the defensible ranking and residues 20 to 25 are a second tier. Every top residue lies in αV C-terminal calf-2 and membrane-proximal regions, at residues 761 to 764, 804 to 805, 909 to 915, and 956 to 962.

## Agreement with published normal-mode analysis

Matsumoto and colleagues identified twelve switch residues in αVβ3 by elastic-network normal-mode analysis in 2008. Each was scored against the angular-variance map by percentile rank, with a ±10-residue neighborhood check.

![Matsumoto 2008 switch residues on the angular variance map](/images/2026-09-02/matsumoto_overlay.png)
*Cα-Cα-Cα angular standard deviation per residue for chain A (top) and chain B (bottom), with the twelve Matsumoto 2008 switch residues marked and colored by classification.*

Two are direct hits at or above the 95th percentile, three are near hits between the 80th and 95th, one is a neighborhood hit, and six are misses. The classification is not random with respect to residue role. All five residues that Matsumoto assigns a backbone-hinge function sit at or above the 80th percentile, including the primary snap residue β3 Arg633 at the 82.7th percentile and the snap-sandwich pair Cys374 and Leu375 at the 99.4th and 97.2nd. All six misses are Interaction-B partners or the α-constraint Ser305, which stabilize the structure through non-bonded contacts rather than by hinging. The two methods measure different things and agree where they overlap.

Separately, the β-knee at B:353 has the highest angular standard deviation in the library at 25.4°, against a pre-registered prediction of residue 352.

## Flexibility depends on state

Labeling each fitted frame by its hidden-Markov state allows the flexibility map to be recomputed per state.

![Per-state RMSF decomposition](/images/2026-09-02/rmsf_per_state_v1.png)
*Per-residue RMSF under bent-closed, intermediate and extended-closed state labels with 500-bootstrap bands (top), the per-residue extended-minus-bent difference (bottom left), the bent-versus-extended scatter (bottom right), and the ten most state-differential residues.*

Mean RMSF is 19.74 Å in bent-closed, 14.92 Å in intermediate and 12.41 Å in extended-closed. The bent state is 60 % more flexible than the extended-closed state. Of 1654 residues, 1640 have a bent-to-extended difference whose 95 % confidence interval excludes zero. The ranking is nonetheless preserved: any pair of states correlates at Pearson r ≥ 0.992 per residue, so the calf-2 hotspot is the most flexible group in all three states and only its magnitude changes. The ten most state-differential residues all lie in that hotspot, with a maximum change of −24.9 Å at A:764.

The same asymmetry appears in global shape. Radius of gyration rises from 53 to 70 Å between bent and extended-closed, asphericity from 2207 to 4337 Å², and relative anisotropy from 0.59 to 0.78, so the molecule moves substantially toward the rod limit. Eleven of twelve pairwise Kolmogorov-Smirnov tests are significant after Bonferroni correction. The bent-state distributions are consistently wider: 7.13 versus 2.80 Å in radius of gyration, 791 versus 324 Å² in asphericity, 0.180 versus 0.061 in anisotropy. The bent state is structurally heterogeneous and the extended state is structurally committed, which is consistent with the four-state model selection preferring to subdivide the bent band.

## What separates during extension

Ten domain-centroid pairs were tracked across the three states.

![Per-state domain-pair distances](/images/2026-09-02/domain_pair_distances_per_state_v1.png)
*(A) Per-state pairwise domain-centroid distances for ten αVβ3 domain pairs. (B) The same pairs ranked by extended-minus-bent change. (C, D) Distributions for the most and least separating pairs, with the full table below.*

The two largest separations are αV tail to β3 head at +43.5 Å (107.8 to 151.3 Å) and αV head to αV tail at +42.6 Å (129.9 to 172.5 Å). The smallest is αV head to β3 head at +3.3 Å (35.0 to 38.3 Å). The two feet stay within 9.5 Å of each other and the two legs within 5.6 Å.

Extension is head-leg separation. The headpiece travels as a unit and the legs travel with each other, which is the defining feature of the accepted structural model, recovered here from the fitted trajectories without assuming it.

At the residue level, contact maps over the 790 headpiece Cα atoms at an 8 Å cutoff give 9083 contacts in the bent state, 7750 in the intermediate and 6697 in extended-closed, a 26 % reduction. Taking a 30-percentage-point change in contact probability as the threshold, 93 contacts break and 29 form. The disrupted contacts cluster in the β3 βA specificity loop at B:162 to 167, the β3 α7-helix region at B:219 to 253, and αV β-propeller blades 2 to 5. The direct αV thigh-knee to β3 βA-PSI contact at A:400 to B:266 breaks with ΔC = −0.51. The contacts that form are intra-thigh pairs such as A:233 to A:289, consistent with the thigh-knee hinge opening.

Thresholding at a contact probability of 0.5 and treating the result as a graph changes the picture. Strong-contact edge count falls only 2.8 %, from 4468 to 4341, against the 26 % reduction at any probability, so most of the loss is in intermediate-probability contacts that weaken rather than disappear. Mean clustering coefficient is unchanged at 0.534 in all three states. Degree changes are asymmetric, with 275 residues losing contacts and 104 gaining, the losses concentrated in αV β-propeller blades 4 to 6 and the gains in the β3 PSI and I-EGF1 domains.

## Cryptic ligand-binding sites

Whether extension exposes a new druggable site was tested by three independent geometric methods on 20 bent and 20 extended library frames.

A solvent-accessible-surface differential over all 1654 residues finds 217 residues opening by more than 20 Å² and 103 closing by more than 20 Å². Restricting to intra-domain pockets, which excludes trivial rigid-body head-leg separation, leaves 9 candidates, and applying a druggability filter of at least 40 % hydrophobic content over at least 5 residues leaves exactly one: β3 K417, V419, G420, F421, K422, at the βA-hybrid-EGF1 hinge, with an aggregate change of +237 Å². The design resolves single-residue changes of 3.4 Å² and five-residue aggregates of 7.7 Å² at α = 0.05, well below the 100 Å² criterion, so a site opening by more than 100 Å² would have been found.

Two follow-up methods downgrade the candidate. A LIGSITE-equivalent pocket-volume calculation finds the region loses 3153 Å³ of pocket volume on extension (t = −44.7), against a flat-surface control losing 1644 Å³ and the MIDAS positive control gaining 471 Å³. A Vina-proxy ligand-fit score finds no significant change for the candidate (Δ = +0.12, t = 0.34), while both controls change significantly. The region is conformationally active but the change is from buried with discrete voids to exposed with bulk-solvent contiguity, which is not the formation of an enclosed binding site.

The canonical RGD site moves in the opposite direction. MIDAS solvent accessibility drops 35 % on extension, from 459 to 298 Å², even though the legs retreat 52 Å from the MIDAS centroid, and its Vina-proxy score falls from 3.07 to 1.47 as clashes rise from 1 to 5. In the library's extended frames the αV β-propeller and β3 βA domains rotate relative to each other and incidentally bury the pocket. These frames are extended-closed, and the headpiece-opening transition that exposes the high-affinity site does not occur in them.

## What remains

The flexibility ranking is derived from a library with no headpiece-open conformers and from fits against that library, so every per-state result above describes the bent-to-extended-closed axis only. Any ligand-accessibility question about the extended-open state is gated on generating those structures.

The angular-variance map is computed from the library, which is a steered trajectory rather than an equilibrium ensemble, so it measures which backbone angles the steering protocol was able to move. Cross-checking against an unbiased simulation would separate intrinsic hinge flexibility from protocol response.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
