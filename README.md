# Week 5: AutoML & No-Code Model Training

Trained a custom image classifier with Google Teachable Machine and compared generic vs fine-tuned Hugging Face models for the **document forgery detection** component of our **Fraud Investigation** capstone project.

## Custom Model Training

- Built a **Forged vs Authentic** financial document image classifier with Teachable Machine
- Training set: 27 forged images, 20 authentic images
- Achieved **90% accuracy** on 10 held-out test images
- Precision: **83.3%** | Recall: **100%** | F1: **90.9%**

## Fine-Tuned Model Comparison

Compared 3 models (1 generic + 2 fine-tuned) on 5 fraud investigation text inputs:

- **Generic:** `distilbert-base-uncased-finetuned-sst-2-english` (binary sentiment)
- **Fine-Tuned A:** `mrm8488/bert-tiny-finetuned-sms-spam-detection` (spam detection)
- **Fine-Tuned B:** `cardiffnlp/twitter-roberta-base-sentiment-latest` (3-class sentiment)

## Finding

Recommended the **generic DistilBERT model** as a short-term baseline for fraud alert triage because it most consistently flagged suspicious financial scenarios with high confidence (0.92–0.99). Neither fine-tuned model was trained on financial fraud data — Fine-Tuned A (SMS spam) only weakly detected one fraud case, and Fine-Tuned B (Twitter sentiment) classified all inputs as neutral. The ideal long-term solution would be a model fine-tuned specifically on labeled financial fraud narratives. **Recall** is the priority metric — missing a real fraud case is far more costly than a false alarm.

See `report.md` for full analysis.

## Files

| File | Description |
|------|-------------|
| `workflow.json` | n8n workflow — Generic vs Fine-Tuned Comparison |
| `report.md` | Full training and evaluation report |
| `results/comparison-table.csv` | Airtable export with all 5 model comparison records |
| `teachable-machine/screenshots/` | Teachable Machine training interface and test results |
| `metrics/confusion-matrix.md` | Confusion matrix and calculated metrics |
