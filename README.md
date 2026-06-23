# TakeMeter: World Cup 2026 Fan Discourse Classifier

## Project Overview

TakeMeter is a fine-tuned text classifier that evaluates discourse quality in World Cup 2026 online discussion. The project focuses on public soccer fan comments from communities such as Reddit match threads, post-match threads, lineup debates, referee discussions, and tournament-format discussions.

The goal is not to classify whether a comment is positive or negative. Instead, the goal is to classify what kind of discourse the comment represents:

- tactical match analysis
- broader tournament context
- unsupported hot take
- emotional reaction

## Current Dataset Status

After collecting real comments, the final dataset should be saved as:

```text
worldcup2026_takemeter_labeled_dataset.csv
```

The file should contain at least these columns:

| Column | Meaning |
|---|---|
| `text` | The public comment text |
| `label` | One of the four TakeMeter labels |
| `notes` | Annotation notes, especially for difficult cases |

## Community

The target community is World Cup 2026 soccer discussion, especially public comments from `r/soccer`, `r/worldcup`, team-specific soccer subreddits, public match threads, public post-match threads, and public tournament-format or host-city discussion threads.

This community is a good fit because soccer discourse is varied. Some comments explain tactics, some discuss travel or tournament format, some make extreme predictions, and some are just live emotional reactions.

## Label Taxonomy

The classifier uses four mutually exclusive labels.

### `match_analysis`

A comment that explains something happening on the field using tactics, player roles, stats, match events, formations, substitutions, or specific soccer reasoning.

Examples:

- "The US midfield is winning because Adams is cutting off Paraguay's central passing lane, forcing them wide every possession."
- "Brazil looked better after switching the winger inside because it created space for the fullback overlap."

### `tournament_context`

A comment that discusses the broader World Cup context, such as format, scheduling, host cities, travel, tickets, group standings, qualification, or tournament structure.

Examples:

- "The 48-team format makes third-place qualification confusing because teams in later groups know exactly what result they need."
- "Playing across Canada, Mexico, and the US is great for exposure, but the travel gap between some host cities could hurt teams with short rest."

### `unsupported_hot_take`

A bold, confident opinion or prediction stated with little or no supporting evidence.

Examples:

- "Argentina are finished. No way they make it past the Round of 32."
- "This USA team is completely overrated. They only beat weak teams."

### `emotional_reaction`

A short emotional response, joke, celebration, frustration, meme-like comment, or instant reaction with little argument.

Examples:

- "WHAT A GOALLLLLL!!!"
- "This ref is killing me."

## Hard Edge Cases and Decision Rules

### Edge Case 1: Aggressive analysis vs. hot take

Example:

> "The US is overrated. They only looked good because Paraguay gave them too much space in midfield."

This could look like `unsupported_hot_take` because it starts with a broad claim. However, it gives a specific soccer reason about midfield space. I label this as `match_analysis` because the comment includes an on-field explanation that supports the claim.

**Decision rule:** If a comment gives a specific soccer reason that explains the claim, label it `match_analysis`, even if the tone is aggressive. If it only says a team is bad, overrated, lucky, finished, or fraudulent without a clear reason, label it `unsupported_hot_take`.

### Edge Case 2: Tournament reasoning vs. match analysis

Example:

> "The new structure means finishing first in the group may be much more valuable because it can produce a cleaner knockout path."

This sounds analytical, but it is not about tactics, formations, or players. It is about tournament structure, so I label it `tournament_context`.

**Decision rule:** If the reasoning is about format, scheduling, travel, standings, qualification, host cities, or fan experience, label it `tournament_context`. If the reasoning is about on-field play, label it `match_analysis`.

### Edge Case 3: Match event vs. emotional reaction

Example:

> "That substitution changed everything."

This mentions a match event, but it does not explain why the substitution mattered. I label it `emotional_reaction`.

**Decision rule:** Mentioning a goal, substitution, red card, or save is not enough for `match_analysis`. The comment needs to explain the tactical or performance effect.

## Data Collection and Annotation Process

For the final real-comment version, I will collect at least 200 public comments from World Cup 2026 discussion threads. I will not use private messages, private Discord servers, or content behind authentication.

Target distribution:

| Label | Target Count |
|---|---:|
| `match_analysis` | 50 |
| `tournament_context` | 50 |
| `unsupported_hot_take` | 50 |
| `emotional_reaction` | 50 |
| **Total** | **200** |

The annotation process:

1. Collect public comments from match, post-match, and tournament discussion threads.
2. Remove deleted, removed, empty, or extremely short unusable comments.
3. Label each comment manually using the taxonomy above.
4. Use the `notes` column for borderline examples.
5. Check label distribution after labeling.
6. If one label is underrepresented, collect from thread types where that label is more common.
7. Save one final CSV file instead of pre-splitting the dataset.

## Model and Training Setup

I fine-tuned `distilbert-base-uncased` using the TakeMeter dataset. The notebook split the data into train, validation, and test sets using a 70% / 15% / 15% split.

Training setup:

| Hyperparameter | Value |
|---|---:|
| Base model | `distilbert-base-uncased` |
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |
| Test set size | 30 |

I used the default learning rate of `2e-5` because it is a common stable starting point for fine-tuning transformer classifiers on small text datasets. I also kept the default 3 epochs to reduce overfitting risk on only 200 examples.

## Baseline Comparison

The zero-shot baseline used Groq's `llama-3.3-70b-versatile`. The prompt included the four label definitions and instructed the model to output only one label.

| Model | Accuracy |
|---|---:|
| Zero-shot Groq baseline | 1.000 |
| Fine-tuned DistilBERT | 0.733 |
| Difference | -0.267 |

The Groq baseline performed better than the fine-tuned DistilBERT model on this test set. This means fine-tuning did not improve over the zero-shot baseline in this version of the project.

## Evaluation Report

### Overall Results

The fine-tuned DistilBERT model correctly classified 22 out of 30 test examples.

| Metric | Value |
|---|---:|
| Test examples | 30 |
| Correct predictions | 22 |
| Wrong predictions | 8 |
| Fine-tuned accuracy | 0.733 |
| Baseline accuracy | 1.000 |

The result shows that the fine-tuned model learned some useful label patterns, but it did not beat the larger zero-shot LLM baseline.

### Per-Class Metrics

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `match_analysis` | 0.571 | 1.000 | 0.727 | 8 |
| `tournament_context` | 0.800 | 0.571 | 0.667 | 7 |
| `unsupported_hot_take` | 0.857 | 0.750 | 0.800 | 8 |
| `emotional_reaction` | 1.000 | 0.571 | 0.727 | 7 |

The strongest class was `unsupported_hot_take`, with an F1 score of 0.800. The weakest class was `tournament_context`, with an F1 score of 0.667. The model had perfect recall for `match_analysis`, but its precision was only 0.571, meaning it over-predicted that label.

### Confusion Matrix

Rows are true labels and columns are predicted labels.

| True label \ Predicted label | `match_analysis` | `tournament_context` | `unsupported_hot_take` | `emotional_reaction` |
|---|---:|---:|---:|---:|
| `match_analysis` | 8 | 0 | 0 | 0 |
| `tournament_context` | 3 | 4 | 0 | 0 |
| `unsupported_hot_take` | 1 | 1 | 6 | 0 |
| `emotional_reaction` | 2 | 0 | 1 | 4 |

The main pattern is that the model over-predicted `match_analysis`. Several `tournament_context` and `emotional_reaction` examples were incorrectly pulled into `match_analysis`.

### Wrong Prediction Examples

| Text | True Label | Predicted Label | Why it was wrong |
|---|---|---|---|
| "The new structure means finishing first in the group may be much more valuable because it can produce a cleaner knockout path." | `tournament_context` | `match_analysis` | The model recognized causal reasoning, but the reasoning is about tournament structure, not tactics. |
| "This game has everything." | `emotional_reaction` | `match_analysis` | The model focused on the word "game," but the sentence is just a short live reaction. |
| "Scotland always find a way to disappoint." | `unsupported_hot_take` | `match_analysis` | The model may have reacted to the team reference, but the comment gives no evidence or tactical explanation. |
| "The group-stage schedule is brutal for fans in some time zones, especially when late games finish after midnight on the East Coast." | `tournament_context` | `match_analysis` | The comment discusses scheduling and fan experience, not on-field play. |
| "That goal deserved a better camera angle." | `emotional_reaction` | `match_analysis` | The comment mentions a goal but reacts to broadcast presentation rather than soccer tactics. |

### Error Pattern Analysis

The clearest error pattern is that the fine-tuned DistilBERT model over-predicted `match_analysis`.

The model seemed to associate soccer words like "game," "goal," "group," "advantage," and "structure" with analysis, even when the comment was actually about tournament context or emotional reaction. This suggests the model learned shallow keyword cues instead of fully learning the intended distinction between on-field tactical reasoning and broader soccer discussion.

The model also struggled with short emotional comments. Examples such as "This game has everything" are easy for a human to recognize as live reactions, but the model had too little context and predicted `match_analysis`.

Finally, all wrong predictions had low confidence scores between 0.26 and 0.29. Since there are four labels, random confidence would be around 0.25. This means the model was uncertain when it made mistakes. In a deployed tool, low-confidence predictions should be flagged for human review.

### Sample Classifications

The table below shows five example posts run through the fine-tuned DistilBERT model.

| Post | True Label | Predicted Label | Confidence | Result |
|---|---|---|---:|---|
| The Netherlands created more after halftime because Japan's midfield stopped tracking the late runner at the top of the box. | `match_analysis` | `match_analysis` | 0.40 | Correct |
| Morocco versus Haiti could be decided by whether Haiti can stop the early pass into the left winger's feet. | `match_analysis` | `match_analysis` | 0.39 | Correct |
| This tournament already clears 2022 and it is not close. | `unsupported_hot_take` | `tournament_context` | 0.28 | Wrong |
| Football is ridiculous. | `emotional_reaction` | `unsupported_hot_take` | 0.26 | Wrong |
| The new structure means finishing first in the group may be much more valuable because it can produce a cleaner knockout path. | `tournament_context` | `match_analysis` | 0.28 | Wrong |

One reasonable correct prediction is: "The Netherlands created more after halftime because Japan's midfield stopped tracking the late runner at the top of the box." The model predicted `match_analysis` with confidence 0.40. This is reasonable because the comment gives a specific on-field explanation about midfield tracking and chance creation, which matches the definition of `match_analysis`.

### Reflection: What the Model Learned vs. What I Intended

I intended the model to learn a discourse-quality taxonomy: separate tactical match analysis, tournament context, unsupported hot takes, and emotional reactions. The model learned part of this structure. It correctly identified all true `match_analysis` examples and performed reasonably well on `unsupported_hot_take`.

However, the model also learned shallow shortcuts. It sometimes treated any soccer-related wording as `match_analysis`, even when the comment was about scheduling, host logistics, or a quick emotional response. This is different from the intended task, where `match_analysis` should require a specific explanation about on-field play.

The fine-tuned model was useful but not better than the Groq baseline. A likely reason is that the dataset is small, while Groq is a much stronger general-purpose language model that can follow detailed label definitions zero-shot. With more real comments, more ambiguous examples, and more balanced short comments, DistilBERT would likely learn the boundaries more reliably.

## Definition of Success

A useful deployed version of TakeMeter should reach:

- at least 0.75 overall accuracy
- at least 0.70 F1 for each class
- no class with near-zero recall
- interpretable confusion patterns
- reasonable behavior on ambiguous comments

The fine-tuned model reached 0.733 accuracy, which is close to the target but below the success threshold. It also had one class below 0.70 F1. I would not deploy this version without collecting more real public comments and retraining.

## How to Run

1. Open the Colab notebook.
2. Upload `worldcup2026_takemeter_labeled_dataset.csv`.
3. Confirm the label map:

```python
LABEL_MAP = {
    "match_analysis": 0,
    "tournament_context": 1,
    "unsupported_hot_take": 2,
    "emotional_reaction": 3,
}
```

4. Run the data split and tokenization sections.
5. Run the Groq baseline section.
6. Run the DistilBERT fine-tuning section.
7. Run the evaluation/export section.

## Files

| File | Purpose |
|---|---|
| `planning.md` | Project planning document |
| `worldcup2026_takemeter_labeled_dataset.csv` | Final labeled dataset |
| `evaluation_results.json` | Baseline vs. fine-tuned accuracy |
| `confusion_matrix.png` | Confusion matrix image |
| `baseline_predictions.csv` | Groq baseline predictions |


## Final Conclusion

TakeMeter successfully demonstrates the full fine-tuning workflow: label design, dataset creation, baseline comparison, model training, evaluation, and error analysis. The most important finding is that the smaller fine-tuned model did not beat the zero-shot Groq baseline and often over-predicted `match_analysis`.

The next improvement is to replace the synthetic seed examples with real public World Cup comments, manually label them, rerun the notebook, and update the final metrics.
