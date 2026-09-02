---
layout: post
title: "Free Energy and Kinetics of Surface-Bound αVβ3 from HS-AFM"
---

The template-matching fits give 1645 unbiased frames of αVβ3 conformation at 1 frame per second, 379 from recording V1 and 1266 from V2. Because the recordings are unbiased observations rather than a driven simulation, the frame distribution is an experimental sample of the equilibrium population and supports a free-energy profile, state populations, transition rates and dwell times. This report covers those quantities, the tests that validate them, and the region of conformational space where they are undefined.

Throughout, CV0 is the αV head-thigh to αV calf centroid distance. States are banded as bent-closed at CV0 ≤ 65 Å, intermediate at 65 to 78 Å, extended-closed at 78 to 85 Å, and extended-open above 85 Å.

![Four-panel dynamics synthesis](/images/2026-09-02/dynamics_synthesis_v1.png)
*(A) ΔG(CV0) with 95 % bootstrap confidence band, state bands, and the Bayesian extended-open floor. (B) State populations under three independent definitions. (C) V2 hidden Markov model Viterbi state path with independently detected change-points and the transition matrix inset. (D) Per-state survival curves with exponential fits.*

## Free energy profile

P(CV0) was estimated by kernel density with bandwidth 0.18 over all 1645 frames, and converted to ΔG = −kT ln P at 300 K, normalized to zero at the global minimum. Panel A shows the result.

The minimum sits at CV0 ≈ 70 to 75 Å, in the intermediate band, not at either canonical endpoint. Barriers to the bent and extended-closed sides are each about 4 to 5 kcal/mol. This is inconsistent with a two-state bent-to-extended equilibrium and consistent with a surface-bound ensemble dominated by an intermediate along the extension pathway.

The profile is undefined above CV0 = 85 Å because the conformer library contains no templates there, and the figure hatches that region rather than extrapolating into it. A bound is still available. Of 1645 frames, 42 are fitted at CV0 ≥ 85 Å. The Jeffreys 95 % upper bound on that population is 3.40 %, which places the extended-open free energy at least 2.02 kcal/mol above the intermediate minimum. Enhanced sampling aimed at that state has to overcome at least 2 kT.

## State populations under three definitions

Panel B compares three ways of assigning population, which agree except in the bent band.

| state | Boltzmann, from ΔG | empirical count | HMM stationary |
|---|---|---|---|
| bent-closed | 25 % | 26 % | 40 % |
| intermediate | 46 % | 45 % | 35 % |
| extended-closed | 24 % | 26 % | 24 % |
| extended-open | 5 % | 3 % | not represented |

The hidden Markov model assigns more weight to the bent state because its bent emission has σ = 8.8 Å and absorbs frames in the 50 to 70 Å transition zone. The hard-threshold partition answers "what fraction of time is spent in the geometric bent band"; the Markov model answers "which state is the system kinetically in". Both are internally consistent under their own definitions.

## Kinetics

A three-state Gaussian hidden Markov model fitted by expectation maximization over 73 iterations gives emission means of 61.0, 74.4 and 82.4 Å, with a transition matrix whose self-transition probabilities are 0.921, 0.859 and 0.903 per frame.

Direct bent-to-extended-closed transitions occur at under 1 % per frame in both directions. The ensemble reaches either endpoint through the intermediate. The intermediate is therefore the kinetic gateway and not only the free-energy minimum, which the ΔG profile alone could not establish. Of transitions leaving the intermediate, 59.7 % go to bent and 40.3 % to extended-closed.

Mean dwell times are 14.6 ± 4.3 s in bent, 7.8 ± 1.8 s in intermediate, and 11.1 ± 3.6 s in extended-closed, over 45, 75 and 36 visits. The intermediate has the shortest lifetime, consistent with its gateway role.

Panel D tests whether those dwell times are memoryless. Kaplan-Meier survival curves, with right-censoring at video boundaries honored, pass a Kolmogorov-Smirnov test against the exponential at p = 0.65, 0.14 and 0.33, and Weibull shape parameters cluster at 1.03, 1.06 and 1.22. Akaike weights favor gamma over exponential by less than one unit in two states, below the threshold for a real preference. State residence is Markovian, which validates the Markov assumption the transition matrix rests on.

Independent estimators agree on the timescale. Rate matrices from one-, two- and three-dimensional HMM fits give slow-mode relaxation times of 7.1 to 9.7 s and fast-mode times of 3.6 to 5.7 s, and the CV0 autocorrelation e-folding time is 5 to 8 s. Three methods place αVβ3 surface-bound dynamics on a 5 to 15 s timescale.

## Model selection and initialization

Ten random expectation-maximization initializations were run for the three-state model. One reached the reported optimum at log-likelihood −5286.34; eight converged to a local optimum 51 nats worse, with means shifted about 6 Å low; one was degenerate. The reported solution was obtained from a physically motivated initialization, and random initialization alone would have returned the worse fit.

Both BIC and AIC prefer a four-state model by a wide margin (ΔBIC = −411, ΔAIC = −460). The best four-state means are 54.5, 66.0, 75.0 and 82.5 Å, which splits the bent band into a very-bent and a transition-zone sub-state and leaves the intermediate and extended-closed states unchanged. The three-state model is used as the main interpretation because it matches the established structural nomenclature, and every conclusion above holds under either choice.

## The ensemble is not stationary over the acquisition window

The population estimates above are time averages, and the underlying trajectory is not stationary.

Splitting each recording into 50-frame blocks and testing block state fractions against a stationary Bernoulli null gives 7 of 28 block-state pairs in V1 and 27 of 100 in V2 beyond the Bonferroni-corrected threshold of |z| = 3.55, with maxima of 6.70 and 7.50. Recomputing ΔG(CV0) per block shows the free-energy minimum wandering with a standard deviation of about 8 Å in both recordings, wider than the intermediate band itself, with block minima spanning 55.5 to 79.2 Å in V1 and 54.3 to 84.0 Å in V2. Per-block ΔG standard deviation over the library-supported range is 1.9 to 2.9 kcal/mol with a maximum of 4.4.

![Change-point detection on FES minima and CV0 trajectories](/images/2026-09-02/change_point_detection.png)
*Per-block free-energy minimum position with detected change-points (top row) and its CUSUM statistic (second row), and per-frame CV0 with change-points (third row) and its CUSUM statistic (bottom row), for V1 (left) and V2 (right).*

The structure is stepwise rather than a gradual drift. Per-frame CUSUM rejects stationarity at p < 1e-3 in both recordings. Binary segmentation with a BIC penalty finds three change-points in V1 (frames 166, 228, 259) and four in V2 (117, 200, 1049, 1205), with segment means of 74.2, 69.1, 77.7 and 64.3 Å in V1 and 80.8, 62.9, 71.1, 65.4 and 76.4 Å in V2. V2 contains an 848-frame intermediate plateau between frames 200 and 1048, bracketed by sharp transitions. Six of the seven change-points co-locate with hidden-Markov Viterbi state transitions to within ten frames, so two independent methods recover the same transition structure.

The drift has a state-population explanation rather than an instrumental one. Across the 25 V2 blocks, the bent fraction correlates with the block free-energy minimum at r = −0.773 and the extended-closed fraction at r = +0.756, both significant after Bonferroni correction, accounting for 57 to 60 % of the drift variance per state. A separate check found no metadata covariate that explains it.

Rates, in contrast, are stationary. Comparing V1 and V2 dwell-time distributions per state gives Kolmogorov-Smirnov permutation p > 0.25 in all four tests, and all four mean-ratio 95 % confidence intervals span 1. The two recordings sample the same kinetic process with fixed mean dwell times, and their finite-window occupancies differ, which is the expected behavior of a stationary Markov chain at these sample sizes. The two recordings can therefore be pooled for rate estimation, and should not be pooled for occupancy without reporting the block structure.

## The data do not support an extended-open state

CV2, the αV head to β3 head separation, distinguishes extended-closed from extended-open at a 50 Å threshold. A two-dimensional hidden Markov model was fitted over (CV0, CV2) with a four-state variant deliberately seeded at CV0 = 85 Å and CV2 = 55 Å, inside the extended-open region, to test whether expectation maximization would keep a state there if the data supported one.

The seeded state converged downward to CV2 = 41.6 Å with a stationary weight of 0.10. The maximum CV2 anywhere in the 1645 fitted frames is 48.3 Å, below the threshold. A three-dimensional fit over (CV0, CV1, CV2) recovers the same partition and confirms CV1 as redundant.

This is a statement about the fit set, and the fit set inherits the library. Because the conformer library contains no headpiece-open templates, the absence of an extended-open state in these recordings cannot be distinguished from an inability to represent one.

## What remains

The extended-open gap is the single limitation that bounds every result above, and closing it requires enhanced-sampling MD rather than more analysis of the existing fits. Until then, the 2.02 kcal/mol Bayesian floor is the strongest statement available about that state.

A formal committor analysis requires saddle-seeded unbiased simulation. The experimental analog reported here, the branching ratio of 0.60 to bent on leaving the intermediate, is available from the existing data and is not a substitute.

## Links

- Pipelines and analysis: [https://github.com/dfu99/conformers](https://github.com/dfu99/conformers)
