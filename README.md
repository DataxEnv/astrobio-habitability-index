# Comparative Habitability Index of Solar System Worlds

This project constructs a quantitative **Habitability Index (HI)** for ten planetary bodies in our solar system. Rather than scoring for Earth-like surface conditions, the index is explicitly framed around **astrobiological interest** — the potential for life to have existed, or to currently exist, within subsurface niches. The analysis is implemented as a reproducible Python workflow in a Jupyter notebook.

## Overview

The search for life beyond Earth has shifted decisively toward subsurface environments. Ocean worlds such as Europa and Enceladus, once considered too cold and distant to be relevant, are now among the most intriguing targets in planetary astrobiology. This project formalises that shift into a scoring framework, weighting the prerequisites for life not by proximity to the Sun or similarity to Earth's surface, but by the presence of liquid solvents, internal energy sources, chemical complexity, and geophysical activity capable of sustaining biochemistry over geological timescales.

**Ten bodies are evaluated:**
Earth · Mars · Europa · Ganymede · Enceladus · Titan · Ceres · Triton · Pluto · Callisto


## Methodology

### 1. Dataset

Each body is characterised across five astrobiological dimensions, comprising thirteen variables drawn from published planetary science literature and mission data (Cassini, Galileo, New Horizons, Mars Reconnaissance Orbiter):

| Dimension | Variables |
|---|---|
| Temperature | Surface temperature (K) |
| Solvent Systems | Subsurface ocean, surface ocean, hydrocarbon liquids |
| Energy Sources | Solar flux (relative to Earth), tidal heating, radiogenic heating |
| Chemistry | Organic complexity, redox disequilibrium |
| Geophysical Activity | Tectonics, hydrothermal activity |

Ordinal chemistry variables are scored on a 1–5 scale: 
0 = none detected
1 = simple organics
2 = hydrocarbons + nitriles
3 = prebiotic-relevant molecules
4 = complex polymers/large organics
5 = biologically rich chemistry (Earth-like)

Energy variables are scored on a 0–1 continuous scale. Solvent and geophysical variables are encoded as binary (0/1), where 1 indicates confirmed or strongly inferred presence and 0 indicates absence or no credible evidence.

Several variable assignments reflect deliberate scientific judgment and carry inherent uncertainty. Key encoding decisions are documented below:

- **Triton** (`subsurface_ocean = 1`): A subsurface ocean is considered plausible based on tidal and radiogenic heating models, though direct observational evidence is limited.
- **Pluto** (`subsurface_ocean = 0`): A conservative encoding. Recent modelling from New Horizons data suggests a possible subsurface ocean; this is noted as a limitation.
- **Titan** (`hydrothermal_activity = 1`): Inferred from radiogenic heating budget and internal thermal models; not directly confirmed.
- **Callisto** (`hydrothermal_activity = 1`): Tentatively assigned based on possible subsurface ocean and heat flux estimates.

### 2. Exploratory Analysis

Descriptive statistics and variable distributions are examined prior to normalization to identify scale mismatches, outliers, and encoding anomalies. Solar flux spans approximately four orders of magnitude across the dataset (Earth: 1.0; Pluto: 0.0006), making pre-normalization treatment of energy variables necessary before aggregation.

### 3. Normalization

A **hybrid physical + astrobiological normalization** scheme is applied. Variables are rescaled against biologically meaningful reference brackets rather than dataset extremes or Earth values. This preserves the scientific interpretation of each score and ensures the index is stable across different body selections.

**Temperature** is normalized using a Gaussian function centered on 275 K (σ = 100), rewarding bodies with surface temperatures near the liquid water stability window and penalizing extreme cold without artificially capping biological plausibility:

```
temp_norm = exp( -((T - 275)² / (2 × 100²)) )
```

**Solvent systems** are scored as a weighted composite and divided by the theoretical maximum (2.4), ensuring the scale is absolute rather than relative to the dataset:

```
solvent_score = 1.0 × subsurface_ocean + 0.9 × surface_ocean + 0.5 × hydrocarbon_liquids
solvent_norm  = solvent_score / 2.4
```

Surface ocean scores slightly lower than subsurface (0.9 vs 1.0) because surface-exposed liquid water is more vulnerable to radiation, atmospheric loss, and UV flux — factors that reduce long-term habitability potential.

**Energy sources** are normalized in two stages. Solar flux is first normalized to the dataset maximum to prevent its wide dynamic range from numerically overwhelming tidal and radiogenic contributions. Internal energy sources are then upweighted relative to solar flux, reflecting the subsurface framing of the index:

```
solar_norm      = solar_flux / max(solar_flux)
internal_energy = 0.6 × tidal_heating + 0.4 × radiogenic_heating
energy_norm     = 0.3 × solar_norm + 0.7 × internal_energy
```

Tidal heating is weighted above radiogenic heating within the internal energy term because it is more dynamically active and more directly implicated in maintaining liquid water at the water-rock interface.

**Chemistry** variables are divided by the ordinal scale maximum (5):

```
chemistry_norm = (organic_complexity/5 + redox_disequilibrium/5) / 2
```

**Geophysical activity** is divided by the theoretical binary maximum (2):

```
geo_norm = (tectonics + hydrothermal_activity) / 2
```

All normalized columns are bounded to [0, 1] and validated prior to weighting.

### 4. Weighting Model

Weights are assigned to each normalized dimension to reflect their relative importance for astrobiological interest. A liquid solvent medium is treated as the primary non-negotiable prerequisite; energy is ranked above chemistry on the basis that no energy budget renders chemical complexity irrelevant regardless of its richness; surface temperature is assigned the lowest weight given its weak direct relevance to subsurface niches.

| Dimension | Weight | Rationale |
|---|---|---|
| Solvent Systems | **0.30** | Primary non-negotiable prerequisite for biochemistry |
| Energy Sources | **0.25** | Sustains chemistry and liquid water over geological time |
| Chemistry | **0.20** | Organic complexity and redox gradients directly feed biology |
| Geophysical Activity | **0.15** | Maintains chemical gradients and water-rock interaction |
| Temperature | **0.10** | Weak proxy for subsurface conditions; least direct relevance |

Weights sum to 1.0.

### 5. Habitability Index

The index is computed as a weighted linear aggregation of the five normalized dimensions:

```
HI = 0.30 × solvent_norm  +
     0.25 × energy_norm   +
     0.20 × chemistry_norm +
     0.15 × geo_norm       +
     0.10 × temp_norm
```

---

## Results & Discussion

### Ranked Scores

| Rank | World | Habitability Index |
|---|---|---|
| 1 | Earth | 0.823 |
| 2 | Europa | 0.597 |
| 3 | Enceladus | 0.588 |
| 4 | Titan | 0.555 |
| 5 | Ganymede | 0.544 |
| 6 | Triton | 0.444 |
| 7 | Callisto | 0.402 |
| 8 | Ceres | 0.289 |
| 9 | Mars | 0.232 |
| 10 | Pluto | 0.111 |

### Tier A — Strong Astrobiological Candidates (HI > 0.55)

**Earth (0.823)** scores highest across all dimensions and serves as the calibration anchor. The gap between Earth and the next-ranked body (~0.23) reflects its unique combination of a confirmed surface ocean, rich redox chemistry, active tectonics, and favorable surface temperature — conditions that are not replicated in full by any other body in the dataset.

**Europa (0.597) and Enceladus (0.588)** score nearly identically, which is consistent with their comparable standing in the current astrobiological literature. Both possess confirmed or strongly inferred subsurface liquid water oceans, high tidal heating from gravitational interaction with Jupiter and Saturn respectively, and strong redox disequilibrium signals. Enceladus's active plume system (confirmed by Cassini) provides direct evidence of water-rock interaction and organic chemistry at the ocean floor — the closest analogue to deep-sea hydrothermal systems identified beyond Earth. Europa's larger ocean volume and stronger tidal energy budget position it as an equally compelling target. The near-tie between these two bodies is a positive indicator of index internal consistency.

**Titan (0.555)** scores strongly on organic chemistry, the highest organic complexity score in the dataset, driven by its dense nitrogen-methane atmosphere and confirmed surface hydrocarbon lakes. Its internal energy budget (moderate tidal and radiogenic heating) supports a possible subsurface water-ammonia ocean. Titan represents a chemically rich environment that challenges water-centric definitions of habitability; its high chemistry score partially compensates for uncertainty in its solvent and energy systems.

### Tier B — Moderate Astrobiological Interest (HI 0.40–0.55)

**Ganymede (0.544)** sits just below Titan despite being the largest moon in the solar system and the only one with an intrinsic magnetic field. A subsurface saline ocean is strongly supported by Hubble observations of auroral rocking. However, Ganymede's ocean is likely sandwiched between ice layers, limiting water-rock contact and therefore hydrothermal chemistry — a key distinction from Europa and Enceladus that is reflected in its lower energy and geophysical scores.

**Triton (0.444)** is an underappreciated candidate. Active nitrogen geysers observed by Voyager 2, a possible subsurface ocean sustained by tidal heating from its retrograde capture orbit, and a young resurfaced terrain suggest ongoing geophysical activity. Its moderate chemistry scores and limited mission data introduce uncertainty.

**Callisto (0.402)** scores similarly to Triton but is geophysically quieter. Magnetic induction measurements suggest a subsurface ocean, but the absence of confirmed tectonic or hydrothermal activity limits its score. Callisto's position near the bottom of Tier B highlights the importance of internal dynamics — ocean presence alone is insufficient without an active energy and chemistry budget.

### Tier C — Low Astrobiological Interest (HI < 0.35)

**Ceres (0.289)** shows evidence of briny subsurface water and possible cryovolcanic activity (bright spots in Occator Crater imaged by Dawn), but lacks confirmed hydrothermal activity and scores poorly on chemistry. It represents a marginal case — potentially habitable in localized subsurface brines but with limited sustained energy and chemical complexity.

**Mars (0.232)** is the most notable result requiring discussion. Despite being the most extensively studied potentially habitable body beyond Earth, Mars ranks ninth in this index — below ocean worlds with far less direct observational coverage. This outcome is a direct consequence of the index framing: Mars's peak habitability was in its Noachian period (~3.5–4 Ga), when liquid surface water, volcanism, and a thicker atmosphere were present. Today, Mars has no confirmed liquid water body, low tidal heating, and weak redox disequilibrium at the surface. The index scores *current astrobiological interest* weighted toward subsurface liquid systems and internal energy — not historical habitability or surface geomorphology. Mars's low score is therefore not a dismissal of its astrobiological significance, but a consequence of this methodological framing. Subsurface perchlorate brines and deep geothermal gradients may sustain limited niches not captured by this variable set.

**Pluto (0.111)** correctly anchors the lower bound. No confirmed ocean, negligible tidal heating, minimal internal energy, and low chemical complexity leave little basis for astrobiological interest under current conditions.


## Limitations

**Binary encoding of complex phenomena.** Hydrothermal activity, tectonics, and ocean presence are encoded as binary or low-resolution ordinal variables. These compress a wide range of geological complexity — the difference between Enceladus's confirmed active seafloor venting and Ganymede's possible but passive ocean floor is not fully captured.

**Static representation.** The index represents current or near-current conditions. Mars's historical habitability during the Noachian period is not scored. A temporally integrated index would require different variable definitions.

**Water-centric framing.** Subsurface ocean presence is weighted most heavily, which implicitly prioritizes water-based biochemistry. Titan's methane-ethane surface lakes and potential for alternative biochemistry are partially captured through the hydrocarbon liquids variable, but non-aqueous habitability is not fully represented.

**Sparse observational coverage.** Several variable assignments — particularly for Triton, Callisto, and Ceres — are inferred from models or limited flyby data rather than dedicated orbital missions. Index scores for these bodies carry higher uncertainty than for well-characterised targets like Europa, Enceladus, and Mars.

**Linear aggregation.** The weighted sum assumes that habitability dimensions contribute independently and additively. In reality, interactions between dimensions (e.g., liquid water is only relevant if energy is also present) are non-linear. 

---

## References
Bankole, O. (2025). Life Beyond Earth: Habitable Environments in Our Solar System. 
*Medium*. https://medium.com/@olamidebankole/life-beyond-earth-habitable-environments-in-our-solar-system-4443eb5418ff

Hendrix, A.R. et al. (2019). The NASA Roadmap to Ocean Worlds. *Astrobiology*, 19(1), 1–27.

Lunine, J.I. (2017). Ocean worlds exploration. *Acta Astronautica*, 131, 123–130.

McKay, C.P. (2014). Requirements and limits for life in the context of exoplanets. *Proceedings of the National Academy of Sciences*, 111(35), 12628–12633.

Nimmo, F. & Pappalardo, R.T. (2016). Ocean worlds in the outer solar system. *Journal of Geophysical Research: Planets*, 121(8), 1378–1399.

Sparks, W.B. et al. (2016). Probing for evidence of plumes on Europa with HST/STIS. *The Astrophysical Journal*, 829(2), 121.

Stern, S.A. et al. (2015). The Pluto system: Initial results from its exploration by New Horizons. *Science*, 350(6258), aad1815.

Waite, J.H. et al. (2017). Cassini finds molecular hydrogen in the Enceladus plume: Evidence for hydrothermal processes. *Science*, 356(6334), 155–159.
