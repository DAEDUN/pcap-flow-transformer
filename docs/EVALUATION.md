# Evidence and evaluation notes

The original report, supplied notebook and saved outputs, training/inference CSVs, predictions and checkpoint tensor metadata were compared. **Training and inference were not rerun.** Raw packet records and report identity fields are not published here.

## Verified artifacts

| Evidence | Finding |
| --- | --- |
| Repository vs supplied training notebook | Code-cell contents match |
| Training CSV | 29,552 rows, 77 labels |
| After `num_packets > 10` | 29,264 retained; 288 excluded |
| Retained per-label counts | 312–413 |
| Retained sequence lengths | 11–131,182; 3,746 exceed 128 packets |
| Sequence integrity | No length/direction/time array-size mismatches; no decreasing timestamps in retained rows |
| Exact sequence-string duplicates | None within either retained CSV; not a session-leakage test |
| Inference CSV | 231 unlabeled rows; 229 retained; 2 excluded |
| Prediction CSV | 229 rows; `id`, `label`; 76 distinct predicted labels |
| Saved split | 26,337 training / 2,927 validation flows |
| Saved Macro / Micro / Weighted F1 | 0.9103844324 / 0.9098052614 / 0.9093325871 |
| Best epoch | 69 of 120 |
| Checkpoint tensor metadata | 67 state entries; input `[192,3]`; 4 encoder layers; final classifier `[77,192]` |

An application-name mapping is not supplied. Prediction rows are not ground truth and cannot establish test accuracy.

## Corrections to earlier descriptions

| Earlier statement | Evidence-based wording |
| --- | --- |
| 120 classes | **77 labels** in training data and checkpoint output |
| Grouped by source to prevent leakage | `from` is absent; fallback groups are row indices; **row-based split** |
| Best epoch 103 | Saved output identifies **epoch 69** |
| Severe long-tail distribution | Retained counts are **312–413 per label** |
| Hybrid loss improved rare classes | Implemented, but no controlled ablation establishes its benefit |
| Verified TLS 1.3 data and dataset provenance | CSV fields cannot establish TLS version or dataset origin |
| First 128 packets alone | Sequence branch uses 128; **statistics use the whole flow** |
| Independent test accuracy of 91% | **Internal validation Macro F1 0.9104**; test file used for inference only |

## Interpretation limits

The validation set also selects the best checkpoint. Row-based splitting does not guarantee separation of related sessions, sources or capture periods. Absence of exact duplicate sequences is not a general leakage audit.

`robust_read_csv` silently skips malformed lines. Feature construction aligns arrays to their minimum length; the supplied retained rows have no such mismatch. The full 120 epochs run and the best state is selected afterward: there is no early stopping.

The custom focal-style loss derives `pt` from class-weighted cross-entropy. When weights differ from 1, this differs from the usual focal-loss definition.

The supplied `model.pth` has a compatible shape but is named differently from the notebook output `best_model1.pth`. Tensor metadata does not prove it is the epoch-69 checkpoint. The state dictionary does not bundle the label encoder, feature configuration or environment. If input lacks an `id` column, prediction IDs are retained-row positions after filtering.

## Reading the charts

`logged-epochs.csv` contains saved text logs at epoch 1 and every fifth epoch. Lines connect those samples, not a reconstructed full history. The epoch-69 marker comes from the saved best-epoch summary.

`class-report.csv` contains the printed report's 77 rows. Precision, recall and F1 are rounded to two decimals. Do not derive an exact aggregate score from those rounded values.

## Proposed next steps

1. Define justified session/capture groups and audit overlap.
2. Freeze a held-out test set before model selection.
3. Compare sequence-only, statistics-only, combined features and loss choices under identical conditions.
4. Save label mapping, preprocessing settings and environment with each checkpoint.
5. Preserve source-row IDs through filtering and validate prediction alignment.

These are future improvements, not completed experiments.
