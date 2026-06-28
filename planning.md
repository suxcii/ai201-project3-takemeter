# TakeMeter — planning.md
**Student:** Success Idemudia
**Course:** AI201 — Applications of AI Engineering
**Project:** Project 3 — TakeMeter

---

## 1. Community

**Chosen community:** r/anime (reddit.com/r/anime)

r/anime is one of the largest anime discussion communities on the internet, with millions of active members posting episode reactions, seasonal rankings, long-form reviews, recommendation threads, and hot takes daily. The community is text-heavy and discourse quality varies enormously — the same day you'll find a meticulously sourced essay on narrative structure sitting next to someone typing in all caps about a plot twist.

This makes r/anime an ideal fit for a classification task because:
- The discourse is genuinely varied in quality and intent, not just topic
- Posts are long enough to carry signal (unlike Twitter-style communities)
- The distinction between "real argument" and "bold opinion" is something the community itself actively debates — regulars frequently call out low-effort hot takes vs. genuine analysis
- Data is publicly accessible and text-rich

We are **not** focusing on one specific anime — we classify the type of discourse regardless of which anime is being discussed.

---

## 2. Label Taxonomy

### `analysis`
**Definition:** The post makes a structured argument supported by specific evidence — episode references, comparisons to source material, directorial choices, narrative structure, character writing, thematic breakdown, or cultural/historical context. The claim could be fact-checked or meaningfully debated on its merits. Removing the evidence would weaken or collapse the argument.

**Example 1:**
> "The reason Guts's character arc in Berserk works so well is because Miura deliberately parallels his trauma responses with real PTSD behavior — the dissociation during the Eclipse, the hypervigilance in the Black Swordsman arc. It's not edginess, it's meticulous psychological writing."

**Example 2:**
> "People forget that Madoka Magica was airing in March 2011 during the Tohoku earthquake — episodes 10-12 were delayed a full month. That context actually changes how the finale hit Japanese audiences at the time, which is why the domestic reception was so different from the Western one."

---

### `hot_take`
**Definition:** A bold, confident opinion stated without meaningful supporting evidence. The poster asserts rather than argues. The claim might be true or interesting — but they're not building a case for it. Framing is declarative and confident, not exploratory.

**Example 1:**
> "Demon Slayer is carried entirely by Ufotable and without that animation it would be a 6/10 at best. The writing has always been shallow."

**Example 2:**
> "Attack on Titan's final arc is actually better than people give it credit for and the hate is pure contrarianism. It's a masterpiece ending."

---

### `reaction`
**Definition:** An immediate emotional response, usually tied to a specific episode, chapter, or moment. Little to no argument is being made — the post is expressing a feeling in real time. Often uses caps, exclamation marks, or informal shorthand. The emotional content is the point.

**Example 1:**
> "I just finished episode 12 of Frieren and I haven't moved from my couch in 20 minutes. What is this show doing to me."

**Example 2:**
> "THAT JJK CHAPTER. GEGE IS ACTUALLY EVIL. I CANNOT BELIEVE WHAT I JUST READ."

---

## 3. Hard Edge Cases

### The one-stat hot take
**Post:**
> "Demon Slayer is overrated — it has an 8.7 on MAL but the writing is genuinely shallow compared to HxH."

**Why it's hard:** It cites a number (MAL rating), which looks like evidence. But the stat doesn't actually support the claim about writing quality — it's decorative, included to sound credible rather than to build an argument.

**Decision rule:** If removing the "evidence" would leave the opinion standing just as strongly, label it `hot_take`. Evidence in a true `analysis` post does real argumentative work — without it, the claim weakens or collapses. The one-stat post above loses nothing if you delete the MAL number. → `hot_take`

### The analytical reaction
**Post:**
> "Episode 23 just broke me. The way they paralleled that scene with episode 1 — same shot composition, same music, but everything means something different now. I'm devastated."

**Why it's hard:** The post is emotionally driven (reaction) but also references a specific structural technique (shot composition callback, musical motif reuse).

**Decision rule:** If the primary purpose of the post is to express an emotional state and the analysis is in service of explaining *why* they feel that way, label it `reaction`. If the emotional language is incidental and the post is primarily building an argument, label it `analysis`. The post above is grief first, observation second. → `reaction`

### The confident analysis
**Post:**
> "Chainsaw Man's anime adaptation failed because MAPPA prioritized visual spectacle over the manga's tonal dissonance. The color grading alone removed half of Fujimoto's intentional ugliness."

**Why it's hard:** Stated with hot-take confidence, but actually makes a specific claim about a directorial choice (color grading) tied to an artistic intent (tonal dissonance / Fujimoto's aesthetic).

**Decision rule:** If the post names a specific, verifiable creative decision and connects it to an artistic outcome, label it `analysis` — even if the tone is confident or confrontational. Confidence of delivery does not override presence of real argument. → `analysis`

---

## 4. Data Collection Plan

**Source:** reddit.com/r/anime — public posts from:
- Weekly discussion threads (episode discussion posts)
- Top posts of the past year filtered by "Top" and "Hot"
- Comments within high-engagement threads

**Target distribution:**
- `analysis`: ~70 examples (35%)
- `hot_take`: ~80 examples (40%)
- `reaction`: ~60 examples (30%)

Slightly higher hot_take representation because it's the most common post type on r/anime and we want the model to learn its nuances.

**If a label is underrepresented after 200 examples:** Search specifically for that type. For `analysis`, search post titles containing words like "breakdown," "why," "explains," "deep dive." For `reaction`, look in episode discussion comment sections from recently aired episodes.

**Collection method:** Manual copy-paste into a CSV file with columns: `text`, `label`, `notes`. AI will be used to pre-label batches of 20–30 posts at a time using my label definitions; I will review and correct every pre-assigned label before finalizing.

**CSV filename:** `anime_dataset.csv`

---

## 5. Evaluation Metrics

**Metrics I will use:**

- **Overall accuracy** — fraction of test examples correctly classified. Necessary baseline but not sufficient on its own.
- **Per-class F1 score** — harmonic mean of precision and recall for each label. This is the primary metric because class imbalance means a model could get high accuracy by over-predicting the majority class. F1 catches this.
- **Precision and recall per class** — to understand the direction of errors. Low recall on `analysis` means the model is missing real analysis posts; low precision means it's over-labeling things as analysis.
- **Confusion matrix** — to identify which label pairs the model systematically confuses. A cell at (hot_take, analysis) tells me the model is calling hot takes "analysis" — a specific, actionable pattern.

Accuracy alone is not enough because if 40% of posts are `hot_take`, a model that predicts `hot_take` every time gets 40% accuracy without learning anything. Per-class F1 exposes this.

---

## 6. Definition of Success

The classifier will be considered genuinely useful if:
- Overall accuracy exceeds **70%** on the test set
- No single label has an F1 below **0.60** (the model must learn all three distinctions, not just the easy ones)
- The fine-tuned model beats the Groq zero-shot baseline by at least **10 percentage points** on overall accuracy

A classifier meeting these thresholds could realistically be used as a moderation or quality-filtering tool in an anime community context — for example, flagging low-effort hot takes vs. surfacing substantive analysis posts in recommendation engines.

If the fine-tuned model does not beat the baseline by 10+ points, I will investigate label consistency and class distribution before concluding the task is too hard for 200 examples.

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating, I gave Claude my three label definitions and the edge case description and asked it to generate posts that sit at the boundary between labels. It produced the three hard edge cases documented in Section 3 above. Two of them (the one-stat hot take and the analytical reaction) required me to sharpen my decision rules before I would have been confident annotating consistently. I updated the definitions in Section 2 based on this.

### Annotation assistance
I will use Claude to pre-label batches of 20–30 posts at a time. I will provide the label definitions from Section 2 and a batch of raw post text, and ask it to assign one label per post with a one-sentence reason. I will then review every label and override where I disagree. Pre-labeled examples I override will be flagged in the `notes` column of the CSV. This workflow is disclosed in the AI usage section of the README.

### Failure analysis
After fine-tuning, I will paste my list of misclassified test examples into Claude and ask it to identify common patterns — post length, use of sarcasm, label pairs that keep getting confused, posts about specific anime that skew the results. I will then verify those patterns by re-reading the examples myself before writing the evaluation report. Any patterns Claude identifies that I cannot verify on re-reading will be discarded.

---

## Stretch Features (to be updated before starting)

- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface

*This section will be updated before beginning any stretch feature work.*