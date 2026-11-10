# DeepLearningIndustryAIChallenge

Team 3 (Jolie Walker, Cooper Gunnels). Industry AI Challenge: From Pre-trained Model to Business Solution.

**Problem.** Help an online retailer's customer experience team find unhappy customers in large volumes of free-text feedback. A fine-tuned DistilBERT model flags likely-negative feedback so agents read it first.

**Data.** Amazon Reviews Multi (English) from Hugging Face (`goosmanlei/amazon_reviews_multi`). 1 to 2 stars = negative, 4 to 5 stars = positive, 3 stars removed, duplicates removed across splits. 16,000 training, 2,000 validation and 4,000 test reviews, balanced.

**Model.** `distilbert-base-uncased` with a classification head, adapted by transfer learning (frozen encoder, or last 2 layers unfrozen).

## Files

| File | What it is |
|---|---|
| `Amazon_Review_Sentiment_Analysis_Final.ipynb` | **Final run.** Baseline plus three controlled experiments, stability check, threshold tuning, error analysis, final test evaluation and inference timing. |
| `Amazon_Review_Sentiment_Analysis.ipynb` | Initial run (kept as the record of our first attempt). |
| `CHANGES.md` | Every change between the initial and final run, and why. |
| `final_run/` | Tables, figures and `run_config.json` from the final run (executed Nov 9, 2026 on a Colab T4). |
| `experiment_comparison.csv`, `final_test_results.csv`, `inference_speed_results.csv`, `*.png`, `Amazon_Review_Project_Evidence.zip` | Initial run outputs. |

## How to reproduce

1. Open `Amazon_Review_Sentiment_Analysis_Final.ipynb` in Google Colab.
2. Runtime > Change runtime type > T4 GPU.
3. Runtime > Run all (about 40 to 60 minutes). The evidence zip and the trained model zip download at the end.

Library versions and all settings for the run are saved in `final_run/run_config.json`.

The trained model (268 MB) is not stored in this repository because it exceeds GitHub's 100 MB file limit. Running the notebook recreates it and downloads it as `final_run_selected_model.zip`.
