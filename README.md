# AI201 Project 3 — TakeMeter

TakeMeter is a fine-tuned text classification system that categorizes comments from the `r/cscareerquestions` community.

The classifier uses four labels:

* `evidence_based_advice`
* `personal_experience`
* `unsupported_claim`
* `emotional_reaction`

This repository will contain:

* The project planning document
* A labeled dataset of at least 200 comments
* The fine-tuning notebook
* Baseline and fine-tuned model evaluation results
* A confusion matrix
* The final evaluation report

## Repository Structure

```text
.
├── README.md
├── planning.md
├── data/
│   └── labeled_dataset.csv
├── results/
│   ├── evaluation_results.json
│   └── confusion_matrix.png
└── notebook/
    └── takemeter.ipynb
```

## Model

The project fine-tunes `distilbert-base-uncased` and compares it against a zero-shot Groq baseline using `llama-3.3-70b-versatile`.

## Evaluation Report

### Dataset Split

The dataset contained 200 synthetic comments distributed evenly across four labels:

| Label                   | Count |
| ----------------------- | ----: |
| `evidence_based_advice` |    50 |
| `personal_experience`   |    50 |
| `unsupported_claim`     |    50 |
| `emotional_reaction`    |    50 |

The notebook used a stratified 70% / 15% / 15% split:

* Training set: 140 examples
* Validation set: 30 examples
* Test set: 30 examples

The test set contained:

* 8 `evidence_based_advice` examples
* 7 `personal_experience` examples
* 8 `unsupported_claim` examples
* 7 `emotional_reaction` examples

### Training Configuration

The fine-tuned model was `distilbert-base-uncased` with a four-label classification head.

The following hyperparameters were used:

| Hyperparameter        |  Value |
| --------------------- | -----: |
| Epochs                |      3 |
| Learning rate         | `2e-5` |
| Training batch size   |     16 |
| Evaluation batch size |     32 |
| Weight decay          |   0.01 |
| Warmup steps          |     50 |
| Maximum token length  |    256 |

I kept the notebook’s default values because they were designed for small datasets containing approximately 100–500 examples. Three epochs provided enough time for the model to begin learning the distinctions while limiting the risk of overfitting.

Validation accuracy was 23.3% after the first two epochs and increased to 60.0% after the third epoch.

---

### Overall Model Comparison

| Model                   | Test Accuracy |
| ----------------------- | ------------: |
| Zero-shot Groq baseline |     **1.000** |
| Fine-tuned DistilBERT   |     **0.733** |

The zero-shot Groq baseline achieved 100% accuracy, while the fine-tuned DistilBERT model achieved 73.3% accuracy on the same 30-example test set.

Fine-tuning therefore resulted in a 26.7 percentage-point regression compared with the baseline.

A likely reason is that the synthetic comments were written using clear patterns that closely matched the label definitions included in the Groq prompt. Groq could reason directly from the full definitions and examples, while DistilBERT had only 140 training examples from which to learn the distinctions.

The perfect baseline score should also be interpreted cautiously because the test set was small and the synthetic examples had relatively explicit label boundaries.

---

### Fine-Tuned DistilBERT Per-Class Metrics

| Label                   | Precision |   Recall |       F1 | Support |
| ----------------------- | --------: | -------: | -------: | ------: |
| `evidence_based_advice` |      0.80 |     1.00 |     0.89 |       8 |
| `personal_experience`   |      0.75 |     0.86 |     0.80 |       7 |
| `unsupported_claim`     |      0.64 |     0.88 |     0.74 |       8 |
| `emotional_reaction`    |      1.00 |     0.14 |     0.25 |       7 |
| **Macro average**       |  **0.80** | **0.72** | **0.67** |  **30** |
| **Weighted average**    |  **0.79** | **0.73** | **0.68** |  **30** |

The model performed best on `evidence_based_advice`, correctly identifying all eight test examples.

It also performed reasonably well on `personal_experience` and `unsupported_claim`.

The major weakness was `emotional_reaction`. Although its precision was 1.00, its recall was only 0.14. This means the model predicted `emotional_reaction` only once, and that prediction was correct, but it missed six of the seven true examples.

---

### Groq Baseline Per-Class Metrics

| Label                   | Precision |   Recall |       F1 | Support |
| ----------------------- | --------: | -------: | -------: | ------: |
| `evidence_based_advice` |      1.00 |     1.00 |     1.00 |       8 |
| `personal_experience`   |      1.00 |     1.00 |     1.00 |       7 |
| `unsupported_claim`     |      1.00 |     1.00 |     1.00 |       8 |
| `emotional_reaction`    |      1.00 |     1.00 |     1.00 |       7 |
| **Macro average**       |  **1.00** | **1.00** | **1.00** |  **30** |

All 30 Groq responses were parseable, and the baseline classified every test example correctly.

---

### Confusion Matrix

Rows represent the true label, while columns represent the predicted label.

| True \ Predicted          | Evidence-Based Advice | Personal Experience | Unsupported Claim | Emotional Reaction |
| ------------------------- | --------------------: | ------------------: | ----------------: | -----------------: |
| **Evidence-Based Advice** |                     8 |                   0 |                 0 |                  0 |
| **Personal Experience**   |                     0 |                   6 |                 1 |                  0 |
| **Unsupported Claim**     |                     1 |                   0 |                 7 |                  0 |
| **Emotional Reaction**    |                     1 |                   2 |                 3 |                  1 |

The confusion matrix shows a clear systematic pattern: emotional reactions were usually assigned to another category.

Of the seven true emotional reactions:

* 1 was classified correctly
* 1 was classified as `evidence_based_advice`
* 2 were classified as `personal_experience`
* 3 were classified as `unsupported_claim`

This suggests that the model relied heavily on sentence structure and topic words rather than consistently identifying emotional intent.

The generated confusion matrix image is available at:

```text
results/confusion_matrix.png
```

---

### Error Analysis

The fine-tuned model made 8 incorrect predictions out of 30 test examples.

#### Error 1

**Comment:**

> “Today’s interview went better than I expected.”

**True label:** `emotional_reaction`
**Predicted label:** `personal_experience`
**Confidence:** 0.26

This comment refers to the writer’s personal interview, which likely caused the model to associate it with `personal_experience`. However, its main purpose is expressing relief and satisfaction rather than describing a detailed experience.

The model appears to focus on first-person interview language instead of the comment’s emotional purpose.

---

#### Error 2

**Comment:**

> “That final round felt like it lasted all day.”

**True label:** `emotional_reaction`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.26

The exaggeration “lasted all day” is an emotional reaction to a stressful interview. The model may have interpreted the sentence as a literal unsupported statement because it lacked enough examples of exaggeration and figurative language.

This indicates difficulty distinguishing emotional hyperbole from unsupported factual claims.

---

#### Error 3

**Comment:**

> “I failed my first technical interview because I froze on a basic hash map problem.”

**True label:** `personal_experience`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.26

This is a direct account of a personal event and should be classified as `personal_experience`.

The phrase “because I froze” may have been interpreted as a claim or explanation rather than part of a personal story. The low confidence suggests the model did not learn this boundary strongly.

---

#### Error 4

**Comment:**

> “The entire hiring process feels random.”

**True label:** `emotional_reaction`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.28

This example sits near the boundary between an emotional complaint and a broad claim.

The phrase “the entire hiring process” sounds generalized, which likely pushed the model toward `unsupported_claim`. However, the word “feels” indicates that the writer is expressing frustration rather than presenting a factual argument.

---

#### Error 5

**Comment:**

> “I actually enjoyed that interview for once.”

**True label:** `emotional_reaction`
**Predicted label:** `personal_experience`
**Confidence:** 0.26

The model again treated a first-person interview statement as a personal experience, even though the central purpose was expressing a positive emotional reaction.

This supports the pattern that first-person language strongly influenced predictions.

---

#### Error 6

**Comment:**

> “That recruiter call made my entire week.”

**True label:** `emotional_reaction`
**Predicted label:** `evidence_based_advice`
**Confidence:** 0.25

This was the most unexpected mistake. The sentence contains no recommendation or supporting reasoning.

The confidence was close to random for a four-class task, suggesting that the model had not learned a meaningful representation for this short emotional sentence.

---

#### Error 7

**Comment:**

> “I am officially done with LeetCode for today.”

**True label:** `emotional_reaction`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.26

The model may have relied on the topic word “LeetCode,” which appeared frequently in advice and claim examples.

The sentence is actually a short expression of exhaustion. This suggests the model sometimes focused on topic keywords more than communicative intent.

---

#### Error 8

**Comment:**

> “Employers do not care about communication skills as long as you can code.”

**True label:** `unsupported_claim`
**Predicted label:** `evidence_based_advice`
**Confidence:** 0.29

This sentence resembles advice because it discusses employer preferences and technical skills. However, it makes a broad claim without evidence and should be labeled `unsupported_claim`.

The model may have learned that career statements containing phrases such as “skills” and “code” often belong to the advice category.

---

### Systematic Error Pattern

The strongest error pattern involved short emotional comments.

The model often confused `emotional_reaction` with:

* `personal_experience` when the sentence used first-person language
* `unsupported_claim` when the sentence contained exaggeration or a broad complaint
* Other labels when career-related keywords such as “interview,” “recruiter,” or “LeetCode” appeared

This suggests the model learned surface-level correlations such as topic words and pronouns more effectively than the deeper purpose or tone of the comment.

The model also showed low confidence on all eight mistakes, with predicted probabilities between 0.25 and 0.29. Since random confidence in a four-class task is approximately 0.25, these scores show that the model was highly uncertain on its incorrect predictions.

---

### What the Model Learned vs. What I Intended

I intended the model to distinguish comments based on their communicative purpose:

* supported advice
* personal storytelling
* unsupported generalization
* emotional expression

The model successfully learned some of these distinctions. It recognized evidence-based advice especially well and classified most personal experiences and unsupported claims correctly.

However, it did not learn emotional intent reliably. Instead, it often used surface-level signals:

* first-person wording suggested `personal_experience`
* broad wording suggested `unsupported_claim`
* career and technical keywords sometimes suggested `evidence_based_advice`

Therefore, the model learned useful category patterns, but not the full semantic boundaries I intended.

---

### Definition of Success Review

The original success criteria were:

* At least 75% overall accuracy
* At least 0.70 macro F1
* No individual label below 0.60 F1
* Better performance than the Groq baseline

The fine-tuned model achieved:

* 73.3% accuracy
* 0.67 macro F1
* 0.25 F1 for `emotional_reaction`
* Lower performance than the Groq baseline

Therefore, the fine-tuned model did not meet the original definition of success.

It came close to the overall accuracy and macro-F1 targets, but its poor emotional-reaction recall would make it unreliable for real deployment.

---

### Limitations

The main limitations were:

1. The dataset contained only 200 examples.
2. The model trained on only 140 examples.
3. The test set contained only 30 examples.
4. The comments were synthetic rather than collected directly from naturally occurring community discussions.
5. The generated examples had clearer label patterns than real online comments.
6. The label boundary between emotional reactions, personal experiences, and unsupported claims remained difficult.
7. DistilBERT received only the labeled examples, while Groq received complete definitions, decision rules, and representative examples in its prompt.

Because of these limitations, the results should be interpreted as a small experimental comparison rather than evidence that the classifier is ready for real-world use.

---

### Improvements

Future versions could improve performance by:

* Collecting more naturally occurring comments
* Adding more emotional-reaction training examples
* Including sarcasm, humor, exaggeration, and mixed-intent comments
* Reviewing ambiguous labels with a second annotator
* Training for additional epochs while monitoring validation performance
* Testing different learning rates
* Adding weighted loss if one category remains difficult
* Comparing DistilBERT with a stronger encoder model
* Evaluating on a larger held-out test set
* Adding contrastive examples that use the same topic words but belong to different labels

The most important improvement would be adding more difficult examples that force the model to distinguish a comment’s intent rather than relying on keywords.

