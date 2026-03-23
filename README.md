# Week 5: AutoML & No-Code Model Training

Trained a custom image classifier with Google Teachable Machine and compared generic vs fine-tuned Hugging Face models for the Auto Response Drafting component of our Email Triage & Auto-Responder capstone project.

## Custom Model Training

- Built a Phishing/Legitimate image classifier with Teachable Machine
- Achieved 90% accuracy on 10 held-out test images
- Precision: 83.3% | Recall: 100% | F1: 90.9%
- Model catches all phishing emails but occasionally flags legitimate emails as phishing (1 false positive out of 10 tests)

## Fine-Tuned Model Comparison

Compared 3 models (1 generic + 2 fine-tuned) on 5 cybersecurity test inputs:

- Generic: distilbert-sst-2 (sentiment) — labeled all 5 records NEGATIVE, no useful differentiation
- Fine-Tuned A: mrm8488/bert-tiny-finetuned-sms-spam-detection — moderate differentiation but low confidence, wrong on key records due to SMS training domain
- Fine-Tuned B: SamLowe/roberta-base-go_emotions — correctly identified all inputs as emotionally neutral, most contextually appropriate for technical log entries

## Finding

Recommended SamLowe/roberta-base-go_emotions for Auto Response Drafting because detecting emotional tone in incoming emails enables the Flowise LLM chain to adjust draft response style — empathetic for complaints, straightforward for questions, professional for requests. Fine-tuned models showed more relevant labeling with higher contextual accuracy compared to the generic sentiment model.

See `report.md` for full analysis.