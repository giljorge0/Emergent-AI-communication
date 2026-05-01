# Emergent AI Communication

This project implements a series of experiments testing hypotheses about the relationship
between the *form* of a communication channel and the *quality* of symbols that emerge
from it. It is grounded in the theoretical framework developed in `emergentAIcommunication.pdf`
and informed by Deutsch (2011), Lazaridou & Baroni (2020), Galke & Raviv (2024), and
Nomura et al. (2025).

The full experimental analysis is in `Report_Emergent_Comm.pdf`.

---

## Experiments 1–3: Discrete channel structure

### Experiment 1 — Discreteness as Error Detection

**Claim:** discrete channels fail loudly; continuous channels fail silently.

A referential game comparing discrete (Gumbel-Softmax, vocab 32) and continuous
(tanh-bounded vector) protocols under increasing channel noise. The key diagnostic
is *wrong-answer entropy*: does the receiver become more uncertain when it is wrong?

**Result:** Confirmed. Discrete wrong-answer entropy rises monotonically from 0.947 to
1.560 nats as noise increases from 0 to 1.0. Continuous wrong-answer entropy stays below
0.52 even at maximum noise — the receiver is confidently wrong. This operationalises the
Deutsch (2011) argument that error detection requires discreteness.

**Output:** `results/noise_results.json`

---

### Experiment 2 — Zero-Knowledge Proof of Communication

**Claim:** genuine communication can be verified without decoding message content.

A two-turn referential game where the ZK score = CIC × RC, where CIC (causal influence
of communication) measures how much scrambling messages changes outputs, and RC (recursive
coupling) measures whether changed receiver behaviour feeds back to change the sender's
next message. Neither component requires knowing what messages mean.

**Result:** Communicating game ZK score = 0.851 vs severed-channel baseline = 0.000.
ZK ratio ≈ 8.5 × 10⁸. The metric cleanly separates genuine communication from
non-communication with no content decoding.

**Output:** `results/zk_results.json`, `results/zk_comparison.png`

---

### Experiment 3 — Barrier Opacity Sweep

**Claim:** the relationship between channel opacity and symbol quality is non-monotonic;
there is an optimal opacity level (Deutsch + Nomura 2025).

Sweeps Gumbel-Softmax temperature τ from 99 (near-continuous) to 0.1 (near-discrete),
measuring RSA score (symbol–meaning alignment) and task accuracy at each level.

**Result:** Partially confirmed. RSA rises monotonically with opacity (confirming Nomura
but not the non-monotonic prediction). However, task accuracy peaks sharply at τ=1.0
(67.9%) and collapses at τ≤0.5 as symbol entropy approaches zero. The non-monotonic
relationship holds for *useful* symbol quality (accuracy-weighted RSA) even if not for
raw RSA. A composite metric combining both is the natural next step.

**Output:** `results/opacity_sweep.json`, `results/opacity_sweep.png`,
`results/rsa_vs_accuracy.png`

---

## Experiments 4–6: Multilingual concept geometry

### Experiment 4 — Concept Gap Finder

**Claim:** the spread of translation embeddings around a centroid is a direct measure
of untranslatability, and this gap is systematically higher for emotional-aesthetic
concepts than for physical or moral ones.

Embeds translations of culturally-loaded concepts across 6+ languages via LaBSE,
computes speaker-weighted centroids, scores each concept by mean pairwise translation
distance. Controls (*joy*, *table*) validate the measure.

**Result:** Controls score 0.081–0.096. Culturally specific concepts score 0.274–0.545.
*Schadenfreude* (0.545) and Japanese aesthetic concepts (0.330–0.446) cluster at the top.
Abstract moral concepts are tighter than emotional-aesthetic ones.

**Output:** `results/concept_gap_finder.png`

---

### Experiment 5 — Translation Validity Score

**Claim:** a context-diversity-weighted similarity score in embedding space is a better
measure of translation quality than frequency-based metrics, by correctly penalising
wrong-sense translations in unusual contexts.

**Result:** TVS correctly ranks correct > wrong-sense translations across all three test
words (*bank*, *light*, *saudade*). Gap is largest for *bank* (0.119), smallest for
*saudade* (0.021). Rare contexts receive higher weight and are more diagnostic.

**Output:** `results/translation_validity_score.png`

---

### Experiment 6 — Interlingua Centroid Tool

**Claim:** the speaker-weighted centroid of all translations is a proto-entry for a
universal language vocabulary; per-language distance to the centroid measures how much
each language would need to extend to reach the universal concept.

**Result:** English is most central for all tested concepts. Abstract moral concepts
(e.g. *justice*) have max centroid distances 3× smaller than emotional-aesthetic ones
(*saudade*, *hygge*), consistent with Exp 4. Arabic and Italian are consistently the
most isolated languages for the tested concept set.

---

## Experiments 7–9: Universal language structure

### Experiment 7 — Universal Language Bootstrapper

**Claim:** a universal vocabulary can be bootstrapped from multilingual embeddings by
computing speaker-weighted concept centroids, producing proto-tokens whose geometry
reflects the structure of the world of ideas.

Embeds 20 seed concepts across 6 languages, computes weighted centroids, visualises
with UMAP. Physical elements and primary emotions cluster tightly (safe for universal
tokens); abstract mental states show more spread (require multi-token expressions).

**Output:** `results/universal_language_bootstrap.png`

---

### Experiment 8 v2 — AI-AI Dense Communication Protocol

**Claim:** a compressed embedding channel can outperform text in information efficiency
while preserving compositionality. The minimum viable bottleneck dimension is the key
design parameter.

Three new sub-experiments beyond v1:

**1. Bottleneck encoder sweep (k ∈ {2, 4, 8, …, 768})**

k=2 achieves text-level accuracy (95%) at 4 bytes — 11× more information-dense than
text (44 bytes). k=4+ achieves 100% accuracy. Breakeven (dense bits/byte > text
bits/byte) occurs at k=2. The practical crossover where dense beats text on both
accuracy and size is k=4 (8 bytes, 100% accuracy, 5.8× more efficient).

**2. Compositionality under compression (key finding)**

MRR at k=2 (0.268) *exceeds* raw LaBSE (0.174) and all other compression levels.
Extreme compression appears to force the encoder toward the most structured
representation possible — an information bottleneck effect. Every other k drops below
baseline. This connects to the Galke (2024) learnability argument: the extreme
bottleneck acts like a generational reset, selecting for the most compressible
(most structured) solution.

**3. Multi-hop relay (A→B→C, up to 6 hops)**

Both protocols maintain near-perfect accuracy at σ=0.05 noise through 6 hops on
20 concepts. This is a ceiling effect — the task is too easy. Follow-up with 200+
concepts needed to create meaningful degradation pressure.

**4. Interpretability layer**

Compressed vectors decode cleanly to correct words across all 4 tested languages
(en, ru, zh, ar) for all 8 test concepts. The k=16 encoder is human-auditable.

**Output:** `results/exp8_v2_sweep.png`, `results/exp8_v2_relay.png`,
`results/exp8_v2_compositionality.png`, `results/exp8_v2_summary.csv`

---

### Experiment 9 — Universal Grammar Structural Probes

**Claim:** the universal concept space derived from multilingual embeddings may exhibit
UG-like structural properties without explicit grammatical supervision.

Three phases + five probes on 56 concepts across 5 languages.

**Phase 1 — Compositional projection layer**

A lightweight projection is trained with a compositionality loss (L_proximity +
L_compositionality). Analogy MRR improves from 0.126 (raw LaBSE) to 0.451 (projected)
— a 3.6× gain. This is the headline result: UG-like compositional structure is not
present in raw embeddings but can be *induced* cheaply by gradient descent.

**Phase 2 — Codebook**

56/56 codebook entries used, 0 collisions. Full coverage, no degenerate solutions.

**Phase 3 — Lewis signaling game**

Mean cosine reward converges to 0.745 (below 0.8 threshold). Symbol entropy *rises*
over training rather than falling. This replicates the Chaabouni et al. (2019)
anti-efficient encoding finding: without a production cost, agents diversify symbol
use rather than converging on a lean vocabulary. Abstract concepts (memory, freedom,
home) are consistently the hardest to communicate (reward 0.63–0.67).

**Probe results:**

| Probe | Space | Metric | Verdict |
|---|---|---|---|
| P1 Category emergence | raw LaBSE | ARI(sem)=0.519 | **EMERGENT** |
| P1 Category emergence | projected | ARI(sem)=0.433 | **EMERGENT** |
| P2 Argument structure | both | LOO acc=0.625 = chance | NOT_DETECTED |
| P3 Zipf/Economy | raw LaBSE | R²=0.708 | PARTIAL |
| P3 Zipf/Economy | projected | R²=0.855 | **DETECTED** |
| P4 Cross-linguistic RSA | universal | min ρ=0.466 < max cross-lang ρ=0.801 | NOT_DETECTED* |
| P5 Semantic directions | raw LaBSE | opp acc=0.053 | NOT_DETECTED |
| P5 Semantic directions | projected | opp acc=0.474 | **EMERGENT** |

*P4 NOT_DETECTED has an important nuance: D_universal has *lower* correlation with
individual languages than some language pairs have with each other (e.g. es–ru ρ=0.801).
The centroid occupies a structurally different position from any individual language —
it is not just the average. Whether D_projected_universal (Phase 1 space) reverses this
verdict is the key open question.

**What is free vs what must be designed:**
- EMERGENT → UG property arises from information geometry; no explicit grammar rules needed
- DETECTED → present in projected space; achievable via the Phase 1 optimization
- NOT_DETECTED → must be explicitly designed in (Probe 2 / transitivity is the clearest case)

**Output:** `results/exp9_full_summary.png`, `results/exp9_ug_probe_results.csv`,
`results/exp9_phase1_training.png`, `results/exp9_phase3_signaling.png`,
`results/exp9_probe3_zipf.png`

---

## Open questions and next experiments

**Exp 8 follow-up:** Why does k=2 compression *improve* compositionality over raw LaBSE?
Information bottleneck theory predicts this, but it has not been directly tested in
the embedding compression setting. Scale to 200+ concepts to escape ceiling effects
in multi-hop and compositionality tests.

**Exp 9 follow-up (P4):** Recompute the Mantel test using D_projected_universal rather
than the raw centroid space. If the projection layer's space is more correlated with
individual languages than they are with each other, Probe 4 becomes EMERGENT and
the universal language project gains a strong structural argument.

**Lewis game entropy:** Add a production cost (length/entropy penalty) to the signaling
game and rerun. Expect Zipfian symbol distribution to emerge (Galke 2024 inductive
bias). This directly connects Exp 9 Phase 3 to Exp 3's opacity sweep.

**Exp 10 (planned) — Grammar bootstrapper:** Test whether syntactic rules (SVO order,
head-final/head-initial) emerge from concept graph structure in the projected space.

**Exp 11 (planned) — Phonetic token assignment:** Map concept centroids to phonetic
tokens via most common sound pattern across the languages closest to each centroid.

---

## References

- Deutsch, D. (2011). *The Beginning of Infinity*. Allen Lane.
- Lazaridou, A. & Baroni, M. (2020). Emergent multi-agent communication in the deep
  learning era. arXiv:2006.02419.
- Galke, L. & Raviv, L. (2024). Learning and communication pressures in neural networks.
  *Language Development Research* 5(1), 116–143.
- Nomura, K. et al. (2025). Decentralized collective world model for emergent
  communication and coordination. arXiv:2504.03353.
- Lu, Y. et al. (2018). A neural interlingua for multilingual machine translation. ACL WMT.
