---
layout: post
title: "A Forward Model for Simulated High-Speed AFM"
---

<p align="center">
  <img src="/images/2026-09-02/sim_afm_video2.gif" alt="Simulated HS-AFM video of αVβ3" width="260px" />
</p>
*Simulated high-speed AFM video of αVβ3, generated from a library of 615 structures produced by simulation. This is an earlier version of the renderer; two of the corrections described below were made after it.*


High-speed atomic force microscopy images a single molecule by dragging a sharp tip across it and recording the height at every point, fast enough to produce a movie rather than a still. What it returns is not a structure. It is a low-resolution height map, blurred by the finite width of the tip, at roughly one nanometre of vertical noise. Recovering a structure from it means going the other way: taking a candidate structure, predicting the image the microscope would have produced, and comparing that to what it actually produced.

This project reconstructs the shape changes of single αVβ3 integrin molecules by matching each video frame against a library of candidate structures, described in a [companion post](/2026/09/02/a-conformer-library-steered-md.html). The forward half of that method, covered here, turns the library into a simulated microscope movie. It exists to test the imaging model on its own: if the same library and the same tip settings that fit a real recording also reproduce how that recording looks, the imaging model is not the thing limiting the result. This report covers the renderer, the settings that were calibrated against real data, and the measured agreement between simulated and real frames.

## How an image is generated from a structure

The renderer takes a folder of structures plus a metadata file recording each one's dimensions and its position along the trajectory, which is the same input the template-matching step consumes, and runs four stages.

1. Load the library of structures in trajectory order.
2. Fix each structure's orientation on the surface, so the simulated molecule sits and turns the way a real one does.
3. Render the microscope image. The tip is modelled as a sphere rolled over the surface of the molecule, which is what makes features in the image wider than the features that produced them, and vertical blur and noise are added to match the microscope's limits.
4. Apply the visual character of a real scan on a single canvas: substrate texture, blur, surface slant, line-to-line jitter, streaks, and the copper colour scheme these instruments are usually displayed in.

## Making the molecule sit on the surface the way a real one does

A molecule stuck to a surface lies on one consistent face and turns in occasional discrete jumps, because it has to break its grip on the surface to move. Rendering each frame straight from fitted coordinates does neither: the simulated molecule rolls over onto other faces and spins smoothly, which no real specimen does.

Five constraints are applied in sequence. The molecule's thinnest direction is turned vertical, so it lies flat rather than standing on edge. The αV face is kept downward, so it does not roll over. Its long axis is sign-matched between consecutive frames, which removes an artificial 180° flip that arises from how the axis is computed rather than from any motion. Rotation about the vertical is then quantised: the molecule holds its heading until the underlying fit has disagreed by more than 50° for at least 20 frames, then turns by at most 30° at once. Finally the head position is smoothed over an 8-frame window. Across the 1266 frames of the longer recording this produces three discrete turns.

## Four corrections made against the real recordings

Three corrections were made by comparison against the real recordings.

The molecule was running off the edge of the image. The original 35-pixel canvas covers 34 nm at just under 1 nm per pixel, and the stabilized coordinates reach an xy span of 26 nm at full extension, so extended frames were clipping at the canvas edge. The canvas was enlarged to 60 px, covering 58.8 nm.

A rectangular seam was visible in the image. The stylizer had been generating the background texture on a large canvas and pasting the rendered molecule into a smaller panel inside it, which left a visible border wherever the molecule reached past the edge of that panel. Every stylization step now runs on a single canvas at one resolution.

The height axis was unrealistically crisp. The renderer was producing a precise height map, where a real instrument averages height over each oscillation of its vibrating tip and carries 0.3 to 0.5 nm of vertical noise. Blurring the simulated height map by 0.8 nm and adding 0.35 nm of noise brings the simulated height axis into the same range, and makes the molecule read as a blob rather than a survey map.

A fourth correction removed a horizontal squash. The final blur had been made stronger across the image than down it, which was a reasonable setting back when the molecule could lie at any angle and the blur therefore hit it evenly on average. Once the orientation lock put the long axis consistently left-to-right, the same setting compressed the molecule along its own length. The blur is now equal in both directions.

![Anisotropic versus isotropic post-dilation blur](/images/2026-09-02/sim_afm_v16_isotropic_blur.png)
*Four frames of the simulated video rendered with unequal blur across and down the image (top row) and with equal blur in both directions (bottom row). The unequal setting compresses the molecule along its locked long axis.*

## How closely the simulated frames match the real ones

The structures fitted to the real recordings, described in a [companion post](/2026/09/02/c-hs-afm-template-matching.html), were rendered with a 1.5 nm tip at just under 1 nm per pixel, then compared frame by frame against the real recordings. Agreement is measured as the correlation between the simulated and real images, with 1 meaning identical. The random baseline is the same measurement made against a randomly chosen frame, which is well above zero because any two images of the same small bright object on the same dark background already resemble each other.

| | simulated vs real | random baseline |
|---|---|---|
| recording 1 | 0.824 | 0.65 |
| recording 2 | 0.722 | 0.43 |

The library and the imaging model together reproduce the real data above the random baseline in both recordings, with a larger margin in recording 2.

The tip radius is the single most consequential setting, because it sets how much wider a feature appears than it really is. Rendering the same structure with a 1, 2 and 5 nm tip gives correlations against the real recording of 0.648, 0.724 and 0.748.

![Simulated frames at three tip radii](/images/2026-09-02/sim_afm_tipsweep.png)
*The same structure rendered with a 1, 2 and 5 nm tip, with the mean correlation against the real recording for each. The blunter tip reproduces the real scan more closely.*

The blunter tip looks more like the real recording, but the sharper tip is the one used for template matching, because a blunt tip smears different structures into images that look alike and so cannot tell them apart. The two criteria pull in opposite directions, which is a sign that treating the tip as a hard sphere is an approximation rather than a description, and it bounds how literally the rendered images can be read.

## How much the tip squashes the molecule

Treating the tip as a hard sphere assumes the molecule does not give way under it. A real tip presses into a soft protein, so the recorded height is lower than the true height. The size of that error was estimated with Hertz contact theory, the standard elastic model for a sphere pressed onto a surface, over the range of forces these instruments image at.

| applied force | indentation | contact radius | peak pressure |
|---|---|---|---|
| 50 pN | 0.111 nm | 0.309 nm | 250 MPa |
| 100 pN | 0.177 nm | 0.389 nm | 315 MPa |
| 200 pN | 0.281 nm | 0.490 nm | 397 MPa |

The tip sinks in by 5 to 14 % of the height of the feature it is measuring. Every one of those values is smaller than the roughly 1 nm of vertical noise the instrument already carries, so the squashing is hidden underneath the noise and the hard-sphere model is defensible at this level. The error is a known offset of a known size, not an unquantified confounder.

## A mismatch between the two halves of the method

The fitted coordinates exist in two versions, one with the orientation lock applied and one without. The renderer read the locked version while the figure that draws fitted structures over the real frames defaulted to the unlocked one. The two differ in heading by 66° at frame 100, which appeared in the combined figure as a rotational misalignment and looked like a scientific finding rather than a mismatched file. Both paths now read the same version.

## What remains

Three items are open. The agreement between simulated and real frames tops out at 0.72 to 0.82, and it is not yet known how much of the shortfall is the imaging model and how much is the library missing the structures the molecule was actually in. The indentation correction is currently bounded but not applied, and should become a setting on the renderer. All calibration so far uses one instrument and two recordings, so the tip and noise settings are fitted to a single source and may not transfer.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
