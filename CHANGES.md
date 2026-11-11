# Changes from the initial run to the final run

The initial run (`Amazon_Review_Sentiment_Analysis.ipynb`, committed Nov 8) is kept unchanged as the record of our first attempt. The final run is `Amazon_Review_Sentiment_Analysis_Final.ipynb`. This file explains every change and why we made it.

## What stays the same, on purpose

The dataset, the cleaning and deduplication, the balanced 16,000 / 2,000 sample (same sampling seed), the full 4,000-review test set, DistilBERT, the four run definitions (baseline, unfreeze last 2 layers, learning rate 5e-5, 256 tokens), the batch size, epochs, base learning rate and seed 42. Keeping these fixed means the final run is still the same controlled study, and its results can be compared directly with the initial run.

## Changes

| # | Problem in the initial run | Change in the final run | Why | Notebook section |
|---|---|---|---|---|
| 1 | Every run kept its **last** epoch. Experiment 1 was overfitting: training loss fell from 0.30 to 0.15 while validation loss rose from 0.306 to 0.323. | Checkpoints are saved every epoch and the epoch with the **lowest validation loss** is kept (`load_best_model_at_end`), for all runs equally. | Model selection should use validation data, and an overfit last epoch is not the model you would deploy. Applying the rule to every run keeps the comparison fair. | 5 |
| 2 | Checkpoint choice by F1 would be unstable. | Validation **loss**, not F1, decides the kept epoch. | F1 at the fixed 0.5 threshold swung by several points between epochs (baseline recall 0.845, 0.919, 0.870) mostly because the decision boundary drifted. Loss measures the probabilities themselves; the threshold is tuned separately (change 4). | 5 |
| 3 | Results had no uncertainty. With 2,000 validation reviews, F1 is only accurate to about plus or minus 1.5 points, so Experiments 2 and 3 (-0.3 and +0.5 points) could not be told apart from the baseline. | 95% bootstrap confidence intervals for every run, and a **paired** bootstrap of each experiment against the baseline with a verdict (better, worse, not distinguishable). | Lets the report say which differences are real instead of ranking runs by noise. | 4, 5.2 |
| 4 | One seed per run, so "stability" (named in the recommendation criteria) was not measured. | The baseline and the selected configuration are each retrained with 2 extra seeds; mean and standard deviation are reported. | Shows whether the selected run's advantage is larger than the run-to-run variation from training randomness. Limited to the two runs that matter for the decision to keep compute time reasonable (about 15 extra minutes). | 6 |
| 5 | The 0.5 decision threshold had no connection to the business goal. | A recall target (`TARGET_NEG_RECALL = 0.90`) is set before the run, and the threshold is chosen on **validation** data as the highest one that meets it. A threshold table and precision-recall plot show the trade-off, including **flag rate** (agent workload). | The client cares about catching unhappy customers at an acceptable review workload. Tuning the threshold turns a model score into an operating point we can defend. | 7 |
| 6 | The test set was evaluated before the protocol was final. | The test set is evaluated **once**, at the end, after the configuration, epoch and threshold are all fixed from validation data. The majority benchmark is reported on test as well. | Keeps the test result an honest estimate. Note for the report: the team did see the initial run's test score (0.917 F1), so we should disclose that the final protocol was set after that. | 10 |
| 7 | Inference speed was measured on the GPU only, for one batch of 100, without tokenization. | End-to-end timing (tokenization plus model) on **GPU and CPU**, for single reviews (latency) and batches of 32 (throughput), plus model size on disk and parameter count. | The stated operational constraint is running on commonly available hardware, which means CPU. Tokenization is part of every real prediction. | 11 |
| 8 | Experiment 3's small effect had no explanation. | Token-length distribution of the reviews, with the share over 128 and 256 tokens and a histogram. | If few reviews exceed 128 tokens, a 256-token limit cannot help much. This turns a flat result into a finding. | 3.1 |
| 9 | Error analysis printed the first 3 errors of each type. | Every validation error is exported (`validation_errors.csv`), with error rate by star rating and by length, the share of confident errors, and seeded random samples of 5 per type. The CSV has a blank `manual_category` column for hand labeling. | Patterns (for example, most errors on 2 and 4 star reviews, or confident errors suggesting label noise) are stronger evidence than three hand-picked examples, and the rubric asks for reasons and consequences. | 8 |
| 10 | The model was never shown mixed feedback. | The selected model scores the 1,000 validation 3-star reviews and reports how many would be flagged at the chosen threshold. | Real CX feedback includes neutral and mixed messages. This estimates the extra agent workload and is a concrete limitation for the recommendation. | 9 |
| 11 | The experiment log was missing required columns. | The log now has: change made, reason, expected result, trainable parameters, kept epoch, validation loss, metrics, F1 confidence interval, training time, practical trade-off, and the comparison against the baseline. | Matches the experiment log the assignment asks for. | 5.2 |
| 12 | No narration, no record of settings or library versions. | Markdown sections throughout, one configuration cell, and `run_config.json` with all settings, the selected run, the threshold and library versions. A README explains the repository. | Reproducibility, and any team member can follow the notebook end to end. | 2, 12 |
| 13 | The 1,000-review speed test was a planning step, not a result. | Removed from the final notebook (it remains in the initial run). Each real run records its own training time. | It already did its job of confirming the sample size fits the 30-minute limit. | 3 |
| 14 | Final outputs would overwrite the initial run's files. | All final outputs are written to `final_run/` and downloaded as `final_run_evidence.zip`; the model as `final_run_selected_model.zip`. | Keeps the initial and final evidence separate in the repository. | 12 |

## What to expect when comparing with the initial run

- Experiment 1's validation F1 may come out slightly **lower** than 0.901, because the kept epoch may now be an earlier one with lower validation loss. That is expected and more defensible.
- Experiment 2 should still be the clearest **unsuccessful experiment**: with the encoder frozen, only the small classifier head trains, so a lower learning rate mainly slows convergence. Its training loss was still falling at epoch 3 in the initial run.

## Decisions the team still owns

1. **Confirm `TARGET_NEG_RECALL` (0.90) before running.** It must be a business decision made before seeing results.
2. **Check `SELECTED_RUN`** after Section 5. It is set to Experiment 1. If the results change that choice, edit it and re-run from Section 6 down.
3. Fill in the three "team to complete" markdown cells (experiment interpretation, error interpretation, recommendation).
4. Hand-label about 20 rows of `manual_category` in `validation_errors.csv` for the error analysis in the report.


** Other Notes I have
Alternative model: RoBERTa-base. It would likely be a point or two more accurate, but it is about twice the size and cost per prediction, and our errors come mostly from noisy labels and mixed sentiment, which a bigger model wouldn’t fix.
Why DistilBERT: it keeps most of BERT’s accuracy while being much smaller and faster, which fits the CPU constraint.
Why CPU is the constraint: a support team usually has no GPUs, message volume is thousands per day with no real-time need, and customer data stays in-house. The run shows about 42,000 reviews an hour on a single CPU thread.
The 0.90 recall target was set before the final run, based on missed unhappy customers costing more than false flags.
The unsuccessful experiment for section 6 is Experiment 2.
Also probably worth mentioning that we used pytorch because thats the easiest to find a compatible pretrained model.

Also in the current draft of the report it mentions selecting experiment 1 because it had the highest validation accuracy but the rubric says not to choose experiments based on that so I can think of ways to reqord that.