# TakeMeter Planning Document

## Project Title

**TakeMeter: World Cup 2026 Fan Discourse Classifier**

## Community

For this project, I chose online fan discussion around the **FIFA World Cup 2026**, especially public Reddit communities such as `r/worldcup`, `r/soccer`, and team-specific match threads. This community is a good fit for a classification task because World Cup discussion is extremely active, emotional, and varied. During a major tournament, fans post tactical analysis, bold predictions, instant reactions, complaints about referees, comments about travel and host cities, and broader discussion about the tournament format.

This discourse is interesting because not every comment has the same purpose or quality. Some posts explain what is happening in a match using tactics or evidence, while others are mainly emotional reactions or unsupported claims. A classifier could help separate more substantive discussion from quick reactions and hot takes.

## Label Taxonomy

The classifier will use four mutually exclusive labels. Each post or comment should receive exactly one label.

### 1. `match_analysis`

A comment should be labeled `match_analysis` when it explains something happening on the field using tactics, player roles, match events, statistics, formations, substitutions, or specific soccer reasoning.

**Example 1:**
> The US midfield looked better because Adams kept cutting off Paraguay's central passing lane and forced them to build attacks wide.

**Example 2:**
> Brazil improved after the winger started drifting inside because it opened space for the fullback overlap on the left side.

### 2. `tournament_context`

A comment should be labeled `tournament_context` when it focuses on the broader World Cup environment rather than one specific piece of match play. This includes the tournament format, scheduling, host cities, travel, tickets, group standings, qualification scenarios, broadcasting, or fan experience.

**Example 1:**
> The 48-team format makes the third-place race confusing because teams in later groups will know exactly what result they need.

**Example 2:**
> Playing across Canada, Mexico, and the United States is exciting, but the travel distance between host cities could really affect teams with short rest.

### 3. `unsupported_hot_take`

A comment should be labeled `unsupported_hot_take` when it makes a bold opinion, prediction, criticism, or claim with little or no supporting evidence.

**Example 1:**
> Argentina are finished. No way they survive the knockout rounds.

**Example 2:**
> This USA team is completely overrated and only looks good against weak teams.

### 4. `emotional_reaction`

A comment should be labeled `emotional_reaction` when its main purpose is hype, frustration, humor, celebration, anger, or a short live reaction rather than explanation.

**Example 1:**
> WHAT A GOALLLLLL!!!

**Example 2:**
> This ref is killing me.

## Hard Edge Cases

The hardest edge cases will be comments that mix a strong opinion with some soccer reasoning. For example:

> The US is overrated. They only looked good because Paraguay gave them way too much space in midfield.

This could be interpreted as `unsupported_hot_take` because it begins with a broad claim that the US is overrated. However, it also gives a soccer-specific explanation: Paraguay allowed too much space in midfield. My decision rule is that if a comment gives a specific, understandable soccer reason that explains the claim, I will label it `match_analysis`, even if the tone is aggressive. If the comment only says a team is bad, overrated, lucky, finished, or fraudulent without a clear reason, I will label it `unsupported_hot_take`.

Another difficult boundary is between `tournament_context` and `emotional_reaction`. For example:

> These ticket prices are insane.

This mentions a tournament-related topic, but it is mostly an emotional complaint. I will label this as `emotional_reaction` unless the comment explains the issue with specific context, such as comparing prices, describing travel problems, or discussing fan access. If the comment gives broader reasoning about pricing, host cities, or fan experience, I will label it `tournament_context`.

A third difficult boundary is between `match_analysis` and `emotional_reaction`. For example:

> That substitution changed everything.

This refers to a tactical event, but it does not explain why the substitution mattered. I will label this as `emotional_reaction` unless the comment explains the tactical effect of the substitution. If the comment says something like “That substitution changed everything because it gave them another runner behind the defense,” then it becomes `match_analysis`.

## Data Collection Plan

I will collect at least **200 public posts or comments** from World Cup 2026 discussions. My main sources will be public Reddit threads such as `r/worldcup`, `r/soccer`, team-specific subreddits, match threads, post-match threads, and tournament discussion threads. I will only use public comments and will not collect private messages, private Discord content, or anything behind authentication.

My target label distribution is approximately balanced:

| Label | Target Count |
|---|---:|
| `match_analysis` | 50 |
| `tournament_context` | 50 |
| `unsupported_hot_take` | 50 |
| `emotional_reaction` | 50 |
| **Total** | **200** |

I expect `emotional_reaction` and `unsupported_hot_take` to be easier to find in live match threads. I expect `match_analysis` and `tournament_context` to be easier to find in post-match threads, lineup threads, group-stage discussion threads, and tournament news discussions.

If one label is underrepresented after collecting 200 examples, I will collect additional examples from thread types where that label is more common. For example, if I do not have enough `match_analysis`, I will look more closely at post-match threads and tactical discussion comments. If I do not have enough `tournament_context`, I will collect from threads about host cities, travel, group standings, match scheduling, and tournament format.

The dataset will be saved as one CSV file with at least these columns:

| Column | Description |
|---|---|
| `text` | The World Cup comment or post text |
| `label` | One of the four label names |
| `notes` | Optional notes for difficult or borderline examples |

The notebook will split the single CSV into train, validation, and test sets using a 70% / 15% / 15% split.

## Evaluation Metrics

I will evaluate the classifier using more than accuracy because this is a multi-class subjective classification task. Accuracy is useful as a general summary, but it can hide problems if the model performs well on common labels and poorly on harder labels.

The main metrics will be:

1. **Overall accuracy**: This shows the percentage of test examples the model classifies correctly.
2. **Per-class precision**: This shows whether the model over-predicts a label. For example, if many emotional comments are incorrectly predicted as `unsupported_hot_take`, precision for `unsupported_hot_take` will suffer.
3. **Per-class recall**: This shows whether the model misses a label. For example, if the model fails to identify many real `match_analysis` examples, recall for `match_analysis` will be low.
4. **Per-class F1 score**: This balances precision and recall and is especially useful for comparing performance across labels.
5. **Confusion matrix**: This will show which labels the model confuses most often, such as `match_analysis` vs. `unsupported_hot_take` or `tournament_context` vs. `emotional_reaction`.

These metrics are appropriate because the goal is not just to get many examples right overall. The goal is to understand whether the model can separate meaningful discourse categories that are easy for humans to confuse.

## Definition of Success

A successful classifier should perform better than a simple zero-shot LLM baseline and should be reliable enough to identify broad discourse patterns in World Cup fan discussions.

My target success criteria are:

- Fine-tuned model overall accuracy of at least **75%** on the test set.
- Fine-tuned model beats the Groq zero-shot baseline by at least **5 percentage points** in overall accuracy.
- Each label has an F1 score of at least **0.65**.
- No label has recall below **0.60**, because that would mean the model is mostly missing that category.
- The confusion matrix should show understandable mistakes, not random failure across all labels.

For a real community tool, I would consider the classifier “good enough” if it reached around **75–80% accuracy** and had no label with very poor recall. I would not use it to automatically remove or punish comments. Instead, I would use it as a soft sorting or analysis tool, such as helping moderators or researchers understand how much discussion is tactical analysis, emotional reaction, hot take, or tournament-context discussion.

## AI Tool Plan

### Label Stress-Testing

Before annotating the full dataset, I will use an AI tool to stress-test my label definitions. I will give the AI my four labels, definitions, and edge-case rules, then ask it to generate 5–10 borderline World Cup comments. The purpose is to see whether my labels are precise enough to classify difficult examples.

If the AI generates examples that I cannot label consistently, I will revise the label definitions before collecting the full dataset. For example, if many generated comments fall between `match_analysis` and `unsupported_hot_take`, I will make the evidence requirement clearer.

### Annotation Assistance

I may use an LLM to pre-label some examples, but I will manually review every label myself. If I use pre-labeling, I will track it in the `notes` column with a phrase such as `LLM pre-labeled, human reviewed`. I will not accept labels automatically without reading the comment.

My final labels will be my own reviewed labels, not purely AI-generated labels. I will disclose this process in the README if I use it.

### Failure Analysis

After evaluating the fine-tuned model, I will collect at least three wrong predictions and analyze why the model made those mistakes. I will also use an AI tool to look for patterns in the wrong predictions, such as whether the model struggles with sarcasm, short comments, aggressive analysis, or comments that mix tournament context with emotion.

I will verify any AI-suggested pattern myself by checking the actual examples and the confusion matrix. The AI tool will help generate hypotheses, but I will decide which patterns are actually supported by the model's errors.

## Current Milestone 2 Checkpoint

This planning document addresses the required Milestone 2 questions: community, labels, hard edge cases, data collection plan, evaluation metrics, definition of success, and AI Tool Plan. The label definitions are specific enough that two annotators should agree on most examples, and the success criteria are measurable rather than vague.
