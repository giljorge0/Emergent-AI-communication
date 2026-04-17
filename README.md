This work on emergent communication is based on the document I wrote (emergentAIcommunication.pdf).
It's a long term project, where I'm implementing a couple smaller experiments
to test some underlying hypothesis and make progress on matters of
communication, symbol systems, and the geometry of meaning across languages.

The analysis of the experiments is in Report_Emergent_Comm.pdf

## Experiment 1 — Discreteness as Error Detection 

**Claim:** discrete channels fail loudly; continuous channels fail silently.

**Key output:** `results/noise_results.json`
Look at `wrong_entropy` vs `noise_level` for each protocol.
Discrete agents should become *more uncertain* when they receive a corrupted
token. Continuous agents stay confident and wrong.

---

## Experiment 2 — ZK Communication Metric 

**Claim:** you can prove communication is real without decoding message content.


**Key output:** `results/zk_results.json`, `results/zk_comparison.png`

The ZK score = CIC × RC where:
- CIC = causal influence of communication (how much scrambling messages
  changes receiver's output distribution)
- RC  = recursive coupling coefficient (does changed receiver behavior
  feed back to change sender's next message?)

A communicating game should score >> 0. A severed channel should score ≈ 0.
Neither computation requires knowing what the messages mean.

---

## Experiment 3 — Opacity Sweep 

**Claim:** the relationship between channel opacity and symbol quality is
non-monotonic. There is an optimal opacity level.

**Key output:** `results/opacity_sweep.json`, `results/opacity_sweep.png`,
`results/rsa_vs_accuracy.png`

Sweeps Gumbel-Softmax temperature τ from 10.0 (near-continuous) to 0.1
(near-discrete). Measures at each level:
- RSA score (symbol–meaning alignment, same metric as Nomura et al.)
- Task accuracy
- Symbol entropy
- Context independence

If RSA peaks at intermediate τ: non-monotonic confirmed (Deutsch hypothesis).
If RSA rises monotonically: confirms Nomura but not the non-monotonic claim.


## Experiment 4 — Concept Gap Finder 

**Claim:** The multilingual embedding space (LaBSE) can be used to systematically
surface concepts that exist in some languages but have no clean equivalent in
others. The geometry of the gap — how spread out the translations are around the
centroid — is a direct measure of untranslatability. This produces a ranked,
empirical map of where languages diverge in their world of ideas.

Embeds words for a set of culturally-loaded concepts across 6+ languages, computes
speaker-weighted centroids, and scores each concept by mean pairwise embedding
distance across translations. High score = hard to translate. Low score = universal
(concrete nouns and primary emotions serve as controls).


Key output: `results/concept_gap_finder.png` — gap score ranking + per-language
isolation heatmap. Red cells = that language diverges most from the concept
consensus. Whole red rows = genuinely untranslatable concept.

---

## Experiment 5 — Translation Validity Score 

**Claim:** Existing translation metrics (BLEU, COMET) don't capture whether a
translation works across diverse contexts — they reward the most common rendering
of a word even if it fails in edge cases. A context-diversity-weighted similarity
score in embedding space is a better measure of whether a translation is genuinely
valid, not just statistically frequent.

For each word, embeds source and translated sentences across multiple contexts.
Rare/unusual contexts get higher weight (they're more diagnostic). TVS =
diversity-weighted mean cosine similarity. Validated by showing it correctly
penalises wrong-sense translations (e.g. translating financial "bank" as "riverbank").

Key output: `results/translation_validity_score.png` — TVS good vs bad per word,
plus per-context breakdown showing which contexts drive the score difference.

---

## Experiment 6 — Interlingua Centroid Tool 

**Claim:** Given a concept, you can compute a speaker-weighted centroid in
multilingual embedding space that represents the universal consensus meaning —
weighted by how many people speak each language. The per-language distance to
that centroid tells you how well each language expresses the concept, and the
centroid itself is a proto-entry for a universal language vocabulary.

Key output: printed report per concept showing per-language distance to centroid,
most central language, most isolated language. The centroid vector itself is the
proposed universal representation of that concept.

---

## Experiment 7 — Universal Language Bootstrapper 

**Claim:** A universal language vocabulary can be bootstrapped from multilingual
embeddings by computing speaker-weighted concept centroids across a large
vocabulary, then projecting into 2D to reveal the geometry of the world of ideas.
This is the data-driven alternative to Esperanto: instead of one person's
linguistic intuitions, the vocabulary emerges from the aggregate of all speakers
weighted by population.

Embeds 20 seed concepts across 6 languages, computes weighted centroids, projects
with UMAP, and plots concept clusters. The stars in the output are proto-tokens:
the universal embedding for each concept, from which a phonetic token would be
assigned in a full pipeline.

Key output: `results/universal_language_bootstrap.png` — concept centroids
(stars) surrounded by per-language word clouds. Tight clusters = safe to assign
a universal token. Diffuse clusters = the concept needs more work.

Next steps: load 50k-word frequency lists per language, find nearest word to each
centroid per language as a proto-definition, assign phonetic tokens by most common
sound pattern across the cluster.

---

## Experiment 8 — AI-AI Dense Communication Protocol (`src/dense_protocol.py`)

**Claim:** Two agents communicating by passing embedding vectors directly rather
than decoding to text transmit significantly more information per byte. The
benchmark is bits of correct identification per byte sent. This tests the section 4
hypothesis: that the move away from token-space toward latent-space communication
is a genuine efficiency gain, not just a different representation.

Runs a referential game in two modes: text protocol (agent sends a text
description, receiver embeds it and picks closest candidate) vs dense protocol
(agent sends a float16-quantized embedding vector directly). Measures accuracy,
message size, and information efficiency. Also sweeps noise levels on the dense
channel to find its robustness boundary.

Key output: `results/dense_communication_protocol.png` — information density
comparison (bits/byte) and noise robustness curve.

Next steps: train a compression encoder reducing 768-dim to 64-dim while
preserving accuracy (find the minimum viable channel width), add an
interpretability layer that decodes transmitted vectors back to nearest words in
multiple languages for human auditability, and test multi-hop degradation (A→B→C)
comparing text vs dense.

