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

## Status

Project setup and planning are currently in progress.
