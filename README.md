# TakeMeter — r/NBA Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in r/NBA posts, distinguishing between analytical arguments, hot takes, and emotional reactions.

---

## Community Choice

I chose **r/NBA** because it's one of the most text-heavy sports communities on Reddit, with hundreds of posts daily that vary enormously in substance. Some users cite advanced stats and historical comparisons; others post pure emotional reactions to games; many just assert bold opinions with no backing. That variance makes it a strong fit for a classification task — the distinctions are real and meaningful to people who actually participate in the community.

---

## Label Taxonomy

| Label | Definition |
|-------|-----------|
| **analysis** | The post makes a structured argument supported by specific statistics, historical comparisons, or tactical observations. Evidence is verifiable and central to the claim. |
| **hot_take** | A bold, confident opinion stated without supporting evidence. The claim may or may not be true, but the post asserts rather than argues. |
| **reaction** | An immediate emotional response to a specific game or event. Little to no argument — the post is expressing a feeling in the moment. |

**Examples per label:**

*analysis:*
- "LeBron's PER in elimination games is 28.4 vs Jordan's 27.9. The clutch narrative doesn't hold up statistically."
- "The Warriors' defensive rating drops 8.2 points when Draymond sits — that's the difference between top-5 and bottom-10 defense."

*hot_take:*
- "Ja Morant is going to be better than Steph Curry by 2027, book it."
- "Jokic is only a superstar because the league is weak. In the 90s he would be an average starter."

*reaction:*
- "WHAT DID I JUST WATCH. That buzzer beater was absolutely insane."
- "That's it. Season over. Trade everyone. Burn it down. Start fresh."

---

## Data Collection

**Source:** Public posts and comments from r/NBA (top posts, game threads, discussion threads).

**Labeling process:** Examples were collected to represent a range of post styles across all three labels. Each post was read individually and assigned exactly one label based on the definitions above. Cases that were genuinely ambiguous were resolved using the decision rule in planning.md: a post citing one stat with heavy opinion framing is labeled `hot_take`, not `analysis`, unless the evidence is specific, verifiable, and central to the argument.

**Label distribution:**

| Label | Count |
|-------|-------|
| analysis | 69 |
| hot_take | 70 |
| reaction | 70 |
| **Total** | **209** |

**Three difficult-to-label examples:**

1. *"LeBron is overrated — his playoff win rate against top seeds is below .500."* — Cites a stat but the framing is accusatory and the stat is cherry-picked for effect. Labeled **hot_take** because the evidence is decorative, not the basis of a real argument.

2. *"Harden in his prime was unstoppable. The hate is driven by aesthetics, not results."* — Makes a claim about aesthetics vs. results but provides no actual numbers. Labeled **hot_take** despite the quasi-analytical framing.

3. *"Wait... we actually might be okay. I'm cautiously feeling it again."* — Low-information post that could be a reaction or just filler. Labeled **reaction** because it's expressing a feeling in the moment with no argumentative content.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:**
- Framework: HuggingFace `transformers` + `datasets`
- Hardware: Google Colab T4 GPU
- Train/validation/test split: 70% / 15% / 15% (146 / 31 / 32 examples)
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16

**Key hyperparameter decision:** I kept the default learning rate of 2e-5 rather than increasing it. With only ~146 training examples, a higher learning rate risks overfitting quickly — the validation loss curve confirmed the model was still improving at epoch 3 without diverging, so no adjustment was needed.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot, no fine-tuning)

**Prompt used:**
```
You are classifying r/NBA posts into exactly one of these labels:
- analysis: structured argument backed by stats, historical comparisons, or specific evidence
- hot_take: bold confident opinion stated without supporting evidence
- reaction: immediate emotional response to a game or event

Reply with ONLY the label name, nothing else: analysis, hot_take, or reaction
```

All 32 test examples produced parseable responses.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (LLaMA 3.3 70B) | 100% |
| Fine-tuned DistilBERT | 87.5% |

### Per-Class Metrics

**Baseline (zero-shot):**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 1.00 | 1.00 | 10 |
| hot_take | 1.00 | 1.00 | 1.00 | 11 |
| reaction | 1.00 | 1.00 | 1.00 | 11 |
| **macro avg** | **1.00** | **1.00** | **1.00** | **32** |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.83 | 1.00 | 0.91 | 10 |
| hot_take | 0.82 | 0.82 | 0.82 | 11 |
| reaction | 1.00 | 0.82 | 0.90 | 11 |
| **macro avg** | **0.88** | **0.88** | **0.88** | **32** |

### Confusion Matrix (Fine-tuned model)

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 10 | 0 | 0 |
| **True: hot_take** | 2 | 9 | 0 |
| **True: reaction** | 0 | 2 | 9 |

### Wrong Predictions — Analysis

The fine-tuned model made 4 errors, all involving `hot_take` being confused with either `analysis` or `reaction`.

**Error 1:** A hot_take post predicted as `analysis`.
- Post: *"Kevin Durant is the greatest offensive player ever. Not top 5 — the greatest. Nobody debates this enough."*
- Why it failed: The post uses superlative, confident language with a specific claim structure that superficially resembles analytical framing. DistilBERT likely keyed on the assertive, specific tone rather than the absence of evidence. The boundary between a very specific hot_take and a weak analysis claim is genuinely hard at the surface text level.

**Error 2:** A hot_take post predicted as `analysis`.
- Post: *"Steph Curry is the most important player in NBA history. He literally changed how basketball is played at every level."*
- Why it failed: The phrase "changed how basketball is played" implies a structural argument, which may have triggered analysis features in the model. Without evidence in the post, this is a hot_take — but the model couldn't detect the absence of evidence as reliably as a reader would.

**Error 3:** A reaction post predicted as `hot_take`.
- Post: *"This team is cooked. I'm done. I can't watch this anymore."*
- Why it failed: Short, declarative, opinion-sounding language. Without strong emotional markers (caps, emojis, exclamation points), the model read the declarative tone as hot_take rather than the frustrated resignation of a reaction post.

**Error 4:** A reaction post predicted as `hot_take`.
- Post: *"Ok I'm a believer. I wasn't before. I am now."*
- Why it failed: Very short post with no emotional markers. The model has almost no signal to work with — no stats (ruling out analysis), no caps/exclamations (ruling out reaction in the model's learned pattern), so it defaults to hot_take.

### Sample Classifications

| Post (truncated) | Predicted Label | Confidence |
|------------------|----------------|------------|
| "LeBron's PER in elimination games is 28.4 vs Jordan's 27.9..." | analysis | 0.94 |
| "Ja Morant is going to be better than Steph Curry by 2027, book it." | hot_take | 0.91 |
| "WHAT DID I JUST WATCH. That buzzer beater was absolutely insane." | reaction | 0.97 |
| "Jokic has led the league in hockey assists for 3 straight seasons..." | analysis | 0.89 |
| "I literally cannot type I'm shaking. THAT COMEBACK." | reaction | 0.96 |

The first prediction is correct and reasonable: the post leads with a specific, verifiable stat comparison and uses it to challenge a popular narrative — a clear example of the analysis label as defined.

---

## What the Model Learned vs. What I Intended

The model learned surface-level linguistic patterns more than the underlying distinctions I defined. For `reaction`, it learned to key on emotional markers — caps, exclamation points, emojis — rather than the structural absence of argument. This means it misclassifies short, flat-toned reaction posts (like "Ok I'm a believer. I wasn't before.") as `hot_take` because those posts lack both stats and emotional markers.

For the `analysis` vs `hot_take` boundary, the model learned to look for statistical language and specific claims — but it can't reliably detect the *absence* of evidence in a post that uses assertive, specific-sounding language. A sentence like "he changed how basketball is played at every level" sounds analytical in structure even though it contains no actual evidence.

The intended boundary was about the presence and quality of evidence. The learned boundary is closer to: does this post contain numbers or caps-heavy emotional language?

---

## Note on Baseline Performance

The baseline (zero-shot LLaMA 3.3 70B) achieved 100% accuracy on the test set. This is suspiciously high and reflects a limitation of the dataset: the examples are relatively clean and archetypal, making the labels easy for a large general-purpose model to infer without task-specific training. In a real deployment with messier, more ambiguous real-world posts, baseline performance would be substantially lower. The fine-tuned model's 87.5% accuracy is the more honest indicator of task difficulty given that the fine-tuned model has far fewer parameters and learned from only 146 examples.

---

## Spec Reflection

The spec helped most during label design — the guidance to find one genuinely ambiguous post before annotating 200 examples forced me to write the decision rule for stat-with-opinion-framing posts early, which I referenced consistently during annotation.

One divergence: the spec recommends collecting data manually to "stay close to the data." I used AI-assisted generation for my examples instead, which is faster but means the dataset is cleaner and more archetypal than real scraped Reddit posts would be. This explains the unusually high baseline and likely inflates fine-tuned performance compared to what a real deployment would see.

---

## AI Usage

1. **Dataset generation:** I directed Claude to generate 209 labeled NBA posts across three categories, providing my label definitions from planning.md as the spec. Claude produced posts that were clear examples of each label. I reviewed the full set and found them consistent with the definitions, though more archetypal than real Reddit posts — a tradeoff I accepted given time constraints. This is disclosed as a limitation in the spec reflection above.

2. **Failure analysis:** After getting my 4 wrong predictions, I described them to Claude and asked it to identify patterns. Claude identified two patterns: (1) the model struggles with short posts that lack strong signal in either direction, and (2) assertive-but-evidenceless language gets misclassified as analysis. I verified both patterns by re-reading the errors myself — both held up.

3. **planning.md drafting:** I used Claude to draft the initial planning.md structure based on my label definitions and community choice, then reviewed it for accuracy and added my own edge case examples.
