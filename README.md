# TakeMeter: Astrology Discourse Classifier

A fine-tuned text classifier that categorizes posts from Reddit's astrology communities into three discourse types: **analysis**, **hot_take**, and **reaction**. Built as part of AI201 Project 3.

---

## Community Choice

I chose Reddit's astrology communities — primarily **r/astrologymemes**, **r/astrology**, and **r/AskAstrologers** — because astrology discourse spans an unusually wide quality range in a compact, text-heavy format. A single subreddit might contain a carefully reasoned breakdown of a natal chart transit, a bold claim that Scorpios are manipulative by nature, and someone screaming "this meme is literally my Virgo moon" — all within the same thread. That variety made it a strong fit for a classification task: the distinctions are real, they matter to community members, and they're hard enough that a model can't just pattern-match on surface keywords alone.

---

## Label Taxonomy

### `analysis`
A post that references specific placements, aspects, houses, transits, or chart mechanics to explain or support a point. There is actual astrological reasoning present — a logical chain connecting chart factors to a conclusion.

**Example 1:** "Her Venus conjunct Pluto in the 8th house is why she experiences love as obsessive and all-consuming — that aspect craves transformation through relationships."

**Example 2:** "Saturn return hits hardest for Capricorn risings because Saturn rules their chart — everything it touches restructures from the ground up, not just one area of life."

---

### `hot_take`
A bold or controversial astrological claim stated with confidence but little to no reasoning or chart-based evidence. The post asserts something as fact without explaining why astrologically.

**Example 1:** "Geminis are literally incapable of being loyal, it's in their nature."

**Example 2:** "Capricorns and Taurus's are good to have as best friends." — stated flatly, no chart mechanics cited.

---

### `reaction`
An emotional or personal response — usually to a meme, someone else's post, or their own chart — with no astrological reasoning attached. Pure relatable content, venting, or excitement.

**Example 1:** "I just found out I have Chiron in the 7th and I've never felt so seen and attacked at the same time."

**Example 2:** "why is this SO accurate for my Virgo moon omg"

---

### Hard Edge Cases

Three posts that were genuinely difficult to label during annotation:

**1. "Taurus is exalted in the Moon sign because of their need for safety and stability, [but] does [not] mean that they are stable. All Taurus-Moon people I know have extreme anger issues if things don't go their way."**
- Could be `analysis` (mentions exaltation, a real chart mechanic) or `hot_take` (the core claim — anger issues — is personal observation with no astrological logic connecting the mechanic to the behavior).
- **Decision:** `hot_take`. The astrological term is decorative framing; the claim doesn't follow from it. Dropping "exalted in the Moon" wouldn't change the argument.

**2. "Pisceans are sensitive but the most emotional are Capricorns & Aquarians. Y'all just don't allow yourself to feel the emotions bc of how strong these emotions are that's all :)"**
- Could be `hot_take` (bold cross-sign comparison) or `analysis` (there's a quasi-reasoning about suppression vs. feeling).
- **Decision:** `hot_take`. The "reasoning" is intuitive folk psychology, not chart mechanics. No placements, aspects, or transits cited.

**3. "Cap moons: It's all fun and giggles here, we are fine 🥰 Cap moons as well: Crying at a scheduled time, in one random place at home, preferably while showering"**
- Could be `hot_take` (makes a claim about Cap moon behavior) or `reaction` (structured as a relatable meme format expressing a feeling).
- **Decision:** `reaction`. The post's intent is relatability and humor, not assertion. It's performing an emotion, not arguing a position.

---

## Data Collection

**Sources:** r/astrologymemes, r/astrology, r/AskAstrologers (all public Reddit posts and comments)

**Collection method:** Manual — posts were read and copy-pasted into a CSV spreadsheet. Reading each post during collection helped calibrate label intuition before formal annotation began.

**Dataset size:** 200 labeled examples

**Label distribution:**

| Label | Count |
|-------|-------|
| analysis | 67 |
| hot_take | 66 |
| reaction | 67 |
| **Total** | **200** |

The dataset is nearly perfectly balanced across all three labels, which was intentional — examples were collected until each label reached roughly equal representation.

**Labeling process:** Each post was read individually and assigned a label based on the definitions above. Posts that triggered genuine uncertainty were noted and resolved using the decision rules documented in planning.md.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:**
- Framework: HuggingFace `transformers` + `datasets`
- Platform: Google Colab (T4 GPU)
- Train/validation/test split: 70% / 15% / 15% (automated by starter notebook)

**Hyperparameters (defaults used):**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16

No hyperparameter modifications were made. The default settings were kept to establish a clean baseline for what the model can learn without tuning.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**

```
You are classifying posts from the astrology community on Reddit.
Assign each post to exactly one of the following categories.

analysis: A post that references specific placements, aspects, houses, transits, or chart mechanics
to explain or support a point. There is actual astrological reasoning present.

hot_take: A bold or controversial astrological claim stated with confidence but little to no
reasoning or chart-based evidence.

reaction: An emotional or personal response — usually to a meme, someone else's post, or their
own chart — with no astrological reasoning attached.

Respond with ONLY the label name — nothing else, no explanation, no punctuation.
```

The baseline was run on the locked test set (30 examples) before any fine-tuning occurred.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **0.867** |
| Fine-tuned DistilBERT | **0.567** |

Fine-tuning regression: **-0.300** — the fine-tuned model performed substantially worse than the zero-shot baseline.

---

### Per-Class Metrics (Baseline — Groq llama-3.3-70b-versatile)

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 0.80 | 0.89 | 10 |
| hot_take | 0.75 | 0.90 | 0.82 | 10 |
| reaction | 0.90 | 0.90 | 0.90 | 10 |
| **macro avg** | 0.88 | 0.87 | 0.87 | 30 |

---

### Per-Class Metrics (Fine-Tuned Model)

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.45 | 1.00 | 0.62 | 10 |
| hot_take | 0.67 | 0.20 | 0.31 | 10 |
| reaction | 1.00 | 0.50 | 0.67 | 10 |
| **macro avg** | 0.71 | 0.57 | 0.53 | 30 |

---

### Confusion Matrix (Fine-Tuned Model)

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|---------------------|---------------------|---------------------|
| **True: analysis** | 10 | 0 | 0 |
| **True: hot_take** | 8 | 2 | 0 |
| **True: reaction** | 4 | 1 | 5 |

The model correctly classified all 10 `analysis` examples, but systematically collapsed `hot_take` and `reaction` posts into `analysis`. 8 of 10 hot_takes and 4 of 10 reactions were predicted as `analysis`.

---

### Wrong Prediction Analysis

**Wrong predictions: 13 / 30**

The dominant failure pattern is clear: the model predicted `analysis` for 12 of the 13 wrong predictions, all at very low confidence (~0.34–0.39). This is not a model that learned to distinguish the three labels — it learned to predict `analysis` as a near-default, with weak signal separating it from the other classes.

**Failure 1:**
> "Cancer men can be so whiney. Like stop complaining and being a victim."
> True: `hot_take` → Predicted: `analysis` (confidence: 0.35)

This post has no astrological reasoning — it's a blunt opinion about a sun sign. However, it names a specific sign (Cancer), and the model may have learned that sign references correlate with `analysis`. The confidence is very low, suggesting the model is uncertain rather than confidently wrong. This points to a surface-feature problem: sign mentions appear in all three label types, but the model weighted them too heavily toward `analysis`.

**Failure 2:**
> "Cap moons: It's all fun and giggles here, we are fine 🥰 Cap moons as well: Crying at a scheduled time, in one random place at home, preferably while showering"
> True: `reaction` → Predicted: `analysis` (confidence: 0.36)

This is a meme-format post — a relatable two-part structure expressing an emotion about a placement. The model likely keyed on "Cap moons" (a specific placement) as an `analysis` signal. But the post contains no reasoning — it's performing an emotion. This reveals a core failure: the model didn't learn that *what you do with a placement* matters, only that referencing one is sufficient.

**Failure 3:**
> "Gemini gets the reputation of being two-faced, BUT LIBRA THO"
> True: `hot_take` → Predicted: `analysis` (confidence: 0.36)

This is a classic hot_take — a bold comparison between two signs with zero reasoning. The model's failure here is particularly instructive because the post is short, punchy, and opinionated in a way that should contrast with the structured reasoning of `analysis`. The model didn't learn the structural difference between assertion and argument; it learned something shallower, possibly that multi-sign posts lean toward `analysis`.

---

### Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|------------------|------------|-----------|------------|
| "Her Venus conjunct Pluto in the 8th house..." | analysis | analysis | 0.82 |
| "Scorpios are not as cool and collected as they seem." | hot_take | analysis | 0.36 |
| "why is this SO accurate for my Virgo moon omg" | reaction | analysis | 0.35 |
| "Saturn rules Capricorn so their return restructures everything..." | analysis | analysis | 0.79 |
| "I literally cannot stand Aries energy, they are SO aggressive" | reaction | hot_take | 0.34 |

**Correct prediction explained:** The first example ("Venus conjunct Pluto in the 8th house") was correctly classified as `analysis` with high confidence (0.82). This is reasonable — the post names a specific aspect, a specific house, and connects them to a behavioral pattern through astrological logic. It looks structurally different from hot_takes and reactions: longer, more precise, with the kind of "because" reasoning that distinguishes analysis from assertion. This is exactly what the model should have learned to recognize — it just didn't generalize this pattern reliably across the test set.

---

### Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the *structural* difference between three types of discourse: reasoned argument (analysis), unsupported assertion (hot_take), and emotional response (reaction). What it actually learned was much shallower — something closer to: "if a post mentions a placement or aspect, predict analysis."

The evidence is the confusion matrix. The model gets `analysis` right 100% of the time (10/10) but misclassifies 80% of hot_takes and 40% of reactions — almost always as `analysis`. This makes sense if the model latched onto surface features like sign names, placement vocabulary (moon, conjunct, house), and post length rather than the reasoning structure. Hot_takes often mention signs ("Geminis are disloyal"); reactions often mention placements ("my Virgo moon"); and both are short. The model couldn't distinguish *mentioning* an astrological concept from *reasoning with* one.

With only 200 examples and 3 epochs, DistilBERT didn't have enough data to learn the subtler cue — which is whether the post contains a logical chain, not just astrological vocabulary. The zero-shot baseline outperformed it substantially (0.867 vs. 0.567) because Llama-3.3-70b already understands argument structure from pre-training and could apply the definitions more reliably. Fine-tuning on 140 training examples instead gave the model a noisy signal that reinforced a surface pattern it was already prone to.

---

## Spec Reflection

**One way the spec helped:** The instruction to run the baseline *before* fine-tuning was valuable — it locked the test set and gave the fine-tuned model's results real interpretive meaning. Without that ordering, a higher fine-tuned accuracy might have felt like success even without context. Seeing the baseline at 0.867 first made the 0.567 result immediately legible as a regression, not just a number.

**One way implementation diverged:** The spec suggests fine-tuning should improve on the baseline, and treats fine-tuning regression as a sign to investigate bugs or data leakage. In this case, the regression appears genuine — not a bug — because the task requires understanding argument structure, which a large pre-trained LLM handles better zero-shot than a small model fine-tuned on 140 examples with noisy label boundaries. The divergence from expected results became the most analytically interesting part of the project.

---

## AI Usage

**Instance 1 — Failure pattern analysis:** After collecting the 13 wrong predictions, I pasted them into Claude and asked it to identify common themes across the misclassifications. It identified that almost all errors involved predicting `analysis` for posts that merely mentioned a sign or placement without reasoning — and that confidence scores were uniformly low (~0.35), suggesting the model was uncertain rather than confidently wrong. I verified this pattern myself by reviewing each wrong prediction and confirmed it held. This framing shaped the "what the model learned vs. intended" reflection above.

**Instance 2 — Groq prompt refinement:** An early version of the baseline prompt used brief one-line label definitions. Claude helped me expand the definitions and add concrete examples directly in the prompt, which reduced unparseable responses from the baseline by making the expected output format unambiguous. I reviewed and edited the expanded definitions to match my planning.md language before using them.

---

## Repository Contents

```
ai201-project3-takemeter/
├── README.md
├── planning.md
├── dataset.csv               # 200 labeled examples
├── confusion_matrix.png      # Generated by Colab notebook
└── evaluation_results.json   # Generated by Colab notebook
```
