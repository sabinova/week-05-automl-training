# Confusion Matrix — Teachable Machine Phishing Classifier

## Test Setup

- **Model:** Google Teachable Machine image classifier
- **Classes:** Phishing, Legitimate
- **Training images:** ~25 per class
- **Test images:** 5 per class (10 total, held out from training)

## Confusion Matrix

| | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 5 | FN = 0 |
| **Actual: Legitimate** | FP = 1 | TN = 4 |

## Metrics

| Metric | Formula | Value |
|--------|---------|-------|
| Accuracy | (TP + TN) / Total = 9 / 10 | **90.0%** |
| Precision | TP / (TP + FP) = 5 / 6 | **83.3%** |
| Recall | TP / (TP + FN) = 5 / 5 | **100.0%** |
| F1 Score | 2 × (P × R) / (P + R) = 2 × (0.833 × 1.0) / (1.833) | **90.9%** |

## Cell Definitions

- **TP (True Positive) = 5:** Phishing images correctly identified as Phishing
- **FN (False Negative) = 0:** Phishing images incorrectly labeled as Legitimate (missed threats)
- **FP (False Positive) = 1:** Legitimate image incorrectly labeled as Phishing (false alarm — test_legit_05.png at 97% confidence)
- **TN (True Negative) = 4:** Legitimate images correctly identified as Legitimate

## Key Finding

The model achieved perfect recall (100%) with no missed phishing emails, but its one error was a false positive classified with 97% confidence. This demonstrates that high confidence scores do not guarantee correctness, reinforcing the need for human review in production systems.