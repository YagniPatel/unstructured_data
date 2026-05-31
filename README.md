# Multimodal Phishing Webpage Detection

Detecting phishing webpages using text (RoBERTa), image (ResNet50), and late-fusion multimodal approaches. Fine-tuned on 5,000 labeled Phishpedia webpages with analysis of why multimodal fusion failed to outperform the text-only baseline.

---

## 🧠 Problem Statement

Phishing webpages impersonate legitimate sites to steal user credentials. Existing detection methods rely on either visual similarity or URL/text analysis in isolation. This project investigates whether combining both modalities improves detection — and critically, *why it doesn't*.

---

## 🏗️ Architecture

```
                    ┌──────────────────┐
                    │  Phishpedia      │
                    │  Dataset (5,000) │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼                             ▼
   ┌─────────────────┐           ┌──────────────────┐
   │  HTML Text      │           │  Screenshot       │
   │  Extraction     │           │  Image           │
   └────────┬────────┘           └────────┬─────────┘
            ▼                             ▼
   ┌─────────────────┐           ┌──────────────────┐
   │  RoBERTa        │           │  ResNet50         │
   │  (fine-tuned)   │           │  (fine-tuned)    │
   │  768-d embed    │           │  2048-d embed    │
   └────────┬────────┘           └────────┬─────────┘
            │                             │
            └──────────────┬──────────────┘
                           ▼
                  ┌─────────────────┐
                  │  Late Fusion    │
                  │  (concat + MLP) │
                  └────────┬────────┘
                           ▼
                  ┌─────────────────┐
                  │  Phishing / Safe│
                  └─────────────────┘
```

---

## 📊 Results

| Model | Accuracy | FNR (False Negative Rate) | Notes |
|---|---|---|---|
| **RoBERTa (text-only)** | **97.0%** | **1.55%** | Best overall |
| ResNet50 (image-only) | 94.0% | 5.84% | Weaker on visual similarity |
| Late Fusion (text + image) | ~96.5% | ~2.1% | Did not improve over text baseline |

**Key Finding:** Multimodal fusion did not outperform the text-only model. Identified **modality imbalance** as the primary bottleneck — RoBERTa's 768-d features dominated the combined representation, effectively drowning out ResNet50's 2048-d image features during gradient updates.

---

## 🗂️ Project Structure

```
phishing_webpage_detection_using_multimodal/
├── models/                  # Saved model checkpoints
├── outputs/                 # Evaluation results, confusion matrices, plots
├── utils/                   # Helper functions (data loading, metrics, etc.)
├── prepare_dataset.py       # Dataset preparation and preprocessing
├── train_text.py            # RoBERTa fine-tuning pipeline
├── train_image.py           # ResNet50 fine-tuning pipeline
├── train_multimodal.py      # Late-fusion multimodal training
└── evaluate.py              # Evaluation and metrics computation
```

---

## 📦 Setup

```bash
# Clone the repo
git clone https://github.com/YagniPatel/phishing_webpage_detection_using_multimodal.git
cd phishing_webpage_detection_using_multimodal

# Install dependencies
pip install torch torchvision transformers scikit-learn pandas numpy matplotlib seaborn

# Download Phishpedia dataset
# https://github.com/lindsey98/Phishpedia
# Place dataset in ./data/ directory
```

---

## ▶️ How to Run

```bash
# Step 1: Prepare the dataset
python prepare_dataset.py --data_dir ./data --output_dir ./processed

# Step 2: Train text-only model (RoBERTa)
python train_text.py --data_dir ./processed --epochs 5 --batch_size 16

# Step 3: Train image-only model (ResNet50)
python train_image.py --data_dir ./processed --epochs 10 --batch_size 32

# Step 4: Train multimodal fusion model
python train_multimodal.py --data_dir ./processed --epochs 10 --batch_size 16

# Step 5: Evaluate all models
python evaluate.py --model_dir ./models --output_dir ./outputs
```

---

## 🔍 Key Takeaways

- **HTML text is a stronger signal than visual appearance** for phishing detection. Attackers can clone visual design more easily than they can replicate legitimate HTML/text structure.
- **Late fusion with naive concatenation is sensitive to feature scale imbalances.** Future work should explore attention-based fusion or feature normalization strategies.
- **FNR (False Negative Rate) matters more than accuracy** in security contexts — a missed phishing page is more costly than a false alarm.

---

## 🛠️ Tech Stack

`Python` · `PyTorch` · `HuggingFace Transformers` · `RoBERTa` · `ResNet50` · `Scikit-learn` · `Pandas` · `Matplotlib`

---

## 📚 References

- [Phishpedia Dataset](https://github.com/lindsey98/Phishpedia)
- [RoBERTa: A Robustly Optimized BERT Pretraining Approach](https://arxiv.org/abs/1907.11692)
- [Deep Residual Learning for Image Recognition (ResNet)](https://arxiv.org/abs/1512.03385)
