---
layout: post
title: "Four Ways an MD Result Can Be a Believable Wrong Answer"
---

<p align="center">
    <img src="/images/2026-08-26/avb3_extension.gif" alt="Bent to extended αVβ3 leg swing" width="420px" />
</p>

The other thing occupying me lately is molecular dynamics on integrin conformers — αVβ3 specifically, and the bent↔extended switch specifically. Integrins are force-sensitive machines. That fact turns out to be the entire story, and I spent a few months learning it the expensive way.

## Building an endpoint that doesn't exist

State A is easy: 1JV2, the bent crystal. State B is the problem — there is no extended αVβ3 crystal structure. So it has to be built, by swinging the lower legs about the genu (knee) hinge until they point away from the head instead of folding back against it.

The one-shot version — rotate the legs 139° in a single rigid-body move, then minimize — left a knee clash at around 10¹³ kJ/mol. No minimizer descends that. A single enormous clash gives L-BFGS no foothold at all; a CPU worker sat at 715% for four minutes without making progress.

The fix was to stop being clever: rotate about 9° at a time and minimize after each step, in vacuum so each minimize is cheap. Sixteen steps, 144° total, and the potential energy stays around 10⁴ kJ/mol the whole way instead of 10¹³.

![Incremental leg swing: extension and energy per step](/images/2026-08-26/route_a_morph_descent.png)

Radius of gyration goes 39 → 67 Å, long-axis extent 130 → 211 Å — about 21 nm, which is what an activated integrin ectodomain should measure — and a short dynamics settle doesn't spring it back to bent. The intermediate frames are a bonus: they're a ready-made initial path for a string method.

Then things got interesting, in the way that means "wrong."

## 1. I measured a barrier that wasn't there

The plan was clean. Find the residues that lock the extended state open, mutate them, run MD, watch the knee fall. A contact differential between bent and extended pointed at a cross-knee salt-bridge network at the genu, and I mutated the two best candidates.

The knee fell. It also fell for wild type.

My first read was "underpowered — need twenty replicates per arm." That was wrong, and it's the useful kind of wrong. Integrins are force-sensitive, so bent is the ground state at zero force and extension is thermodynamically *uphill* without tension. Roughly 20 pN over 15 nm tilts the landscape by ~43 kcal/mol; a single genu salt bridge screens to maybe 6–8. I hadn't measured an under-sampled barrier. I'd measured a system that has no barrier in that ensemble. More replicates would only have pinned down the collapse rate more precisely.

## 2. The observable itself was lying

The residue ranges that define the knee angle were keyed to 1JV2 numbering and then applied to a PDBFixer-renumbered file. On chain B those ranges selected a residue **61 Å away from the actual αV knee**, at fraction 0.84 along the head→foot axis instead of the real hinge at 0.44.

A true 30° bend registered as 15.3°. Compressed 1.8×, with variance 3.3× too small — a stiffness estimate off that would have come out three times too stiff, and a 20°-bent structure would have scored as extended. Nothing raised an exception, because the ranges resolved to real residues; they were just the wrong ones. The check is now an assertion on the exact CA counts, which is the sort of thing you only write after it bites you.

## 3. The protocol would have produced a believable wrong answer

The natural experiment is to ramp force from 0 to 60 pN and read off the force at which the thing gives way. Except this run *starts* extended, and unloaded, the knee loses half its drop in 3.8 ± 2.4 ns on its own.

Map that zero-force collapse clock onto the ramp schedule and it reports **F½ = 12 ± 7 pN with no force applied at all** — which lands right on the literature figure for integrin extension. A null model that reproduces the answer you were hoping for is about the worst thing you can find, and this one cost zero GPU-hours to compute.

So the protocol got flipped. Equilibrate *under* the holding load, then ramp **down** from 60 pN to 0, and measure the force at which holding stops. That number has to beat 12 ± 7 pN to mean anything.

## 4. The lock might be my own fingerprints

The four-pair genu network that everything above rests on appears in the morph product — and in neither the bent crystal nor the pre-relaxation seed. Two facts about the model explain why, and I checked both locally rather than taking them on trust:

- **There is no calcium in it.** 1JV2 carries six structural Ca²⁺, one of them at the genu, and one of my four lock residues is a first-shell coordinating residue for it at 2.09 Å. The morph dropped the ions. Xiong's 2001 paper explicitly predicted that this ion neutralizes the thigh/calf-1 acidic interface in an extended integrin — so lysines being recruited to a bare acidic patch is the *predicted consequence of my own deletion*.
- **Zero ionic screening.** The implicit-solvent build carries κ = 0 verbatim, i.e. 0 mM salt, with reaction-field dielectric 1.0. Every salt bridge in the model is unscreened, which is an upper bound on lock strength and nothing more.

Two literature sweeps came back with the verdict "narrowly novel, not yet defensible." Correct.

## What actually survives

Two things, and I want to be precise about which.

**The force is real, verified twice.** Requesting 20 pN puts −20.000 pN on the head anchor and +20.000 pN on the foot, net 2×10⁻¹³ pN; 400 pN in vacuum visibly straightens the knee while the 0 pN control contracts. After four prior silent-null bugs in this project, "prove the force reaches the atoms" is a standing precondition on every force run, not a debug flag.

**The readout is the bridge distance, not the knee angle.** Under a 60 → 0 pN ramp, wild type kept **0 of 12 genu bridges open across three runs**. At zero force, matched protocol, same script, they came apart every time. K459A ruptured 4 of 6; E598A, 9 of 9. Fisher's exact on WT loaded versus unloaded is p = 0.029. That's suggestive at n = 3 runs and not a word more — but it's the first force-attributable observation this project has produced, and bridge distances are direct atom-pair measurements, so the hinge-numbering bug above can't touch them.

![Knee angle, extension probability and stiffness versus applied load](/images/2026-08-26/route_a_force_ramp.png)
*The left two panels are the ones you can't quote — see below.*

Inside a single ramp, force and time are perfectly confounded, so anything that merely decays produces the F½ signature you were hoping for. The null model already reproduces the ordering across all three genotypes. The stiffness panel is empty on purpose: under a ramp, the variance in knee angle is the drift the ramp is driving, not thermal fluctuation, so the analysis reports `null` rather than printing a meaningless number.

One more piece of geometry, if you ever pull on a hinge: the axial pull couples to the knee as cos(θ/2), which vanishes at full extension. Measured from my own coordinates — genu→head arm 67.7 Å, genu→foot 85.6 Å — delivering the knee torque at 174° that 17.8 pN delivers at 146° takes about **108 pN**. That's why 60 pN held the salt bridges but never straightened the knee, and it's a second, independent reason the bridges are the better observable.

## Next

Cheap and decisive, in that order: restore the genu Ca²⁺ from the crystal, set κ for 150 mM salt, re-run wild type loaded and unloaded. About 13 GPU-hours on a shared A5000. That decides whether any of the above is a result or an artifact of a model I built myself — and it's worth more than another twenty replicates of the wrong ensemble.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
