---
layout: post
title: "One Genu-Hinge Morph Recipe Across Five Integrins"
---

<p align="center">
  <img src="/images/2026-09-02/multi_integrin_morph.gif" alt="Bent to extended morph for five integrin heterodimers" width="640px" />
</p>
*Incremental genu-hinge morph applied to five integrin heterodimers from their deposited bent or half-bent structures. Blue is the headpiece, aqua the upper leg, orange the lower legs.*

Integrins are cell-surface receptors built from two chains running alongside each other, each with a head at the top and a long leg below, folded at a knee roughly halfway down. They switch between a compact folded form and an upright one. Structures of the folded form have been solved for several members of the family; structures of the upright form largely have not, so the upright ones have to be built.

[Earlier work in this project](/2026/08/26/conformers-md.html) built one, for αVβ3, by rotating the lower legs about the knee. Doing that in one move drives atoms through each other at the knee, producing an energy so large that no minimisation routine can recover from it. Turning the legs a few degrees at a time and relaxing the structure after each step avoids this, because no single step creates a collision too severe to repair. That recipe was written for one receptor, with the boundary between the parts that turn and the parts that stay hand-checked. This report covers generalising it to five integrins with those boundaries transferred automatically, validating it against the hand-built result and against an experimental structure, and the two cases where the transfer is not trustworthy.

## How the models are built

The procedure needs to know, for each receptor, where the knee is and which residues belong to the parts that swing. Those were obtained by structurally superimposing each target on αVβ3 and carrying the αVβ3 definitions across the superposition, so no target needs manual annotation. Each run then swings the lower legs about the knee in steps of roughly 9° and relaxes the structure after every step, stopping once the knee angle passes 150°, which is straight enough to count as upright. Relaxation is done without surrounding water, which is what makes the whole thing cheap: every run is 35 to 45 minutes on ordinary processors, with no GPU.

A run is accepted only if all four of the following hold. The knee angle reaches at least 150°. It increases throughout, without backtracking by more than 5°, so the structure is not being wrenched back and forth. The molecule's overall length grows by at least a fifth. And the energy stays finite at every step, meaning no unrecoverable collision was created along the way.

## The five receptors

| receptor | starting structure | boundary transfer | knee angle, folded to upright | length, folded to upright | overall size, folded to upright | steps |
|---|---|---|---|---|---|---|
| αVβ3 | 1JV2 | high | 41.2° to 176.2° | 130.0 to 209.9 Å | 39.1 to 66.6 Å | 16 |
| αIIbβ3 | 3FCS | high | 36.3° to 179.4° | 118.5 to 207.3 Å | 39.0 to 63.5 Å | 17 |
| α5β1 | 7NXD | high | 78.0° to 178.2° | 160.2 to 214.7 Å | 48.7 to 63.3 Å | 12 |
| αMβ2 | 7USM | low | 48.2° to 172.3° | 109.0 to 207.9 Å | 39.8 to 61.4 Å | 15 |
| αXβ2 | 5ES4 | low | 12.9° to 175.5° | 137.3 to 238.6 Å | 54.8 to 80.3 Å | 20 |

All five pass all four acceptance conditions.

![Bent to extended morph across five integrin variants](/images/2026-09-02/variant_extension_summary.png)
*(A) Knee angle against total leg rotation for each receptor, with the 150° acceptance threshold marked. (B) Energy against total leg rotation. (C) Overall length from the folded starting structure to the built upright one, with experimentally determined upright structures marked for αVβ3.*

Panel A shows the knee opening steadily in every case, the curves offset from each other only by how folded each starting structure was. Panel B is the check that matters for the method: the energy stays low throughout, apart from two spikes in the αIIbβ3 run where a step created a collision. Those spikes are the failure mode the incremental approach exists to survive, and the next relaxation step removes them rather than carrying them forward, which is visible in the curve returning to baseline instead of stepping up.

The five starting structures are not equally folded, and the procedure absorbs that by taking however many steps each needs. α5β1 begins at a knee angle of 78°, already half open, and needs 108° of rotation over 12 steps. αXβ2 begins at 12.9°, folded almost double, and needs 180° over 20 steps.

That α5β1 starts half open is worth noting on its own, because it was determined here by measurement rather than taken from the structure's description. Three independent geometric measures agree: it buries 10,555 Å² of surface between head and leg against 13,442 for αVβ3 and 15,298 for αIIbβ3, its head sits 73.9 Å from its foot against 42.6 and 37.2, and it is larger overall. The depositors described that structure as half-bent, and the geometry supports the description rather than merely repeating it.

## Checks against a known answer and an experimental structure

The αVβ3 run is included as a control, because its answer is already known: the same molecule was built once before with hand-checked boundaries. The automated procedure gives an overall size of 66.57 Å against 67.32 Å and a length of 209.9 Å against 211.36 Å, differing by 1.1 % and 0.7 %. Transferring the boundaries automatically reproduces the hand-checked result, which is the claim the other four runs rest on.

αVβ3 is also the only one of the five with an experimentally determined upright structure to compare against. Measured against that structure, solved by cryo-electron microscopy, the built model is 209.9 Å long against 214.5 Å, a difference of 4.6 Å on a 21 nm molecule, and 66.6 Å in overall size against 62.4 Å.

The knee angles agree far less well: 176.2° for the built model against 145.3° for the experimental one. The built model is straighter than a real upright integrin. This follows from the procedure, which keeps swinging until the acceptance threshold is passed rather than aiming at a particular angle, and from relaxing without water or ions, which removes the interactions that would hold a real molecule at an intermediate angle. The models are the right length and too straight.

## What remains

Three limitations bound these endpoints.

The procedure relaxes in vacuum, with no water, no ions and no thermal motion. What it produces is a structure free of collisions and suitable as a starting point for a proper simulation, not a structure the molecule would actually be found in. Only the αVβ3 model has been carried through that next step.

The automatic boundary transfer is only as good as the structural superposition it rides on, and that quality is measurable. It scores 1357, 663 and 353 for αVβ3, αIIbβ3 and α5β1, and only 137 and 146 for the two β2 receptors, an order of magnitude lower. The reason is structural: the β2 receptors carry an extra inserted domain in the head that the αV reference does not have, so the superposition has nothing to match it against and the leg boundaries downstream of it are less certain. Their runs do pass all four acceptance conditions, but those conditions test whether the procedure worked, not whether it was applied to the right residues. Those two models need their boundaries checked by hand before their numbers are used.

Only αVβ3 has an experimental upright structure to check against, so the 4.6 Å agreement in length is a single data point, and the 31° disagreement in knee angle in the same comparison says the built models are systematically too straight. Neither is established for the other four.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
