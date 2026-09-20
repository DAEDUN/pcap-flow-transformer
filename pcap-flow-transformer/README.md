# FlowTransformer — Encrypted Traffic Classification

A Transformer-based classifier that identifies the underlying app/service behind **encrypted (TLS 1.3) network traffic**, using only packet-level metadata (length, direction, inter-arrival time) — no payload content required.

Built as a team project for a university network security course (Fall 2025), using the CSTNET encrypted-traffic PCAP dataset.

## Results

| Metric | Score |
|---|---|
| Validation Macro F1 | **0.9104** |
| Validation Micro F1 | 0.9098 |
| Validation Weighted F1 | 0.9093 |

- 120-class multi-class classification (individual apps/services)
- Best checkpoint at epoch 69 / 120
- Train/val split: 26,337 / 2,927 flows (9:1, grouped by source to prevent leakage)

## Problem

TLS 1.3 encrypts payload content, so traditional deep-packet-inspection approaches can't see *what* is being sent — only metadata about *how* it's being sent. This project tests whether packet length, direction, and timing patterns alone are enough to fingerprint a service, and how well that holds up across 120 imbalanced classes.

## Approach

**Input representation** — each flow is converted into two views:
- A padded 3-channel sequence `[max_len=128, 3]` of (log-scaled length, direction, log inter-arrival time)
- A 13-dim flow-level statistics vector (length mean/std/sum, direction-change ratio, IAT mean/std/max, flow duration, upstream/downstream byte totals)

**Model — `FlowTransformer`**
- Custom `nn.TransformerEncoder` (Pre-LN, 4 layers, 8 heads, `d_model=192`, GELU) over the packet sequence, with a learned CLS token and sinusoidal positional encoding implemented from scratch
- A learned-query multi-head **attention pooling** head over the full sequence output (in addition to the CLS token), so the model isn't relying on a single summary token
- The 13-dim stats vector is projected through a small MLP and concatenated with the CLS and attention-pooled representations before the final classification head

**Handling class imbalance** — the dataset is long-tailed across 120 service classes:
- Hybrid loss: `0.6 × FocalLoss(γ=2.0) + 0.4 × LabelSmoothingCrossEntropy(0.02)`, both weighted by inverse class frequency
- `GroupShuffleSplit` on the flow source to keep the same session out of both train and validation
- `ReduceLROnPlateau` scheduler tracking validation macro F1

## Repo structure

```
notebooks/
  pcap_preprocessing.ipynb      # Parses raw PCAP files into per-flow sequences (scapy) and exports to CSV
  train_flow_transformer.ipynb  # Dataset/model/training pipeline, evaluation, inference
```

## Tech stack

`PyTorch` · `scikit-learn` · `scapy` · `pandas` / `numpy` · Google Colab (GPU)

## My role

I worked on model training and construction (in collaboration with AI tooling for implementation speed) and led experiment comparison/tuning — iterating from an initial Random Forest baseline to the final Transformer architecture, and running the hyperparameter/loss-function experiments that took validation macro F1 from the 0.8 range to 0.91.

## Team

3-person team project (Software Engineering × 2, Public Administration × 1) for a university Network Security course.

## Notes

- Raw PCAP files and the processed CSVs are not included (dataset not redistributable / large).
- Notebooks were run on Google Colab; paths reference Google Drive mounts and will need to be updated to run elsewhere.
