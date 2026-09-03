---
layout: post
title: "Building an αVβ3 Conformer Library by Steered MD"
---

This project reconstructs the shape changes of single αVβ3 integrin molecules from high-speed AFM recordings, by matching each video frame against a library of candidate structures. Both halves of that method, rendering simulated AFM images from structures and matching real images back to them, need the same input: an ensemble of αVβ3 ectodomain structures spanning the bent and extended states. There is no extended αVβ3 crystal structure, so the ensemble has to be generated. This report covers how it was generated, what the resulting library covers, and the one region of conformational space it does not reach.

## The molecule, and the three distances used throughout

Integrin αVβ3 is a cell-surface receptor built from two protein chains, αV and β3, that run alongside each other along their whole length. Each chain has a globular head at one end and a long leg below it, and both legs pass through the cell membrane. The receptor works as a mechanical switch. Folded over at a knee roughly halfway down the legs, it is compact and binds its targets weakly; straightened out, it stands about 20 nm tall and binds them tightly. The portion studied here is the ectodomain, meaning everything outside the membrane.

Three shapes are named throughout. **Bent-closed** is the folded, weak-binding form. **Extended-closed** is standing up with the two heads still packed together. **Extended-open** is standing up with the heads separated, which is the strong-binding form. Shapes partway between bent-closed and extended-closed are called **intermediate**.

Three distances tell those shapes apart. Each is measured between the centres of mass of two domains, and each is abbreviated CV, for collective variable, the usual term for a small set of geometric quantities chosen to summarise a molecule of tens of thousands of atoms.

- **CV0, the αV head-to-calf distance.** Measured from the αV head down to the calf of its own leg. It is short when the molecule is folded over and long when it stands up, so it is the primary measure of extension.
- **CV1, the β3 head-to-tail distance.** The same measurement on the other chain.
- **CV2, the head-to-head separation.** Measured between the αV head and the β3 head. The two heads stay together whether the molecule is folded or standing, and move apart only when the headpiece opens, so this distance reports opening rather than extension.

The bent crystal structure of αVβ3, deposited as 1JV2, sits at CV0 = 51 Å. The library described below spans 47 to 85 Å.

## Structure prediction returns only the bent state

Two structure predictors, AlphaFold2 and Protenix, were tested for whether they produce anything other than the bent form. Each was asked to predict αVβ3 from sequence, and its prediction was compared against 19 frames of a pulling simulation running from the bent structure (frame 0) to an over-stretched one (frame 300). Two standard similarity measures were used: root-mean-square deviation between corresponding atoms after optimal superposition, in Å, and TM-score, which runs from 0 to 1 with 0.5 conventionally marking the boundary between the same fold and a different one.

Both predictors read a multiple sequence alignment, the set of evolutionarily related sequences that supplies most of their structural signal. Deliberately thinning that alignment makes some predictors emit alternative shapes, so Protenix was run twice, on the full alignment and on a 5 % subsample. The two runs differ by less than 0.001 in TM-score at every frame. AlphaFold2 produced 25 ranked models whose structures differ from each other by only 0.1 to 2.9 Å. Both predictors score 0.96 to 0.99 against the bent frame and 0.06 against frame 300, and their score profiles agree to within 0.03 everywhere in between.

![AF2 and Protenix TM-score and RMSD against pulled frames](/images/2026-09-02/af2_vs_protenix_validation.png)
*Best TM-score (top) and best Cα-RMSD (bottom) of AF2 and Protenix predictions against each pulled MD frame. The two curves are superimposed across the full range.*

Alignment subsampling produces no shape diversity here. Every deposited αVβ3 structure in the training data is bent, and that is apparently enough to fix both predictors on the bent form. Generating the extended structures by simulating the physics is therefore the only route left.

AlphaFold2 does carry usable information about which parts of the molecule move, in the form of pLDDT, its own per-residue confidence score on a 0 to 100 scale. Confidence is 96.8 in the head and upper leg but drops to 82.8 in the αV lower leg and 83.6 in the β3 lower leg. Those low-confidence regions are exactly the regions that move furthest when the molecule is pulled, so the predictor is uncertain about the parts that are genuinely mobile.

<p align="center">
  <img src="/images/2026-09-02/avb3_pulling_confidence.gif" alt="Pulled αVβ3 frames colored by AF2 per-residue confidence" width="480px" />
</p>
*Frames from a pulling simulation, colored by AlphaFold2's per-residue confidence in its bent prediction. The regions that move most are the regions the predictor is least confident about.*

## Only one way of pushing the molecule opens the leg

Steered molecular dynamics simulates the molecule atom by atom while adding an artificial force that pushes a chosen measurement toward a target value. The choice of what to push on is not obvious, so four schemes were tried, each simulating 1 ns of molecular time, each keeping the individual domains intact and moving only their arrangement.

The two angle schemes apply a torque across a hinge. The restrained-pull scheme pins some atoms in place and drags others. The distance scheme applies a flat-bottom restraint between two domain centres, meaning a force that is zero while the distance is within a tolerance of its target and grows the further outside it strays. In each case k is the stiffness of that restraint.

| preset | mechanism | Δ α-leg angle | Δ head-tail distance |
|---|---|---|---|
| gentle_open | angle torque, k = 50 | −4.5° | |
| moderate_open | angle torque, k = 200 | −0.2° | |
| restrained_pull | position restraints plus centroid pulling | numerically unstable (NaN) | |
| cv_distance_extend | flat-bottom distance bias, k = 200 | +88.2° | +112.7 Å |

![Comparison of four domain-steering presets](/images/2026-09-02/domain_steering_comparison.png)
*Hinge angles and domain-centre distances before and after each scheme, with the energy and temperature traces confirming the simulations were stable. Only the flat-bottom distance restraint (green) moves the geometry.*

The angle torques produce changes smaller than the molecule's own thermal jitter. Restrained pulling blows up numerically. The distance restraint opens the αV leg by 88° and extends the head-to-tail distance by 113 Å within 1 ns, and it is the method used for every library run below.

## Library coverage

The library is the union of two steering runs driven in opposite directions from the same starting structure, one pulling the molecule open and one folding it further shut. Saving a snapshot at regular intervals along each run turns the trajectories into a set of candidate structures.

The opening run contributes 309 structures covering CV0 from 52.9 to 85.0 Å. The closing run, added later, contributes 306 and reaches CV0 = 47.3 Å, which is more compact than the 51.4 Å of the bent crystal structure itself. The combined library is 615 structures.

![Steered MD CV trajectories and combined library manifold](/images/2026-09-02/steering_cv_trajectory.png)
*CV0 against time for the opening run (top left) and the closing run (top right), β3 leg extension for both runs (bottom left), and the combined 615-structure library plotted as αV leg extension against β3 leg extension, with the bent crystal 1JV2 marked (bottom right).*

The closing run overshoots the crystal structure rather than falling short of it, so the library brackets the known bent form rather than merely approaching it. On the extended side it stops at 85.0 Å, and the next section explains why it stops there.

## The headpiece does not open in an ordinary simulation

Standing up and opening the headpiece are two different motions, and the library above only performs the first. The second is measured by CV2, the head-to-head separation, which is about 36 Å when the heads are packed together and is taken to indicate an open headpiece above 50 Å. Three attempts were made to drive it.

The first attempt did not apply the force it claimed to. The scheme named for headpiece opening inherited the default list of domain pairs, which contains only head-to-leg pairs and no head-to-head pair, so nothing was pushing the heads apart. CV2 sat at 34 ± 0.4 Å for the whole run. After adding the head-to-head pair explicitly, two further runs were made. At a restraint stiffness of 250, CV2 moved from 34.6 to 34.9 Å. At a stiffness of 1000, with the leg restraints set to their current values so they could not compete, it moved from 35.7 to 36.6 Å.

![CV2 under headpiece-opening steering at k=1000](/images/2026-09-02/headopen_v5_cv2_failed.png)
*Head-to-head separation under the corrected headpiece-opening restraint. The trace is flat on the scale of the 60 Å target.*

The heads separate at 0.07 Å per picosecond, which extrapolates to roughly 320 ns of simulation to reach the target, against a budget of 3 ns. The interface holding the two heads together is too stable to be prised apart this way at any stiffness tested, and a stiffer restraint does not help because it distorts the protein before it separates the heads.

## No published structure supplies the missing shape

If the extended-open form cannot be simulated, the alternative is to take one that somebody has already solved experimentally. All five published crystal structures of the complete αVβ3 ectodomain were measured on the same three distances.

![Library coverage against five published αVβ3 ectodomain structures](/images/2026-09-02/library_coverage_v3.png)
*Library and fitted-trajectory distributions with the five published αVβ3 ectodomain crystal structures marked in red, for the αV head-to-calf distance (top), the β3 head-to-tail distance (middle) and the head-to-head separation (bottom). The hatched region in the bottom panel is the extended-open band.*

All five sit at CV0 = 51 to 52 Å and CV2 = 36 to 37 Å, which is to say all five are bent with their heads together. None comes near the extended-open threshold. Even the structure solved with the drug cilengitide bound, which shows the local signature of an opened head, is folded over as a whole molecule. The bent form is evidently the only one that crystallises, so importing an extended-open structure from the databank returns nothing usable.

## What remains

The library covers the bent-to-extended-closed axis and does not cover extended-open. Two consequences follow for everything built on it. Any state assignment made from this library can label bent-closed, intermediate and extended-closed, but cannot label extended-open. Any free-energy or population estimate is undefined above CV0 = 85 Å.

Closing the gap needs a method that deliberately drives a simulation over energy barriers, rather than a longer ordinary run. Candidates are metadynamics, replica exchange, a string method seeded from structures of the related integrin αIIbβ3, or a simplified model in which groups of atoms are treated as single particles. A string-method calculation along the bent-to-extended path, described in an [earlier post](/2026/08/26/conformers-md.html), is the current line of attack.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
