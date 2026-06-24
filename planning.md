# TakeMeter Project Plan

## Project Overview

This project will build a text classifier for comments from the `r/cscareerquestions` community. The classifier will categorize comments based on the type and quality of discourse they contain.

The four categories are:

- `evidence_based_advice`
- `personal_experience`
- `unsupported_claim`
- `emotional_reaction`

The goal is to determine whether a comment provides supported advice, shares a personal experience, makes an unsupported claim, or mainly expresses an emotional reaction.

---

## 1. Community

I chose the `r/cscareerquestions` community because it contains active discussions about software engineering careers, internships, resumes, interviews, education, workplace experiences, and the technology job market.

This community is a strong fit for a classification task because the comments vary significantly in purpose and quality. Some users provide detailed and actionable advice, while others share personal stories, make broad claims without evidence, or post short emotional reactions.

These distinctions matter because users looking for career guidance may want to separate useful, supported advice from anecdotes or unsupported opinions.

---

## 2. Labels

### `evidence_based_advice`

A comment that gives actionable career advice supported by specific reasoning, concrete steps, hiring experience, technical context, examples, or other meaningful evidence.

#### Examples

1. “Tailor the top half of your resume to the job description because recruiters often scan quickly for relevant skills and experience.”

2. “For backend internships, build one deployed API project with a database and tests instead of several unfinished projects because it gives interviewers more technical decisions to discuss.”

---

### `personal_experience`

A comment that mainly describes the writer’s own job-search, internship, interview, education, or workplace experience without presenting a broadly supported argument.

#### Examples

1. “I applied to around 150 internships before getting two interviews and one offer.”

2. “My first software job came from a university career fair rather than an online application.”

---

### `unsupported_claim`

A comment that makes a broad, confident, or generalized claim without enough explanation, supporting evidence, or context.

#### Examples

1. “Computer science degrees are useless now because AI will replace junior developers.”

2. “Nobody gets internships without referrals anymore.”

---

### `emotional_reaction`

A comment that mainly expresses excitement, frustration, disappointment, humor, fear, or another immediate emotion without providing substantial advice or analysis.

#### Examples

1. “The job market is completely cooked.”

2. “I finally got an offer! Let’s go!”

---

## 3. Hard Edge Cases

Some comments may fit more than one label. I will use the following decision rules during annotation.

### Advice vs. Personal Experience

A comment may describe a personal experience while also recommending an action.

Example:

> “I stopped grinding LeetCode and focused on projects, and that helped me get my internship.”

This will be labeled `personal_experience` because the main evidence is one person’s individual outcome and the comment does not explain why the strategy should apply more broadly.

A more detailed version would be labeled `evidence_based_advice`:

> “I focused more on deployed projects because most of my startup interviews asked about architecture and implementation decisions rather than difficult algorithm questions.”

This comment provides reasoning that supports the recommendation.

### Unsupported Claim vs. Emotional Reaction

Example:

> “The market is dead.”

This could be interpreted as either a claim or an emotional reaction.

It will be labeled `emotional_reaction` when it is mainly a short expression of frustration.

A longer statement such as:

> “The software job market is dead because companies no longer hire junior engineers.”

will be labeled `unsupported_claim` because it presents a broad conclusion without enough supporting evidence.

### Evidence-Based Advice vs. Unsupported Claim

Example:

> “You need referrals to get interviews because online applications are useless.”

This will be labeled `unsupported_claim` because it gives a recommendation but does not provide enough evidence or reasoning.

A comment will only be labeled `evidence_based_advice` when it includes meaningful explanation, concrete steps, or relevant context.

### Difficult Cases During Annotation

While labeling the dataset, I will record at least three comments that were genuinely difficult to classify. For each one, I will document:

- The original comment
- The two labels I considered
- The final label I selected
- The decision rule I used

This section will be updated after annotation with real examples from the dataset.

---

## 4. Data Collection Plan

I will collect at least 200 public comments from `r/cscareerquestions`.

The comments will come from public discussions about topics such as:

- Internship searches
- Resume advice
- Technical interviews
- LeetCode preparation
- Software engineering careers
- Computer science degrees
- Layoffs and the job market
- Workplace and internship experiences

I will save the dataset as a CSV file with the following columns:

- `text`: the full comment
- `label`: one of the four label names
- `notes`: optional notes for ambiguous or difficult examples

I will aim for approximately 50 examples per label:

| Label | Target Count |
|---|---:|
| `evidence_based_advice` | 50 |
| `personal_experience` | 50 |
| `unsupported_claim` | 50 |
| `emotional_reaction` | 50 |
| **Total** | **200** |

I will read and review every comment before assigning its final label.

If one category is underrepresented, I will search discussions that are more likely to contain that type of comment. For example:

- For `evidence_based_advice`, I will search resume, interview, and internship advice threads.
- For `personal_experience`, I will search offer, rejection, internship, and career-change threads.
- For `unsupported_claim`, I will search debates about AI, degrees, hiring, layoffs, and the job market.
- For `emotional_reaction`, I will search rejection, offer, frustration, and celebration threads.

No category should make up more than 70% of the dataset. I will aim to keep every category close to 25%.

The notebook will later divide the complete dataset into:

- 70% training data
- 15% validation data
- 15% test data

---

## 5. Evaluation Metrics

I will evaluate both the fine-tuned DistilBERT model and the zero-shot Groq baseline on the same test set.

### Overall Accuracy

Accuracy will measure the percentage of test examples classified correctly.

Accuracy is useful for showing overall model performance, but it is not sufficient by itself because a model could perform well overall while struggling with one particular label.

### Precision

Precision will measure how often the model is correct when it predicts a specific label.

This is important because the model may overuse a label such as `evidence_based_advice` or `unsupported_claim`.

### Recall

Recall will measure how many of the actual examples from each label the model correctly identifies.

This is important for determining whether the model consistently misses a particular category.

### F1 Score

F1 score combines precision and recall.

I will report the F1 score for every label and the macro-average F1 score. Macro F1 is especially useful because it gives equal importance to every category, even if the label counts are not perfectly balanced.

### Confusion Matrix

The confusion matrix will show which labels are most frequently confused.

I expect the most difficult boundaries to be:

- `evidence_based_advice` vs. `personal_experience`
- `unsupported_claim` vs. `emotional_reaction`

### Error Analysis

I will analyze at least three incorrect predictions in detail.

For each incorrect prediction, I will report:

- The original comment
- The true label
- The predicted label
- The model’s confidence, when available
- Why the model may have made the mistake
- Whether the error came from ambiguity, limited context, sarcasm, annotation inconsistency, or another pattern

---

## 6. Definition of Success

The classifier will be considered successful if the fine-tuned model meets the following criteria:

- Overall test accuracy of at least 75%
- Macro F1 score of at least 0.70
- No individual label with an F1 score below 0.60
- Better performance than the zero-shot Groq baseline on either accuracy or macro F1
- A confusion matrix that does not show the model predicting one label for most examples

For a real community tool, I would prefer:

- At least 80% overall accuracy
- At least 0.75 macro F1
- At least 0.70 F1 for every individual label

Even if the model does not reach these thresholds, the project can still be useful if the evaluation clearly identifies why it struggled and what data or label changes would improve it.

---

## AI Tool Plan

### Label Stress-Testing

I will use an AI assistant to generate example comments that sit near the boundaries between:

- `evidence_based_advice` and `personal_experience`
- `unsupported_claim` and `emotional_reaction`
- `evidence_based_advice` and `unsupported_claim`

I will manually classify these examples using my decision rules.

If several generated examples cannot be classified consistently, I will revise the label definitions before completing the dataset.

### Annotation Assistance

I may use an AI assistant to suggest preliminary labels for some collected comments.

However, I will manually review every example and make the final labeling decision myself.

Any AI-generated preliminary labels will only be used as suggestions and will not be accepted without review.

If I use AI-assisted pre-labeling, I will disclose it in the README’s AI usage section.

### Failure Analysis

After running the fine-tuned model and the Groq baseline, I will provide the incorrect predictions to an AI assistant and ask it to identify possible patterns.

Possible patterns may include:

- Short comments being misclassified
- Sarcasm being misunderstood
- Personal stories being mistaken for advice
- Emotional statements being mistaken for unsupported claims
- Keywords such as “resume,” “referral,” or “LeetCode” influencing predictions too strongly

I will verify every proposed pattern by reviewing the actual model errors before including it in the final evaluation.

---

## Planned Stretch Feature

If time allows, I will complete the error pattern analysis stretch feature.

I will group incorrect predictions by possible causes and identify whether the model makes systematic errors rather than only isolated mistakes.

I may also build a simple interface that accepts a new comment and displays:

- The predicted label
- The confidence score
- A short definition of the predicted category
