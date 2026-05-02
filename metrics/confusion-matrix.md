# Confusion Matrix — Forged vs Authentic Document Classifier
 
## Teachable Machine Results
 
|  | Predicted: Forged | Predicted: Authentic |
|---|---|---|
| **Actual: Forged** | TP = 5 | FN = 0 |
| **Actual: Authentic** | FP = 1 | TN = 4 |
 
## Calculated Metrics
 
| Metric | Formula | Result |
|--------|---------|--------|
| Accuracy | (TP + TN) / Total | 9/10 = **90%** |
| Precision | TP / (TP + FP) | 5/6 = **83.3%** |
| Recall | TP / (TP + FN) | 5/5 = **100%** |
| F1 Score | 2 × (P × R) / (P + R) | **90.9%** |
 
## Interpretation
 
- **Recall = 100%** — the model caught every forged document in the test set
- **1 false positive** — img_auth1 (authentic) was misclassified as Forged at 88% confidence
- For fraud investigation, **recall is the priority metric** — missing a forgery is more costly than a false alarm
 
