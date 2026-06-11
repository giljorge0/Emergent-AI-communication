# Emergent AI Communication

This project implements a series of experiments testing hypotheses about the relationship
between the *form* of a communication channel and the *quality* of symbols that emerge
from it. It is grounded in the theoretical framework developed in `emergentAIcommunication.pdf`
and informed by Deutsch (2011), Lazaridou & Baroni (2020), Galke & Raviv (2024), and
Nomura et al. (2025).

The full experimental analysis is in `Report_Emergent_Comm.pdf`.

---

## Experiment structure

The project has two connected research programmes:

**Programme 1 — Emergent communication under channel constraints** (Exp 1–3, 1b, 3b, 9b, 14, B2, C1): referential games testing how discreteness, opacity, compression, and production cost shape the quality of emergent symbols.

**Programme 2 — Multilingual concept geometry and universal language** (Exp 4–15, A1, A2): tools treating the LaBSE multilingual embedding space as a window onto a language-independent "world of ideas," building toward a data-driven universal language.

---

## Experiments 1–3: Discrete channel structure

### Experiment 1 — Discreteness as Error Detection

**Claim:** discrete channels fail loudly; continuous channels fail silently.

A referential game comparing discrete (Gumbel-Softmax, vocab 32) and continuous
(tanh-bounded vector) protocols under increasing channel noise. The key diagnostic
is *wrong-answer entropy*: does the receiver become more uncertain when it is wrong?

**Result:** Confirmed. Discrete wrong-answer entropy rises monotonically from 0.947 to
1.560 nats as noise increases from 0 to 1.0. Continuous wrong-answer entropy stays below
0.52 even at maximum noise — the receiver is confidently wrong. Operationalises the
Deutsch (2011) argument that error detection requires discreteness.

**Output:** `results/noise_results.json`

---

### Experiment 1b — REINFORCE Discrete Channel

**Claim:** REINFORCE with a control variate closes the accuracy gap of discrete protocols
without losing the error-detection property.

**Result:** PARTIAL. REINFORCE reaches 76.2% accuracy (vs 67.0% for Gumbel, 9.2
percentage-point improvement). However, the canonical monotonically rising wrong-answer
entropy signature is not cleanly reproduced — a receiver-architecture confound (embedding
lookup vs linear projection) is the likely explanation. Verdict: accuracy gap closes; error
detection needs further ablation.

**Output:** `results/reinforce_results.json`

---

### Experiment 3 — Barrier Opacity Sweep

**Claim:** the relationship between channel opacity and symbol quality is non-monotonic.

Sweeps Gumbel-Softmax temperature τ from 99 (near-continuous) to 0.1 (near-discrete),
measuring RSA score and task accuracy.

**Result:** Partially confirmed. RSA rises monotonically (confirming Nomura 2025) but
task accuracy peaks sharply at τ=1.0 (67.9%) and collapses at τ≤0.5 as symbol entropy
approaches zero. The non-monotonic relationship holds for *accuracy-weighted* RSA, not
raw RSA.

**Output:** `results/opacity_sweep.json`, `results/opacity_sweep.png`

---

### Experiment 3b — Accuracy-Weighted RSA Opacity Sweep

**Claim:** AW-RSA = accuracy × RSA will peak at an interior temperature.

**Result:** CONFIRMED. All metrics (plain RSA, topsim, AW-RSA, AW-topsim, accuracy)
peak at interior temperatures (τ=2 for RSA/topsim/AW metrics, τ=1 for accuracy). Symbol
entropy is itself non-monotonic with a peak near τ=1. The non-monotonic opacity hypothesis
is definitively confirmed for accuracy-weighted symbol quality.

---

## Experiments 4–7: Multilingual concept geometry

### Experiment 4 — Concept Gap Finder

**Claim:** translation embedding spread measures untranslatability; emotional-aesthetic
concepts have higher gap than physical or moral ones.

**Result:** Confirmed. Controls (joy 0.081, table 0.096) score ~4× lower than cultural
concepts (hygge 0.274–schadenfreude 0.545). Japanese aesthetic concepts and Yaghan words
top the ranking. Abstract moral concepts are tighter than emotional-aesthetic ones.

**Output:** `results/concept_gap_finder.png`

---

### Experiment 5 — Translation Validity Score

**Claim:** context-diversity-weighted LaBSE similarity correctly penalises wrong-sense
translations in rare contexts.

**Result:** TVS correctly ranks correct > wrong-sense for all three test words. Gap largest
for *bank* (0.119), smallest for *saudade* (0.021). Rare contexts receive higher weight and
are more diagnostic.

**Output:** `results/translation_validity_score.png`

---

### Experiment 6 — Interlingua Centroid Tool

**Result:** English is most central for all tested concepts. Abstract moral concepts have
max centroid distances 3× smaller than emotional-aesthetic ones. Arabic and Italian are
consistently the most isolated languages.

---

### Experiment 7 — Universal Language Bootstrapper

**Result:** 20 seed concepts across 6 languages produce interpretable UMAP clusters.
Physical elements and primary emotions cluster tightly (safe for universal tokens); abstract
mental states show more spread. Sets up the scale-up in Exp 11/A1.

**Output:** `results/universal_language_bootstrap.png`

---

## Experiments 8–9: Dense communication and universal grammar

### Experiment 8 v2 — AI-AI Dense Communication Protocol

**Claim:** a compressed embedding channel can outperform text in information efficiency.

**Key results:**
- k=4 achieves 100% accuracy at 8 bytes, 5.8× more efficient than text
- k=2 MRR (0.268) *exceeds* raw LaBSE (0.174) — extreme compression improves compositionality (information bottleneck effect)
- Multi-hop relay: ceiling effect at 20 concepts; needs 200+ for meaningful degradation pressure
- Interpretability: in-sample vectors decode cleanly; held-out vectors do not (see Exp 13)

**Output:** `results/exp8_v2_sweep.png`, `results/exp8_v2_compositionality.png`

---

### Experiment 8b — Controlled-Duration Bottleneck Compositionality

**Result:** MIXED, trending toward training artefact. The k=2 compositionality peak
disappears under both convergence and matched-loss controls; k=4 dominates at convergence.
k=2 converges ~2× faster than larger k, confirming a training-duration confound. The
compositionality sweet spot is k=4, not k=2.

---

### Experiment 9 — Universal Grammar Structural Probes

Five probes on 56 concepts across 5 languages after a compositional projection (Phase 1):

| Probe | Verdict |
|---|---|
| P1: Category emergence | EMERGENT (ARI=0.519 raw) |
| P2: Argument structure (transitivity) | NOT_DETECTED (= chance) |
| P3: Zipf/Economy | DETECTED in projected (R²=0.855) |
| P4: Cross-linguistic RSA | NOT_DETECTED* |
| P5: Semantic directions | EMERGENT in projected (acc=0.474) |

Phase 1 projection MRR: 0.126 → 0.451 (3.6× gain). Phase 3 Lewis game: symbol entropy
*rises* (anti-efficient encoding; production cost needed).

**Output:** `results/exp9_full_summary.png`, `results/exp9_ug_probe_results.csv`

---

### Experiment 9b — Probe 4 Projected + Production-Cost Lewis Game

**Part 1:** Probe 4 NOT_DETECTED verdict not reversed by Phase 1 projection; projection
widens the gap (min univ–lang ρ: 0.686→0.536).

**Part 2:** Entropy-penalty production cost fails to recover Zipfian structure (wrong sign:
penalises collapse toward uniform usage rather than skewed Zipfian). Correct intervention
is a *length penalty* or *frequency-based cost*.

---

## Experiments 10–15: Grammar, vocabulary, and phonology

### Experiment 10 — Grammatical Frame Induction

**Claim:** frame roles (AGENT, THEME, EXPERIENCER, etc.) have continuous geometric
structure, reversing the NOT_DETECTED verdict from Exp 9 Probe 2.

**Result:** Confirmed. Direction ρ = 0.821/0.805 for agent/patient (p<0.0001). 6-class role
classification LOO accuracy = 0.554 raw → 0.607 projected (chance = 0.167). Probe C
achieves 100% agent identification in projected space. AGENT/THEME nearly perfectly
separable; EXPERIENCER is the main confusion class. Frame role suffixes can be grounded
in data-driven geometry.

**Output:** `results/exp10_frame_induction.png`

---

### Experiment 11 — Wiktionary Frequency Centroid Sweep

Scales Exp 7 bootstrapper from 20 concepts to 2000 words × 6 languages. Pivot alignment
on English, HDBSCAN synonymy collapse, gap-score translatability tiers. Exports
`exp11_universal_vocab.csv` for downstream use.

---

### Experiment 12 — TVS on Corpus Contexts (Tatoeba)

**Claim:** TVS generalises to naturally occurring parallel text.

**Result:** Human references outperform MT for 3 of 4 words on Tatoeba contexts; mean
weighted gap = +0.0015. Effect sizes substantially smaller than Exp 5 (Tatoeba sentences
are not adversarially sense-diverse). Diversity weighting amplifies the gap for 1 of 4 words.
A production run requires ~10M parallel sentence pairs or a curated sense-diverse corpus.

---

### Experiment 13 — Blind Decoding Audit

**Claim:** tests whether compressed vectors are genuinely human-auditable for held-out
concepts (revising the in-sample Exp 8 interpretability claim).

**Result:** Hit@1 = 0.10 at k=16 (chance = 0.05); Hit@3 = 0.25 (chance = 0.15). The
decoder maps held-out concepts to the nearest training-set attractor (sky/earth/water/life
cluster). The Exp 8 interpretability claim was an in-sample artefact. Fix: larger training
set (500+ concepts) or a dedicated decoding head.

**Output:** `results/exp13_blind_decoding_audit.png`

---

### Experiment 14 — ZK Metric on Partially Communicating Systems

**Claim:** tests whether ZK tracks communication quality continuously (usable as training
signal) or only detects its presence (diagnostic only).

**Result:** PARTIALLY CONTINUOUS. ZK = 0 for all severed baselines (binary detection
confirmed). Spearman ρ(accuracy, ZK) = 0.42 (p=0.20) — insufficient for training signal.
Root cause: RC (sender responsiveness) has std=0.05 vs CIC₂ std=3.14; the multiplicative
formula amplifies RC noise. Recommended fix: additive ZK_revised = α·CIC₁ + β·RC + γ·CIC₂
with variance-equalising weights.

**Output:** `results/exp14_zk_components.png`, `results/exp14_zk_partial_communication.png`

---

### Experiment 15 — Phoneme Centroid Mapping

**Claim:** speaker-count-weighted phoneme consensus naturally satisfies WALS
near-universal phonological constraints without explicit enforcement.

**Result:** CONFIRMED. All five WALS near-universals present (stops, nasals, fricatives,
vowels, approximants). Key finding: English proto-token edit distance = 0.163 vs
0.789–0.921 for all other languages — the phoneme inventory is effectively English-biased.
Mean proto-token edit distance across all language-concept pairs = 0.764 (four-phoneme
tokens are substantially different from source words; optimal token length warrants a sweep).

**Output:** `results/exp15_proto_token_pronunciations.csv`

---

## Scale-up experiments: A1, A2, B2, C1

### Experiment A1 — Master Vocabulary at Scale

5000 words × 12 languages → 4863 content-word concepts with speaker-weighted LaBSE
centroids, HDBSCAN cluster IDs, gap scores, and translatability tiers.

| Tier | Count | % |
|---|---|---|
| Universal (<0.05) | 151 | 3.1% |
| Near-universal (0.05–0.20) | 1766 | 36.3% |
| Culture-specific (0.20–0.40) | 2601 | 53.5% |
| Untranslatable (>0.40) | 343 | 7.1% |

Key findings: 309 HDBSCAN clusters (min_cluster_size too large; re-run with smaller value
to recover expected 2000–4000). Top untranslatable concepts are almost entirely English
proper nouns (Hamilton, NFL, FBI) — NER filtering needed. The translatability atlas shows
universal concepts at the UMAP centre, untranslatable at the periphery.

**Outputs:** `master_vocab.parquet`, `master_vocab_centroids.npy` (14.9 MB, used by A2/B2/C1)

---

### Experiment A2 — Scaled Universal Grammar Probes (409 concepts, 5 languages)

Re-runs Exp 9/10 probes at 10× scale using A1 vocabulary + LaBSE NN translation.

| Probe | Exp 9/10 result | A2 result | Verdict |
|---|---|---|---|
| P1: Category ARI | 0.519 | 0.137 | DEGRADED (metric not scale-stable) |
| P2: Transitivity CV acc | 0.625 (=chance) | 0.745 (=chance) | CONFIRMED NOT_DETECTED |
| P3: Zipf R² (projected) | 0.855 | 0.915 | STRENGTHENED |
| P4: Agent direction ρ | 0.821 | 0.588 | PARTIALLY CONFIRMED |
| P4: Patient direction ρ | 0.805 | 0.600 | PARTIALLY CONFIRMED |

Transitivity NOT_DETECTED is the most definitive result: 200 verbs, identical to majority
baseline. Zipf structure strengthens at scale. Frame role geometry persists (ρ≈0.59,
p<0.0001) but drops from Exp 10 due to noisier automatic annotation.

---

### Experiment B2 — Unified Constraint Grid Search (81 conditions)

Full 3×3×3×3 factorial grid over τ ∈ {0.3, 1.0, 5.0}, k ∈ {2, 8, 32},
L ∈ {1, 2, 4}, cost ∈ {0.0, 0.05, 0.2}.

**Key results:**
- **Message length** is the dominant constraint: L=4 vs L=1 gap = 0.156 AW-RSA
  (larger than τ gap 0.025, k gap 0.043, cost gap 0.011)
- Optimal: τ=1.0, k=32, L=4, cost=0.2 (accuracy=0.959, AW-RSA=0.315) — the
  single Pareto-efficient point
- Intermediate τ=1.0 remains optimal, confirming Exp 3b non-monotonicity at scale
- Production cost raises mean Zipf R² by 7 points but mainly through entropy collapse,
  not genuine Zipfian economy

**Output:** `results/b2_grid_results.csv`

---

### Experiment C1 — Shannon Source/Channel Pareto Frontier

Places all 11 protocols (raw LaBSE, dense k=2–64, Gumbel τ=0.1–10.0, text) on the
compression–resilience plane.

| Protocol | Bytes | Compression | Resilience | Pareto |
|---|---|---|---|---|
| Text (4B avg) | 4 | 0.153 | 0.828 | ★ |
| Gumbel τ=10.0 | 1 | 0.613 | 0.086 | ★ |
| Dense k=2 | 4 | 0.153 | 0.617 | — |
| Dense k=4–64 | 8–128 | 0.005–0.077 | 0.611–0.614 | — |
| Raw LaBSE | 1536 | 0.000 | 0.549 | — |

**Key result:** no protocol dominates on both dimensions — Shannon's theorem is empirically
visible. Dense protocols are not Pareto-efficient: they match text on compression (k=2) but
trail by 0.211 on resilience. Dense protocol resilience is near-uniform across all k (0.611–0.617);
noise robustness is determined by channel type, not compression level.

**Output:** `results/exp_c1_pareto.png`

---

## Pipeline status

The universal language pipeline is now complete end-to-end:

| Component | Experiment | Status |
|---|---|---|
| Draft vocabulary + translatability tiers | A1, Exp 11 | ✓ (4863 concepts, 12 languages) |
| Translation quality metric (TVS) | Exp 5, 12 | ✓ (corpus-validated) |
| Compositional projection | Exp 9 Phase 1 | ✓ (MRR 0.126→0.451) |
| Frame role morphology | Exp 10, A2 | ✓ (agent ρ=0.82/0.59 at 56/409 concepts) |
| Phoneme inventory + proto-tokens | Exp 15 | ✓ (40 concepts, 6 languages) |
| Dense AI-AI channel | Exp 8 | ✓ (k=4 beats text) |
| ZK communication metric | Exp 2, 14 | ✓ (binary detection; not yet continuous) |
| Optimal constraint configuration | Exp B2 | ✓ (τ=1.0, k=32, L=4, cost=0.2) |
| Shannon Pareto frontier | Exp C1 | ✓ |

**Next binding constraint:** scale (500–5000 concepts, 20+ typologically diverse languages)
and integration: running the full pipeline end-to-end with B2-optimal settings on the
A1-scale vocabulary. Immediate next steps:
1. Re-run A1 HDBSCAN with `min_cluster_size=3` to recover finer cluster granularity
2. Add NER filter to A1 to remove English proper nouns from the untranslatable tier
3. Apply Phase 1 compositional projection to the full A1 centroid matrix
4. Apply Exp 10 frame-role geometry to A1 vocabulary (Experiment A3)

---

## References

- Deutsch, D. (2011). *The Beginning of Infinity*. Allen Lane.
- Lazaridou, A. & Baroni, M. (2020). Emergent multi-agent communication in the deep
  learning era. arXiv:2006.02419.
- Galke, L. & Raviv, L. (2024). Learning and communication pressures in neural networks.
  *Language Development Research* 5(1), 116–143.
- Nomura, K. et al. (2025). Decentralized collective world model for emergent communication
  and coordination. arXiv:2504.03353.
- Lu, Y. et al. (2018). A neural interlingua for multilingual machine translation. ACL WMT.
- Feng, F. et al. (2022). Language-agnostic BERT sentence embedding. ACL 2022.
