# FlowTransformer implementation

The full project introduction, architecture, results and reproduction instructions are now at the repository root:

- [Project overview](../README.md)
- [한국어 소개 및 개인 기여](../docs/README.ko.md)
- [Evaluation notes and data checks](../docs/EVALUATION.md)
- [Training notebook](notebooks/train_flow_transformer.ipynb)
- [Preprocessing notebook](notebooks/pcap_preprocessing.ipynb)

**Saved internal validation Macro F1: 0.9104 · 77 observed labels.** The supplied CSV has no `from` column, so the recorded split is row-based rather than session-grouped. This is not an independent test result.
