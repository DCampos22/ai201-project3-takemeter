# TakeMeter — planning.md

> Complete this document before collecting or labeling any data.
> Your label definitions and annotation rules are what you'll use to direct AI tools (Claude, etc.)
> to help stress-test boundaries, pre-label examples, and analyze failures.
> Update this document before starting any stretch features.

---

## Community

**What community did you choose and why?**

I chose the astrology community on Reddit, primarily drawing from r/astrology and r/AskAstrologers. These subreddits are highly active, text-heavy, and produce a wide range of discourse quality — from deeply reasoned chart analysis to pure emotional reaction to bold, unsubstantiated takes.

This community is a strong fit for a classification task because the *gap between sounding authoritative and actually reasoning astrologically* is real and consistent. Astrology has its own vocabulary (placements, aspects, transits, houses) and someone fluent in that vocabulary can still make a claim without any logical support. That tension — between a confident claim and a reasoned one — is exactly what TakeMeter is designed to surface. Regular participants in this community absolutely recognize the difference between a hot take and genuine analysis, even if they don't always call it out explicitly.

---

## Label Taxonomy

List every label your classifier will use. For each label, fill in all four fields.

### Label 1: `hot_take`

**Definition:**
A bold or controversial astrological claim stated with confidence but little to no reasoning or chart-based evidence. The post asserts something as fact without explaining *why* astrologically.

**Example 1 (clear case):**
"Geminis are literally incapable of being loyal, it's in their nature"

**Example 2 (clear case):**
"Scorpio risings are the most intimidating people in any room, not up for debate"

**Uncertain case:**
"Scorpio Moons are emotionally manipulative, it's a Pluto thing"
→ This names a planet but doesn't explain the mechanism. I'll label this `hot_take` because naming a planet without reasoning from it is still just a claim with a prop.

---

### Label 2: `analysis`

**Definition:**
A post that references specific placements, aspects, houses, transits, or chart mechanics to explain or support a point. There is actual astrological reasoning present — not just a claim, but a logical chain connecting chart factors to a conclusion.

**Example 1 (clear case):**
"Her Venus conjunct Pluto in the 8th house is why she experiences love as obsessive and all-consuming — that aspect craves transformation through relationships"

**Example 2 (clear case):**
"Mercury in Pisces people struggle to communicate directly because Mercury is in detriment there; the mind operates through feeling rather than logic"

**Uncertain case:**
"Libra risings are charming because Venus rules their chart"
→ This has a "because" but the reasoning is shallow — it doesn't explain *how* Venus rulership produces charm. I'll label this `hot_take` unless the post expands on the mechanism.

---

### Label 3: `reaction`

**Definition:**
An emotional or personal response — usually to a meme, someone else's post, or their own chart — with no astrological reasoning attached. Pure relatable content, venting, or excitement.

**Example 1 (clear case):**
"I just found out I have Chiron in the 7th and I've never felt so seen and attacked at the same time"

**Example 2 (clear case):**
"why is this SO accurate for my Virgo moon omg"

**Uncertain case:**
"I literally cannot stand Aries energy, they are SO aggressive"
→ This reads like frustration (reaction) but also makes a sign-based claim (hot_take). I'll label it `reaction` because the primary function is venting an experience, not asserting a general truth about Aries placements.

---

## Hard Edge Cases

**What type of post will be genuinely ambiguous between two labels? How will you handle it?**

**Edge case 1: `hot_take` vs. `analysis` — naming a placement without reasoning**

The hardest cases will be posts that *name a placement* but don't actually reason from it:
> "Scorpio Moons are emotionally manipulative, it's a Pluto thing"

This mentions a planet but doesn't explain the mechanism. My rule: **if there is no logical chain connecting the placement to the claim, it's a `hot_take`** — even if astrological vocabulary is present. The presence of a planet name alone does not make something analysis.

**Edge case 2: `reaction` vs. `hot_take` — frustrated claims**

Some posts are emotionally charged *and* make a general claim:
> "I literally cannot stand Aries energy, they are SO aggressive"

My rule: **if the primary function of the post is to vent or share a personal experience, label it `reaction`. If the primary function is to assert something about a sign or placement as a general truth, label it `hot_take`.** When stuck, I ask: is this person talking about themselves, or making a claim about a sign?

**Tiebreaker rule:** When genuinely stuck between two labels, I assign the label that reflects the post's *primary intent* and note it in the `notes` column of the CSV.

---

## Data Collection Plan

**Sources:**
- r/astrology — high volume, mixed quality; good for `hot_take` and `reaction`
- r/AskAstrologers — more analysis-heavy; good for `analysis`
- Sign-specific subreddits (r/Scorpio, r/Gemini) — concentrated source of `hot_take` content

**Collection method:** Manual copy-paste into a Google Sheet with three columns: `text`, `label`, `notes`. Collecting comments over full posts where possible — they tend to be shorter, more opinionated, and easier to classify.

**Target distribution (200 examples total):**

| Label | Target Count | Notes |
|-------|-------------|-------|
| `hot_take` | ~67 | Sign-specific subs are a reliable source |
| `analysis` | ~67 | r/AskAstrologers, posts tagged "chart reading" |
| `reaction` | ~67 | Meme posts and "this is so me" comment threads |

**If a label is underrepresented after 150 examples:** I will seek out content from sources likely to produce that label specifically rather than continuing to collect from general feeds. For `analysis`, I'll search for posts with terms like "natal chart," "transit," or "aspect." For `hot_take`, I'll browse sign-specific subreddits sorted by Top. I will not move to annotation until all three labels are within 15 examples of each other.

---

## Evaluation Metrics

**Which metrics will you use and why are they the right ones for this task?**

I will report the following for both the fine-tuned model and the zero-shot baseline:

| Metric | Why it matters for this task |
|--------|------------------------------|
| Overall accuracy | Gives a top-line sense of performance across all classes |
| Per-class F1 score | Most important — balances precision and recall per label; catches cases where the model ignores one class |
| Per-class precision | "Of all examples predicted as this label, how many actually were?" — reveals false positives per label |
| Per-class recall | "Of all true examples of this label, how many did the model catch?" — reveals false negatives per label |
| Confusion matrix | Shows which label pairs are being confused and in which direction |

Accuracy alone is insufficient because a model could achieve ~33% accuracy by always predicting the majority class on a 3-label task. F1 per class tells me whether the model is learning each distinction or defaulting to one label.

The confusion matrix is especially important here because the `hot_take` / `analysis` boundary is the hardest one — I expect most errors to cluster there, and the matrix will confirm or disprove that hypothesis.

---

## Definition of Success

**What performance would make this classifier genuinely useful? What would you accept as "good enough" for deployment?**

A classifier is **genuinely useful** for a community moderation or discourse quality tool if:
- Overall accuracy ≥ 70% on the test set
- All three per-class F1 scores ≥ 0.60 (no label is being systematically ignored)
- Fine-tuned model outperforms zero-shot baseline by at least +10 percentage points on overall accuracy

**Strong result:** ≥ 85% accuracy with all F1 scores ≥ 0.75 — would deploy with confidence.

**Acceptable result:** 70–84% accuracy with all F1 ≥ 0.60 — would deploy with a human review layer for low-confidence predictions.

**Not deployable:** Any single class F1 below 0.50 — means one boundary hasn't been learned and predictions for that label can't be trusted.

---

## AI Tool Plan

**Label stress-testing:**
Before annotating, I'll give Claude my three label definitions and edge case rules and ask it to generate 10 posts that sit at the boundary between `hot_take` and `analysis`. If it produces posts I can't cleanly classify, I'll tighten the definitions before committing to 200 examples. I'll verify by checking: can I assign every generated post to exactly one label without hesitation? If not, the boundary needs sharpening.

**Annotation assistance:**
I may use Claude to pre-label a batch of 50–100 examples by providing my label definitions and a set of unlabeled posts and asking it to assign one label per post. If I use this workflow: (1) I will review and correct every pre-assigned label individually — no bulk acceptance, (2) I will track which examples were pre-labeled in the `notes` column, and (3) I will disclose this in the AI usage section of the README. Skimming without genuine review produces noisy training data, so I won't do it.

**Failure analysis:**
After fine-tuning, I'll paste my list of misclassified examples into Claude and ask it to identify common patterns — post length, sarcasm, label pairs being confused, low-information posts, etc. I'll verify those patterns myself by re-reading the examples before writing up my evaluation. I'll note what the AI identified and what I had to correct or discard.

---

## Hard Annotation Decisions (updated during data collection)

*This section will be updated as I annotate. At least 3 specific difficult cases will be documented here — the post text, which labels it could belong to, and what I decided and why.*

| # | Post text | Could be | Decision | Reasoning |
|---|-----------|----------|----------|-----------|
| 1 | *(to be filled in)* | | | |
| 2 | *(to be filled in)* | | | |
| 3 | *(to be filled in)* | | | |

---

## A Complete Classification Example (Step by Step)

**Example post:**
> "Her Venus conjunct Pluto in the 8th house is why she experiences love as obsessive — Pluto intensifies everything it touches, and in the 8th house that energy goes deep into shared resources and emotional merging. Venus here doesn't do casual."

**Step 1 — Read the post:**
The post references a specific aspect (Venus conjunct Pluto), a specific house (8th), and explains *why* that combination produces a particular pattern (Pluto intensifies, 8th house = emotional merging). There is a logical chain: placement → mechanism → behavioral outcome.

**Step 2 — Check against label definitions:**
- `hot_take`: No — there is reasoning present, not just a claim.
- `analysis`: Yes — specific placement + aspect + house + explained mechanism.
- `reaction`: No — this is not a personal emotional response.

**Step 3 — Assign label:**
Label: `analysis`

**Step 4 — Check for edge case:**
Does this merely name a placement without reasoning? No — it explains the mechanism ("Pluto intensifies everything it touches"). This is a clean `analysis` case.

**Notes column entry:** None — clear case, no ambiguity.

---

## Stretch Features (update before starting each one)

- [ ] Inter-annotator reliability — have at least one other person label 30+ examples, report Cohen's kappa
- [ ] Confidence calibration — report whether confidence scores correlate with accuracy
- [ ] Error pattern analysis — identify a systematic pattern across wrong predictions
- [ ] Deployed interface — build a simple UI that accepts a post and returns label + confidence