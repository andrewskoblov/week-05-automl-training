# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** Andrew Skoblov  
**Date:** May 2, 2026  
**Capstone Project:** Fraud Investigation  
**My Component:** Document Forgery Detection

---

## Part A: Teachable Machine Training

### Training Setup

- **Task:** Forged vs Authentic financial document image classification
- **Training images per class:** Forged: 27 | Authentic: 20
- **Test images per class:** 5 per class (10 total)
- **Total training time:** ~30 seconds

### Test Results

| # | File Name | Actual Class | Predicted Class | Confidence | Correct? |
|---|-----------|--------------|-----------------|------------|----------|
| 1 | img_auth1 | Authentic | Forged | 88% | No |
| 2 | img_auth2 | Authentic | Authentic | 93% | Yes |
| 3 | img_auth3 | Authentic | Authentic | 78% | Yes |
| 4 | img_auth4 | Authentic | Authentic | 98% | Yes |
| 5 | img_auth5 | Authentic | Authentic | 53% | Yes |
| 6 | img_forg1 | Forged | Forged | 99% | Yes |
| 7 | img_forg2 | Forged | Forged | 99% | Yes |
| 8 | img_forg3 | Forged | Forged | 51% | Yes |
| 9 | img_forg4 | Forged | Forged | 78% | Yes |
| 10 | img_forg5 | Forged | Forged | 98% | Yes |

**Overall: 9/10 correct (90%)**

### Confusion Matrix

|  | Predicted: Forged | Predicted: Authentic |
|---|---|---|
| **Actual: Forged** | TP = 5 | FN = 0 |
| **Actual: Authentic** | FP = 1 | TN = 4 |

### Calculated Metrics

- **Accuracy:** (5 + 4) / (5 + 4 + 1 + 0) = 9/10 = **90%**
- **Precision:** 5 / (5 + 1) = 5/6 = **83.3%**
- **Recall:** 5 / (5 + 0) = 5/5 = **100%**
- **F1 Score:** 2 × (0.833 × 1.0) / (0.833 + 1.0) = **90.9%**

> Note: The confusion matrix screenshots provided show TP=4, FN=0, FP=1, TN=4 (80% accuracy). The discrepancy is due to a recount of the test results table which shows 9/10 correct with all 5 forged images correctly identified. Metrics above reflect the full results table.

### Interpretation

The model achieved perfect recall (100%) on forged documents, meaning it caught every forgery in the test set — the most critical outcome for fraud investigation where missing a forged document carries serious consequences. Precision was slightly lower at 83.3% due to one false positive (img_auth1 classified as Forged at 88% confidence), which represents an acceptable trade-off: a false alarm is far less costly than a missed forgery. To improve the model, adding more diverse authentic document types — especially handwritten checks and photos of documents — would help reduce false positives, as img_auth1 appeared to be a physical document photo that visually resembled the forged training examples.

---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested

1. **Generic:** `distilbert-base-uncased-finetuned-sst-2-english` — binary sentiment (POSITIVE/NEGATIVE), trained on SST-2 movie reviews
2. **Fine-Tuned A:** `mrm8488/bert-tiny-finetuned-sms-spam-detection` — spam vs ham classification (LABEL_0 = ham/legitimate, LABEL_1 = spam/suspicious), trained on SMS spam dataset, validation accuracy 0.98
3. **Fine-Tuned B:** `cardiffnlp/twitter-roberta-base-sentiment-latest` — 3-class sentiment (negative/neutral/positive), trained on ~124M tweets via TweetEval benchmark

### Results

| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Label (Score) | Best Model |
|-------|-----------------------|----------------------------|----------------------------|------------|
| Wire transfer of $47,500 — Lagos — account opened 3 days ago | NEGATIVE (0.9758) | LABEL_1 / spam (0.5040) | neutral (0.8724) | Generic |
| Monthly salary deposit of $4,200 — known employer | NEGATIVE (0.9544) | LABEL_0 / ham (0.7308) | neutral (0.8324) | Fine-Tuned A |
| Credit card — 12 transactions, 4 countries, 6 hours | NEGATIVE (0.9170) | LABEL_0 / ham (0.7229) | neutral (0.9454) | Generic |
| Insurance claim — same VIN stolen twice in 14 months | NEGATIVE (0.9839) | LABEL_0 / ham (0.8611) | neutral (0.5200) | Generic |
| POS terminal — 340 identical $49.99 transactions overnight | NEGATIVE (0.9959) | LABEL_0 / ham (0.5580) | neutral (0.8382) | Generic |

### Analysis

**Generic model strengths:** The generic DistilBERT model consistently flagged all five inputs as NEGATIVE with high confidence (0.92–0.99). While it was not trained for fraud detection, its strong negative sentiment detection aligned reasonably well with the suspicious nature of the fraud scenarios. Its binary output (POSITIVE/NEGATIVE) is simple to interpret and acted as a reliable baseline.

**Generic model weaknesses:** The generic model cannot distinguish between truly suspicious fraud signals and merely negative-sounding language. It flagged the routine salary deposit as NEGATIVE (0.9544), showing it responds to tone rather than fraud-specific patterns. This makes it unreliable as a standalone fraud detector — it would generate far too many false positives in a real workflow.

**Fine-Tuned A weaknesses:** The spam detection model (bert-tiny) was trained on SMS messages, not financial fraud scenarios. Its labels (LABEL_0/LABEL_1) lack meaningful domain context, and it only correctly flagged the wire transfer case as suspicious (LABEL_1, 0.504) while treating the other clear fraud cases as legitimate (LABEL_0). The model's low confidence on the wire transfer (barely above 0.5) further reduces its reliability for this use case.

**Fine-Tuned B weaknesses:** The Twitter RoBERTa sentiment model classified every single input as "neutral" with high confidence (0.52–0.95). While technically fine-tuned on a large dataset, its training domain (social media tweets) does not translate to formal financial fraud descriptions. The neutral classification for all inputs — including clear fraud scenarios — makes it the least useful model for this task.

**Biggest surprise:** The most unexpected result was that the generic DistilBERT model outperformed both fine-tuned models on this task. Despite being trained on movie reviews, its sensitivity to negative language happened to align with fraud scenarios better than the domain-specific alternatives. This highlights that "fine-tuned" does not automatically mean "better" — domain match matters more than fine-tuning alone.

### Recommended Model for My Capstone Component

**Component:** Financial document fraud detection (text-based alert triage)  
**Primary model:** `distilbert-base-uncased-finetuned-sst-2-english` (Generic) — as a short-term baseline, because it most consistently flagged suspicious inputs. However, the ideal long-term solution would be a model fine-tuned specifically on labeled financial fraud narratives.  
**Confidence threshold:** 0.85 — inputs above this threshold would be auto-escalated for human review; inputs below would be logged but deprioritized.  
**Priority metric:** **Recall** — in fraud investigation, missing a real fraud case (false negative) is far more costly than generating a false alarm (false positive). A model that flags everything suspicious for human review is preferable to one that silently passes fraudulent activity.

---

## Limitations & Next Steps

The primary limitation of this evaluation is the small sample size — 5 test inputs is insufficient to draw statistically meaningful conclusions about model performance. With only 5 records, a single correct or incorrect prediction shifts results by 20%. In a real deployment, I would evaluate across 100+ labeled fraud/non-fraud examples to get reliable precision and recall estimates.

The fine-tuned models selected (SMS spam, Twitter sentiment) were not ideal domain matches for financial fraud text. A more effective approach would be to search for models fine-tuned on financial crime narratives, SEC filings, or banking fraud datasets. If no suitable model exists, fine-tuning a base model (e.g., DistilBERT or RoBERTa) on labeled fraud investigation records from John Jay's case databases would produce far more relevant results.

For the Teachable Machine image classifier, expanding the training set to 50+ images per class with greater diversity — including different document types, lighting conditions, and scan qualities — would reduce the false positive rate and improve generalization. Combining image-based document classification with the text-based fraud alert model in a single n8n workflow would represent a more complete fraud investigation tool.
