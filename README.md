The work on emergent communication is based on the document I wrote (emergentAIcommunication.pdf).
It's a long term project, where I'm implementing a couple smaller experiments
to test some underlying hypothesis and progress.

Two experiments (1 and 3) testing the hypothesis that **discreteness in communication
channels is epistemically load-bearing** — not just a practical limitation but
the mechanism that enables error detection, symbol stability, and faithful
world-model alignment.

One experiment (2) moving towards a zero knowledge proof of communications among agents,
building on the causal work in the literature.

Grounded in: Deutsch (2011) *The Beginning of Infinity* + Lazaridou & Baroni
(2020) + Galke & Raviv (2024) + Nomura et al. (2025).

---

## Setup (Kaggle)

```python
# In a Kaggle notebook cell:
!git clone https://github.com/facebookresearch/EGG.git
!pip install -e EGG
!pip install scipy matplotlib seaborn
```

---

## Experiment 1 — Discreteness as Error Detection (`src/game.py`)

**Claim:** discrete channels fail loudly; continuous channels fail silently.

```bash
python src/game.py \
  --n_epochs 60 \
  --protocol both \
  --vocab_size 32 \
  --batch_size 128 \
  --lr 1e-3
```

**Key output:** `results/noise_results.json`
Look at `wrong_entropy` vs `noise_level` for each protocol.
Discrete agents should become *more uncertain* when they receive a corrupted
token. Continuous agents stay confident and wrong.

---

## Experiment 2 — ZK Communication Metric (`src/zk_metric.py`)

**Claim:** you can prove communication is real without decoding message content.

```bash
python src/zk_metric.py \
  --n_epochs 60 \
  --vocab_size 16 \
  --query_dim 16 \
  --n_scrambles 20
```

**Key output:** `results/zk_results.json`, `results/zk_comparison.png`

The ZK score = CIC × RC where:
- CIC = causal influence of communication (how much scrambling messages
  changes receiver's output distribution)
- RC  = recursive coupling coefficient (does changed receiver behavior
  feed back to change sender's next message?)

A communicating game should score >> 0. A severed channel should score ≈ 0.
Neither computation requires knowing what the messages mean.

---

## Experiment 3 — Opacity Sweep (`src/opacity_sweep.py`)

**Claim:** the relationship between channel opacity and symbol quality is
non-monotonic. There is an optimal opacity level.

```bash
python src/opacity_sweep.py \
  --n_epochs 80 \
  --vocab_size 32 \
  --n_distractors 9
```

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

---

## References

- Deutsch, D. (2011). *The Beginning of Infinity*. Allen Lane.
- Lazaridou, A. & Baroni, M. (2020). Emergent multi-agent communication in
  the deep learning era. arXiv:2006.02419.
- Galke, L. & Raviv, L. (2024). Learning and communication pressures in
  neural networks. *Language Development Research 5*(1), 116–143.
- Nomura, K. et al. (2025). Decentralized collective world model for emergent
  communication and coordination. arXiv:2504.03353.
