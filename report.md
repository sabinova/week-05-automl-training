# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** Sabina Ruzieva
**Date:** March 22, 2026
**Capstone Project:** Email Triage & Auto-Responder
**My Component:** Component 2 — Auto Response Drafting

---

## Part A: Teachable Machine Training

### Training Setup

- **Task:** Phishing vs Legitimate email screenshot classification
- **Training images per class:** ~20
- **Test images per class:** 5
- **Total training time:** ~30 seconds

### Test Results

| # | File Name | Actual Class | Predicted Class | Confidence | Correct? |
|---|-----------|-------------|-----------------|------------|----------|
| 1 | test_phish_01.png | Phishing | Phishing | 90% | Yes |
| 2 | test_phish_02.png | Phishing | Phishing | 100% | Yes |
| 3 | test_phish_03.png | Phishing | Phishing | 100% | Yes |
| 4 | test_phish_04.png | Phishing | Phishing | 100% | Yes |
| 5 | test_phish_05.png | Phishing | Phishing | 100% | Yes |
| 6 | test_legit_01.png | Legitimate | Legitimate | 99% | Yes |
| 7 | test_legit_02.png | Legitimate | Legitimate | 98% | Yes |
| 8 | test_legit_03.png | Legitimate | Legitimate | 91% | Yes |
| 9 | test_legit_04.png | Legitimate | Legitimate | 100% | Yes |
| 10 | test_legit_05.png | Legitimate | Phishing | 97% | No |

### Confusion Matrix

| | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 5 | FN = 0 |
| **Actual: Legitimate** | FP = 1 | TN = 4 |

### Calculated Metrics

- **Accuracy:** 90% — (5 + 4) / (5 + 4 + 1 + 0) = 9/10
- **Precision:** 83.3% — 5 / (5 + 1) = 5/6
- **Recall:** 100% — 5 / (5 + 0) = 5/5
- **F1 Score:** 90.9% — 2 × (0.833 × 1.0) / (0.833 + 1.0)

### Interpretation

My model has perfect recall (100%) but lower precision (83.3%), meaning it catches every phishing email but occasionally flags a legitimate email as phishing. This is a preferable trade-off for cybersecurity — missing a phishing email (false negative) is far more dangerous than a false alarm (false positive). The one misclassification (test_legit_05.png predicted as Phishing at 97% confidence) shows that confidence scores alone are not a reliable safety net, since the model was confidently wrong. To improve the model, I would add more diverse training images — particularly legitimate emails that visually resemble phishing emails (promotional emails with large buttons, urgent-sounding subject lines) — so the model learns to distinguish these edge cases. Adding a third "Uncertain" class could also help route borderline cases to human review.

---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested

1. **Generic:** distilbert-base-uncased-finetuned-sst-2-english (sentiment analysis — trained on movie reviews)
2. **Fine-Tuned A:** mrm8488/bert-tiny-finetuned-sms-spam-detection — classifies text as spam (LABEL_1) or not spam (LABEL_0), trained on the SMS Spam Collection dataset with 98% validation accuracy
3. **Fine-Tuned B:** SamLowe/roberta-base-go_emotions — detects 28 emotion categories (anger, fear, neutral, approval, etc.), trained on the GoEmotions Reddit dataset with 40,000+ labeled examples

### Results

| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Label (Score) | Best Model |
|-------|-----------------------|----------------------------|----------------------------|------------|
| Unauthorized login from IP traced to Moscow at 3:47 AM | NEGATIVE (0.9961) | LABEL_0 / not spam (0.7415) | neutral (0.9159) | Fine-Tuned B |
| Routine firewall rule update during maintenance | NEGATIVE (0.9986) | LABEL_1 / spam (0.5376) | neutral (0.8776) | Fine-Tuned B |
| Phishing email with spoofed Amazon domain detected | NEGATIVE (0.9959) | LABEL_0 / not spam (0.7045) | neutral (0.9568) | Fine-Tuned B |
| Multiple failed SSH attempts from Beijing — 47 in 5 min | NEGATIVE (0.9994) | LABEL_0 / not spam (0.8388) | neutral (0.5590) | Fine-Tuned B |
| System resource utilization normal — no anomalies | NEGATIVE (0.9880) | LABEL_1 / spam (0.5719) | neutral (0.8523) | Fine-Tuned B |

### Analysis

**Generic model strengths:** The generic sentiment model returned results with very high confidence scores (0.988–0.999), showing it is decisive. However, decisiveness without accuracy is useless.

**Generic model weaknesses:** It labeled every single record as NEGATIVE regardless of content — the same failure observed in Week 4. A routine maintenance update and an active security breach both received the same label. This model was trained on movie review sentiment, so it has no ability to distinguish between threat levels, urgency, or the operational meaning of cybersecurity text. For email triage, this would mean every incoming email gets the same classification, which defeats the purpose of automated sorting.

**Fine-tuned model advantage:** Fine-Tuned B (emotions) was the most consistent and contextually appropriate model. It correctly identified all five inputs as "neutral," which is accurate — these are system-generated technical log entries written in a clinical, factual tone, not emotional human communication. This model would become significantly more useful on actual human-written emails, where it could detect anger in complaints, curiosity in questions, or gratitude in positive feedback, enabling my Component 2 to adjust auto-response tone accordingly.

Fine-Tuned A (spam detection) showed moderate differentiation but with low confidence (0.53–0.84). It classified the routine firewall update and the system-normal status as spam, while classifying the actual phishing alert as not-spam. This inversion happened because the model was trained on SMS spam ("You won a free iPhone!"), which has very different language patterns than cybersecurity alerts.

**Biggest surprise:** The spam detection model classified the phishing email alert (Record 3) as "not spam" with 0.70 confidence. The word "phishing" is literally in the input, yet the model didn't flag it because it was trained on SMS-style spam, not security alert language. This demonstrates that a model's domain training matters more than keyword matching — the model doesn't understand what "phishing" means in a cybersecurity context.

### Recommended Model for My Capstone Component

**Component:** Component 2 — Auto Response Drafting (generates draft replies for classified emails and stores them in Airtable for human review)

**Primary model:** SamLowe/roberta-base-go_emotions — For my component, understanding the emotional tone of an incoming email is directly actionable. An angry complaint email needs an empathetic, apologetic draft response. A neutral informational request needs a straightforward, helpful reply. A curious question needs an encouraging, detailed response. The emotions model provides this signal, enabling the Flowise LLM chain to adjust the tone and content of auto-generated drafts based on how the sender is feeling. In this lab, it correctly identified technical alerts as neutral, and I expect it will produce more varied and useful labels when processing actual human-written emails.

**Confidence threshold:** 0.70 — Based on the test results, the emotions model returned high confidence for its top label (0.56–0.96). Setting a threshold at 0.70 would mean that if the model is less than 70% confident in its emotion classification, the email skips auto-response and goes directly to human review. This balances automation efficiency with safety, ensuring that ambiguous emails (like Record 4 at 0.559) receive human attention.

**Priority metric:** Recall — For an email triage system, missing an important email that needs a response (false negative) is worse than generating an unnecessary draft that gets rejected in review (false positive). High recall ensures that the system catches and drafts responses for as many actionable emails as possible, while the human review queue (NEEDS_REVIEW status in Airtable) serves as the safety net for any drafts that miss the mark.

---

## Limitations & Next Steps

The primary limitation of this lab is domain mismatch. All three text models were tested on cybersecurity system alerts, but my Component 2 will process human-written emails (complaints, questions, requests). The emotions model labeled everything as "neutral" here, which is correct for technical logs but doesn't demonstrate its full potential on actual email content. Testing with realistic email inputs like "I haven't received my refund and it's been two weeks" or "Can you send me the meeting agenda?" would produce more varied and meaningful emotion labels.

If given more time, I would: (1) test all three models on actual email-style text rather than security alerts, since that matches my component's real-world input; (2) fine-tune a custom model specifically for email category classification (question, complaint, request, informational, spam) using labeled email datasets; (3) experiment with combining the emotions model and a spam filter as a two-stage pre-processing pipeline before the Flowise LLM generates a draft; and (4) expand the test dataset to 20 records (using the full set from Appendix A) with edge cases like ambiguous language and very short inputs to stress-test the models.