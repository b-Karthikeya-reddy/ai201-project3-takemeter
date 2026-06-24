# TakeMeter Planning — r/NBA Discourse Classifier

## Community

I chose r/NBA because it's one of the most active sports communities on Reddit, with hundreds of posts daily ranging from data-driven breakdowns to pure emotional reactions. The discourse is highly varied: some users cite advanced stats and historical comparisons, others post visceral reactions to games, and many just assert bold opinions with no backing. This variance makes it a strong fit for a classification task.

## Labels

**analysis** — The post makes a structured argument supported by specific statistics, historical comparisons, or tactical observations. Evidence is verifiable and central to the claim, not decorative.
- Example 1: "LeBron's PER in elimination games is actually higher than Jordan's — 28.4 vs 27.9 — which makes the 'clutch' narrative hard to sustain."
- Example 2: "The Warriors' defensive rating drops 8 points when Draymond sits, which explains why they lose leads in the third quarter."

**hot_take** — A bold, confident opinion stated without supporting evidence. The claim may or may not be true, but the post asserts rather than argues.
- Example 1: "Ja Morant is going to be better than Steph Curry by 2027, it's not even close."
- Example 2: "The NBA is unwatchable without LeBron. Nobody else moves the needle."

**reaction** — An immediate emotional response to a specific game or event. Little to no argument — the post is expressing a feeling in the moment.
- Example 1: "WHAT DID I JUST WATCH. That buzzer beater is going to live in my head rent free forever."
- Example 2: "I can't believe they blew a 20-point lead. This team is cooked."

## Hard Edge Cases

The hardest case is a post that cites one stat but is mostly opinion framing — e.g., "LeBron is overrated, his playoff win rate vs. top seeds is below .500." 

**Decision rule:** If the evidence is specific and verifiable AND would support the claim even without the opinion framing, label it **analysis**. If the stat feels cherry-picked or decorative — included to sound credible rather than to reason — label it **hot_take**. One vague stat + strong opinion framing = hot_take.

## Data Collection Plan

- Source: r/NBA public posts and comments (top posts, controversial posts, game threads)
- Target: ~70 examples per label (210 total) to avoid imbalance
- If a label is underrepresented after 150 examples, I'll specifically search for posts that fit that label (e.g., game thread comments for reactions, film breakdown posts for analysis)
- All data collected manually from public Reddit

## Evaluation Metrics

I will use:
- **Overall accuracy** — for a broad picture
- **Per-class F1** — because the classes may be imbalanced and accuracy alone would hide poor performance on minority classes
- **Confusion matrix** — to identify which specific label pairs are being confused

Accuracy alone is insufficient because a model that always predicts "hot_take" could achieve decent accuracy if that label is overrepresented. F1 penalizes this.

## Definition of Success

A classifier is useful for community moderation or flair-tagging if:
- Overall accuracy ≥ 70% on the test set
- No single class has F1 below 0.55
- Fine-tuned model meaningfully outperforms the zero-shot baseline (at least +10% accuracy)

## AI Tool Plan

**Label stress-testing:** I'll give Claude my label definitions and ask it to generate 10 boundary posts — posts that could plausibly belong to two labels — to test whether my definitions hold up before annotating 200 examples.

**Annotation assistance:** I'll use an LLM to pre-label batches of 30 posts using my label definitions, then review and correct every single label before using it in training. All pre-labeled examples will be noted in the AI usage section.

**Failure analysis:** After getting wrong predictions from the model, I'll paste them into Claude and ask it to identify patterns (length, sarcasm, label pair confusion). I'll verify each pattern myself before including it in the report.
