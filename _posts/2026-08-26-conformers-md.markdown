---
layout: post
title: "Force-Biased MD on the αVβ3 Genu"
---

<p align="center">
    <img src="/images/2026-08-26/avb3_extension.gif" alt="Bent to extended αVβ3 leg swing" width="420px" />
</p>

Notes on where the integrin conformer work stands. The target is the bent↔extended switch in αVβ3 — what holds the extended ectodomain open, and whether removing a specific contact changes how it behaves under load.

## Building the extended endpoint

State A is 1JV2, the bent crystal. There is no extended αVβ3 crystal structure, so state B has to be constructed by swinging the lower legs (αV calf-1/2, β3 I-EGF/β-tail) about the genu hinge until they point away from the head rather than folding back against it.

A one-shot 139° rigid-body rotation leaves a knee clash at ~10¹³ kJ/mol that no minimizer descends — a single clash that large gives L-BFGS no line-search foothold, constrained or not. Swinging incrementally instead — ~9° per step with a vacuum minimize after each — keeps the potential energy near 10⁴ kJ/mol at every step. Sixteen steps, 144° total, CPU-only.

![Incremental leg swing: extension and energy per step](/images/2026-08-26/route_a_morph_descent.png)

| | bent | extended |
|---|---|---|
| radius of gyration | 39.1 Å | 67.3 Å |
| long-axis extent | 130.0 Å | 211.4 Å |
| β head↔tail | 28.4 Å | 106.6 Å |

The 211 Å extent is ~21 nm, matching the height of an activated integrin ectodomain rather than an arbitrary elongation. A 10 ps dynamics settle holds it (extent 211 → 208 Å, Rg 67 → 66 Å), so the pose does not spring back to bent. The sixteen intermediate frames are also a usable initial path: the adjacent-image Cα-RMSD chain is 55.6 Å over 16 segments with a segment-length coefficient of variation of 0.23, near-uniform enough to reparametrize onto 12 equally spaced nodes for a string method.

Segment lengths shrink from 4.9 Å on the bent side to 2.3 Å on the extended side, which is the expected lever-arm effect: a fixed 9° swing sweeps more Cα displacement when the legs are folded than when they are nearly straight.

One caveat on the reaction coordinate. Radius of gyration, long-axis extent and head↔foot distance are computed over all Cα and are independent of residue numbering. The genu hinge angle is not — it depends on explicit residue ranges, and those ranges were keyed to 1JV2 numbering while state B is renumbered contiguously by PDBFixer. On chain B that selected a residue 61 Å from the αV knee, compressing a true 30° bend to 15.3°. The angle-based coordinate is being re-derived; the shape metrics above are unaffected.

## The genu lock

A heavy-atom contact differential between the two states across all 1466 residues (6749 contacts bent, 6940 extended, 1252 extended-unique) puts the extension-specific contacts at the genu: a cross-knee network between the αV thigh (K459, D457) and calf-1 (E598, D595) plus K688 and E636, with charged partners 19–25 Å apart in bent and ~2.6–2.9 Å in extended.

Two follow-ups characterize it without dynamics:

- **The locks engage sequentially**, not together. Taking each bridge's engagement point as the knee angle at which it first closes below 4 Å along the morph: D595–K688 at ~50°, E598–K459 at ~84°, K459–E636 at ~159°.
- **K459 is the energetic hub.** Direct ff14SB nonbonded interaction energy per residue pair, combined with the knockout matrix for each mutation, gives ~179 kcal/mol of lock energy removed by K459A (two bridges) against ~97 for E598A (one). Screened at ε = 4r these fall to the 5.7–8.1 kcal/mol range expected for buried salt bridges.

## Zero force is the wrong ensemble

Mutating a linchpin and watching the extended state decay at zero force does not test the lock. Integrins are force-sensitive: bent is the ground state at F = 0, and extension is thermodynamically uphill without tension. Roughly 20 pN over 15 nm tilts the landscape by ~43 kcal/mol, against ~6–8 kcal/mol screened per genu lock. In that regime wild type collapses alongside every mutant — knee 174° → 146°, mutant−WT differential ~9° against ±10–17° replicate scatter (Welch t = 1.1 and 0.85). That is an absent barrier, not an under-sampled one, and more replicates only measure the collapse more precisely.

## Applying force, and verifying it arrives

Load is applied as a `CustomCentroidBondForce` between the ligand-binding headpiece (736 Cα) and the C-terminal 30 residues of each chain (60 Cα), where the TM helices would continue — a separation-independent force, equal and opposite, along a 153 Å axis.

Two checks run on every force run:

- **Readback.** Requesting 20 pN puts −20.000 pN on the head anchor and +20.000 pN on the foot, net 2×10⁻¹³ pN; 40 pN gives −40.000/+40.000, on the real 22,483-atom build.
- **Displacement.** 20 ps at 400 pN in vacuum takes the knee 174.4° → 176.9° and holds head↔foot at 152.5 Å, while the 0 pN control falls to 172.2° and contracts to 151.5 Å.

The protocol is a **down**-ramp: equilibrate 2 ns at 60 pN, then release 60 → 0 pN over 8 ns, so the measurement is the force at which holding stops. An up-ramp from zero is not interpretable from this starting structure — the unloaded knee loses half its drop in 3.8 ± 2.4 ns on its own, and mapping that collapse clock onto a 0 → 60 pN schedule reports F½ = 12 ± 7 pN with no force applied at all. That number is the null any result has to beat.

## Result under load

Three genotypes × 3 replicates, plus a matched no-load control arm; 12 runs, ~26 GPU-hours on a shared A5000.

| genu bridges opened, first 10 ns | WT | K459A | E598A |
|---|---|---|---|
| under 60 → 0 pN load | 0 of 12 | 4 of 6 | 9 of 9 |
| at F = 0, matched protocol | all | all | all |

Load keeps the wild-type genu network clasped over a window in which, unloaded, it always comes apart; neither single mutant is rescued by the same load. Fisher's exact on WT loaded versus unloaded is p = 0.029, and at the run level this is n = 3 — suggestive, not significant. Bridge distances are direct atom-pair measurements, so this readout is independent of the residue-range problem above, and the F = 0 control carries identical columns.

![Knee angle, extension probability and stiffness versus applied load](/images/2026-08-26/route_a_force_ramp.png)

The knee-angle F½ values in the left panels do not beat their null model and are not quotable: within a single ramp, force and time are perfectly confounded, so anything that merely decays reproduces the signature. The null predicts F½ = 42.8 ± 15.1 pN (WT), 60.0 ± 0.0 (K459A), 59.8 ± 0.4 (E598A) — the same ordering as observed. The stiffness panel is empty by construction; under a ramp the variance in knee angle is ramp drift, not thermal fluctuation, so κ requires a constant-force ladder instead.

The static ranking does not survive either. E598A collapsed further than K459A (final knee 110.3° vs 118.9°, three bridges opened vs two), inverting the lock-energy prediction for the second time — under load and without it.

## Two properties of the model that bound the reading

**The pull couples to the knee as cos(θ/2), which vanishes at full extension.** From the coordinates: genu→head arm 67.7 Å, genu→foot 85.6 Å, head→foot 153.1 Å; cos(θ/2) = 0.049 at 174.4° against 0.296 at 145.6°. Delivering the knee torque at 174° that 17.8 pN delivers at 146° takes ~108 pN of axial force. Force-versus-knee is intrinsically weak at the extended end, which is a second reason the bridge distances are the better observable.

**The lock network is partly an artifact of how state B was built.** The four pairs are at 2.6–2.9 Å only in the morph product; they are not formed in 1JV2 (23.0 / 19.4 / 25.4 / 7.4 Å) nor in the pre-relaxation seed. Two reasons, both verified against the files:

- **No calcium.** 1JV2 carries six structural Ca²⁺, one at the genu, first-shell coordinated by E636 OE1 at 2.09 Å with D595 and E598 in its second shell. The morph dropped them. Xiong (2001) predicted that this ion neutralizes the thigh/calf-1 acidic interface in an extended integrin, so lysine recruitment there is a predicted consequence of the deletion.
- **No ionic screening.** The GB-OBC2 build carries κ = 0, i.e. 0 mM salt, with reaction-field dielectric 1.0. Every salt bridge is unscreened, an upper bound on lock strength.

This does not touch the WT-versus-mutant differential, since all arms share the same model, but it bounds how literally the rupture forces can be read.

## Next

Restore the genu Ca²⁺ from 1JV2, set κ for 150 mM salt, and re-run n = 3 loaded and n = 3 unloaded wild type — ~13 GPU-hours. Two zero-cost checks come first: measure the four pairs in PDB 6DJP, a deposited extended αV leg, and score them on published explicit-solvent force-clamp trajectories that include metals and salt.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
