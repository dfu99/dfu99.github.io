---
layout: post
title: "Free Energy and Kinetics of Surface-Bound αVβ3 from HS-AFM"
---

Fitting each frame of two high-speed AFM recordings against a library of candidate structures, described in a [companion post](/2026/09/02/c-hs-afm-template-matching.html), gives 1645 measurements of the shape of a single αVβ3 integrin molecule at one per second: 379 from recording 1 and 1266 from recording 2.

The value of those frames is that nothing was pushing the molecule. A simulation that drives a molecule from one shape to another tells you the shapes exist but not how often they occur, because the driving force sets the answer. Here the molecule was simply watched. How often it was seen in each shape is therefore a measurement of how often it adopts that shape, which is what makes the rest of this report possible: how much energy separates the shapes, how often it moves between them, and how long it stays.

The shape of each frame is summarised by one number, the distance from the αV head down to the calf of its own leg, written CV0. It is short when the molecule is folded over and long when it stands up. Four bands are used: **bent-closed** below 65 Å, **intermediate** from 65 to 78 Å, **extended-closed** from 78 to 85 Å, and **extended-open** above 85 Å. The first three are increasingly upright forms and the fourth is the fully activated one, in which the two head domains also separate.

![Four-panel dynamics synthesis](/images/2026-09-02/dynamics_synthesis_v1.png)
*(A) The free-energy profile against extension, with its 95 % confidence band, the four state bands, and the lower bound on the extended-open state. (B) The population of each state under three independent definitions. (C) The state the molecule occupies over the course of recording 2, with independently detected abrupt transitions marked and the table of hop probabilities inset. (D) The fraction of visits to each state still ongoing after a given time, with fitted exponentials.*

## Which shapes cost energy

A shape the molecule is seen in often is a shape it costs little energy to adopt, and the relationship between the two is fixed: energy is proportional to the negative logarithm of the frequency. Counting how often each extension occurs across all 1645 frames, smoothing the histogram, and applying that relation at body temperature converts the observed distribution into a free-energy profile. Energies are quoted in kcal/mol; for scale, about 0.6 kcal/mol is the ambient thermal energy at this temperature, so a 4 kcal/mol barrier is roughly seven times what thermal jostling supplies at any instant. Panel A shows the result, with zero set at the lowest point.

The lowest point sits at 70 to 75 Å, in the intermediate band, and not at either of the two shapes the textbook picture names. Climbing out toward either the bent or the extended-closed side costs about 4 to 5 kcal/mol. The molecule on this surface is therefore not switching between two stable forms with a transition state in between; its preferred shape is the one that picture treats as the transition.

Above 85 Å the profile is not merely uncertain, it is undefined, because the library contains no structures there and the method cannot return a match it was never given. The figure hatches that region rather than drawing a line through it.

A one-sided statement is still available. Of 1645 frames, 42 were fitted at or above 85 Å. Treating that as a counting problem gives an upper bound on how often the extended-open form can occur of 3.40 %, and converting a population ceiling into an energy floor by the same relation used above puts the extended-open state at least 2.02 kcal/mol above the intermediate. That is a floor, not an estimate: the true value could be far higher, but it cannot be lower. Any simulation aiming at that state has to pay at least this much.

## How much time is spent in each shape

How much time the molecule spends in each shape depends on where the boundaries between shapes are drawn, so it was computed three ways: from the free-energy profile, by simply counting frames in each band, and from the state model described in the next section, which infers boundaries from the data instead of imposing them. Panel B compares the three, which agree except in the bent band.

| state | Boltzmann, from ΔG | empirical count | HMM stationary |
|---|---|---|---|
| bent-closed | 25 % | 26 % | 40 % |
| intermediate | 46 % | 45 % | 35 % |
| extended-closed | 24 % | 26 % | 24 % |
| extended-open | 5 % | 3 % | not represented |

The state model puts more weight on the bent state because the bent state it infers is broad, spanning about 9 Å, and so claims frames in the 50 to 70 Å region that the fixed bands assign to the intermediate. The two are answering different questions. The fixed bands answer how much time the molecule spends within a given range of extensions; the state model answers which state it is kinetically in, including frames caught partway. Neither is wrong, and the difference is worth stating rather than averaging away.

## How the molecule moves between shapes

The trajectory was then fitted with a hidden Markov model, which assumes the molecule is always in one of a small number of discrete states, that each state produces a characteristic range of measured extensions, and that it hops between them at fixed probabilities. Fitting it to the data returns all three: where the states sit, how likely a hop is, and which state the molecule was in at each moment. Nothing about the states is specified in advance.

A three-state fit places them at 61.0, 74.4 and 82.4 Å, matching bent, intermediate and extended-closed. Each state is sticky: in any given second the molecule stays where it is with probability 0.92, 0.86 and 0.90.

The informative entries are the ones that are nearly zero. Hops directly between bent and extended-closed happen at under 1 % per second in either direction, so the molecule essentially never goes from folded to upright without stopping in between. The intermediate is not just the shape the molecule prefers, it is the only route between the other two, which the energy profile alone could not have shown: a profile says where the valleys are, not whether the molecule must pass through one to reach another. Of the departures from the intermediate, 59.7 % return to bent and 40.3 % continue to extended-closed.

A visit to a state lasts, on average, 14.6 ± 4.3 s in bent, 7.8 ± 1.8 s in intermediate and 11.1 ± 3.6 s in extended-closed, measured over 45, 75 and 36 separate visits. The intermediate is both the most visited and the shortest-lived, which is what a waypoint looks like.

The hop probabilities above are only meaningful if the molecule has no memory, that is, if its chance of leaving a state in the next second does not depend on how long it has already been there. That is testable. If there is no memory, the distribution of visit lengths must be exponential.

Panel D plots, for each state, the fraction of visits still running after a given time, counting visits cut short by the end of a recording as unfinished rather than discarding them. All three match an exponential, at p = 0.65, 0.14 and 0.33, and a more flexible distribution that would reveal any time dependence returns shape parameters of 1.03, 1.06 and 1.22 where 1 means no dependence. Fits allowing memory are not meaningfully better. The molecule does not keep track of how long it has been folded, which is what the state model assumed.

Three unrelated ways of extracting a timescale agree. The state models, fitted using one, two and three shape measurements, give a slowest relaxation of 7.1 to 9.7 s and a faster one of 3.6 to 5.7 s. Simply asking how long the extension measurement stays correlated with itself gives 5 to 8 s. The visit lengths above give 8 to 15 s. Surface-bound αVβ3 rearranges on a timescale of seconds to tens of seconds.

## How many states there are, and whether the fit finds them

Fitting a state model is an optimisation that can settle into a merely good answer instead of the best one, depending on where it starts. Ten random starting points were tried. One found the solution reported above; eight settled into a distinctly worse one with the states shifted about 6 Å too low; one collapsed. The reported fit was reached from a starting point chosen using what is known about the molecule, and starting at random would have returned the worse answer nine times in ten. This is worth stating because the worse answer also looks entirely reasonable.

How many states there are is also a fitted choice. Two standard criteria, which reward fitting the data and penalise using more parameters to do it, both prefer four states over three by a wide margin. The four sit at 54.5, 66.0, 75.0 and 82.5 Å: the intermediate and extended-closed states are unchanged, and the bent state has split into a very folded form and a partly folded one. Three states are used as the main interpretation because they correspond to the three shapes the structural literature names, and every conclusion above holds either way. The fourth state is a refinement of the bent state, not a new shape.

## The molecule does not behave the same way throughout a recording

The population estimates above are time averages, and the underlying trajectory is not stationary.

Everything above is an average over the whole recording, and that is only meaningful if the molecule behaved the same way throughout. It did not.

Cutting each recording into 50-frame blocks and asking whether each block's state fractions are consistent with a single fixed set of proportions, at a threshold corrected for the number of blocks tested, rejects that in 7 of 28 cases in recording 1 and 27 of 100 in recording 2. Recomputing the energy profile block by block shows its lowest point wandering by about 8 Å either side, which is wider than the intermediate band itself, ranging from 55.5 to 79.2 Å in recording 1 and 54.3 to 84.0 Å in recording 2. The local depth of the profile varies by 1.9 to 2.9 kcal/mol between blocks, and up to 4.4.

![Change-point detection on FES minima and CV0 trajectories](/images/2026-09-02/change_point_detection.png)
*The position of the energy minimum block by block with detected transitions marked (top row) and the running statistic used to find them (second row), then the same for the frame-by-frame extension (third and bottom rows), for recording 1 (left) and recording 2 (right).*

The change is not a slow drift. Two methods were used, neither of which assumes where a change might be. The first accumulates the running departure from the overall average and asks whether its largest excursion is bigger than shuffling the data at random ever produces; it is, in both recordings, in more than 999 shuffles out of 1000. The second searches for the set of moments that best splits the trajectory into segments of constant average, penalising each extra split. It finds three such moments in recording 1 (at frames 166, 228 and 259) and four in recording 2 (117, 200, 1049 and 1205). The segments hold steady averages of 74.2, 69.1, 77.7 and 64.3 Å, and 80.8, 62.9, 71.1, 65.4 and 76.4 Å. Recording 2 contains a 14-minute stretch sitting in the intermediate, bracketed by abrupt changes at either end.

Six of those seven moments coincide, to within ten seconds, with a state change identified by the hidden Markov model, which was fitted without reference to them. Two independent methods find the same transitions.

The wandering is the molecule, not the instrument. Across the 25 blocks of recording 2, the blocks with more bent frames are exactly the blocks whose energy minimum sits at lower extension, and the blocks with more extended-closed frames are the ones whose minimum sits higher, each relationship accounting for about 60 % of the variation. A separate check found nothing in the acquisition metadata, such as scan settings or elapsed time, that explains it.

The rates, in contrast, do not change. Comparing the distribution of visit lengths between the two recordings, state by state, finds no detectable difference in any of the four tests, and the ratio of mean visit lengths between recordings is consistent with 1 in every case.

That resolves the apparent contradiction. The molecule leaves each state at a fixed rate, and the fraction of a short window it happens to spend in each state still varies, exactly as a fair coin gives runs of heads. Twenty-one minutes is a short window when visits last ten seconds. The practical consequence: the two recordings can be pooled to estimate rates, and should not be pooled to state occupancies without also reporting how much those vary within each recording.

## The recordings show no sign of the fully activated shape

Everything so far has used a single measurement, extension. The extended-open form is distinguished from extended-closed by a second one, the separation between the two head domains, which is about 36 Å when they are packed together and taken as open above 50 Å. A state model was refitted using both measurements, with one of its four states deliberately started inside the extended-open region, to see whether the fit would keep a state there if the data supported one.

It did not. The seeded state slid down to 41.6 Å, below the threshold, and ended up describing only 10 % of the frames. The largest head separation anywhere in the 1645 frames is 48.3 Å, which is still short of open. Adding a third measurement changes nothing.

This says something about the fitted frames, and the fitted frames can only be as open as the library allows. Since the library contains no structure with an open headpiece, a fit can never report one. The absence of an extended-open state in these recordings is therefore not evidence that the molecule did not adopt it. The two possibilities cannot be separated with this library, and saying so is the honest reading.

## What remains

The missing extended-open structures bound every result above, and no further analysis of the existing fits can fix it. It needs simulations that deliberately drive the molecule over the energy barrier that ordinary simulation cannot cross. Until then, the 2.02 kcal/mol floor is the strongest statement available about that state.

A proper calculation of which way the molecule will fall when poised at the top of the barrier requires simulations started from that point. The 60-to-40 split measured here on leaving the intermediate is the closest thing the recordings can supply, and is not the same quantity.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
