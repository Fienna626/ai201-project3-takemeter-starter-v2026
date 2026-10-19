# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts comments from **games@lemmy.world**, a Reddit-style gaming
discussion community, by the kind of opinion they express. Each comment is
labelled `analysis` (an argument that rests on a specific, checkable fact),
`hot_take` (a confident claim with no support offered), or `reaction` (the
writer's own immediate experience or feeling). The classifier is DistilBERT
fine-tuned on 233 comments I collected and labelled. The goal is to tell
argued opinions apart from unsupported ones and from personal reactions in a
community where all three show up in the same thread.


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `label_one`

**Definition:**

**Example 1:**
>

**Example 2:**
>

### `label_two`

**Definition:**

**Example 1:**
>

**Example 2:**
>

### The hardest boundary

**Which two labels:**

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public comments from **games@lemmy.world**, a
Reddit-style gaming community, collected on 2026-10-11 through Lemmy's public
API. I first tried r/Games, but Reddit blocked collection, and most r/Games posts
are news headlines that don't fit any of my labels. I kept only game-related
comments, using a keyword filter plus my own judgement, and deleted 17 that
were off-topic or had no real text: 3 while labelling cold, 14 flagged during
pre-labelling. Each row keeps its `thread`, `reply_to` and `post_title`, so
connected comments can be traced. Most comments share a thread with others,
so the same discussion can appear in both train and test.

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->
- **Cold first (20 rows, `note` = `cold`):** I labelled these myself with no
  AI suggestions.
- **The rest (213 rows, `note` = `pre-labelled`):** Claude pre-labelled them
  using the staff definitions. I spot-checked some and kept the rest as
  pre-labelled. Full disclosure is under How I Used AI.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `hot_take` | 119 | 51.1% |
| `reaction` | 62 | 26.6% |
| `analysis` | 52 | 22.3% |
| **Total** | 233 | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**2.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**3.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** <!-- epochs, learning rate, batch size, seed -->
- 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42.
- Trained on CPU (torch 2.14.1+cpu). Training took about 45 seconds.

**Anything I changed from the defaults, and why:** Nothing. I used the
notebook's defaults.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->
Train 163, validation 35, test 35.

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 36 | 8 | 8 |
| `hot_take` | 83 | 18 | 18 |
| `reaction` | 44 | 9 | 9 |

`analysis` (8) and `reaction` (9) are right at the "about 8" line in the test
split. Their F1 is computed on very few posts, so I expect it to move a lot
between seeds next unit.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked Claude Code to build `labels.csv` from r/Games.
- *What came back:* Reddit blocked the scraper (403, then 429 rate-limits). When
  I pasted three r/Games posts with my labels, Claude pointed out that all three
  were news headlines, and that a headline doesn't fit any of my labels
  well: it reports a fact without arguing anything, and it isn't anyone's
  personal reaction. It suggested collecting comments from games@lemmy.world
  instead, since the comments are where people actually argue and react.
- *What I changed:* I switched sources to Lemmy comments. After reading the
  first batch I saw off-topic comments (politics, movies), so I had it refill
  the file with game-related comments only, using a keyword filter. I kept my
  already-labelled rows and deleted the three cold rows I'd marked "not games".

**Moment 2**

- *What I asked for:* While labelling my cold rows, several comments were
  replies I couldn't understand on their own. I asked Claude to number the
  rows so I could see which ones were connected.
- *What came back:* It added `id`, `thread`, `thread_size`, `reply_to` and
  `post_title` columns by matching each comment back to its Lemmy thread. It
  also warned that the model only sees `text`, and that comments from the same
  thread will land in both train and test.
- *What I changed:* I re-did my 20 cold labels with the post titles visible.
  Three changed from my first pass, and I recorded each change in the `note`
  column (rows 3 and 10 became `hot_take`; row 1, which I'd been torn on,
  became `reaction`). I labelled from the comment text, using the title only
  to understand what the comment was about.

**Pre-labelling disclosure:**

- **Collection:** Claude Code wrote and ran the script that collected the
  comments through Lemmy's public API. I chose the community and the
  "game-related only" rule.
- **Cold labels (20 rows, `note` = `cold`):** I labelled these myself with no
  AI suggestions. The only help was the thread/title context columns described
  in Moment 2.
- **Pre-labelled (227 rows, `note` starts `pre-labelled`):** Claude pre-labelled
  these using the staff definitions in `data/staff_taxonomy.md`. It flagged 61
  as `unsure` and 14 as possible drops. I spot-checked some of the pre-labelled
  rows and kept the rest as pre-labelled, without reading every one. I deleted
  all 14 flagged rows (Discord-hack posts, tech-help replies, and image-only or
  link-only comments) because they broke my "game-related only" rule. That
  leaves 233 rows. Because I didn't read every pre-label, some of the labels are
  the AI's judgement rather than mine. That's a known source of noise, and I'll
  check it against my agreement report next unit.

**Moment 3**

- *What I asked for:* I asked Copilot to help interpret the saved confusion
  matrix and continue the staff-taxonomy labeling exercise.
- *What came back:* It identified that the seed-42 model predicted `hot_take`
  for every held-out post, and drafted provisional labels for the 30 staff
  posts from the written taxonomy.
- *What I changed:* I recorded the seed-42 confusion pattern in this README.
  The 30 staff labels are in `my_staff_labels.csv`, but they were drafted with
  AI assistance and have not been checked against the independent staff answer
  key; I do not report them as verified staff labels or as an agreement score.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
I expect the fine-tuned model to beat the zero-shot baseline overall,
especially on `hot_take`, because it has learned the distinctions from
examples in this gaming-comment dataset.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Held-out accuracy | ≥ 0.70 | 0.514 | Not run | Not run | MISSED |
| 2. Held-out macro F1 | ≥ 0.65 | 0.226 | Not run | Not run | MISSED |
| 3. F1 for each label | ≥ 0.60 each | `analysis` 0.000; `hot_take` 0.679; `reaction` 0.000 | Not run | Not run | MISSED |
| 4. No label exceeds 65% of the dataset | ≤ 65% | 51.1% max (`hot_take`, 119/233; dataset-level) | Same dataset | Same dataset | MET |
| 5. Confidence: most-confident third ≥ 0.70 accurate and ≥ 0.20 above least-confident third | Both conditions on each seed | 0.909 most; 0.273 least; gap 0.636 | Not run | Not run | MISSED |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42 (`results.json`; 35 held-out posts):

| true \ predicted | analysis | hot_take | reaction |
|---|---:|---:|---:|
| **analysis** | 0 | 8 | 0 |
| **hot_take** | 0 | 18 | 0 |
| **reaction** | 0 | 9 | 0 |

**My biggest off-diagonal number, and what it means:**
The largest off-diagonal is 9 real `reaction` posts called `hot_take`, with
zero `hot_take` posts called `reaction`; in this seed the model predicted
`hot_take` for every test post and did not distinguish either other label.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Held-out accuracy | ≥ 0.70 on each seed | MISSED | Seed 42 was 0.514, below target. Seeds 7 and 2024 were not run. |
| 2 | Held-out macro F1 | ≥ 0.65 on each seed | MISSED | Seed 42 was 0.226, below target. Seeds 7 and 2024 were not run. |
| 3 | F1 for each label | ≥ 0.60 on each seed | MISSED | Seed 42 had F1 0.000 for `analysis` and `reaction`; `hot_take` was 0.679. |
| 4 | Label balance | No label above 65% | MET | `labels.csv` has 233 posts; the largest class is `hot_take` at 119/233 (51.1%). |
| 5 | Confidence signal | On each seed, top third ≥ 0.70 accurate and ≥ 0.20 above bottom third | MISSED | Seed 42 passed (0.909 top-third accuracy; 0.273 bottom-third; gap 0.636), but seeds 7 and 2024 were not run, so the three-seed target was not demonstrated. |

**Diagnoses**

The available `results.json` records only seed 42; `results_three_seeds_before.json`
is not available, so there is no three-seed spread. Accuracy, macro F1, and
per-label F1 all miss their targets on seed 42. The confusion matrix shows the
model predicted `hot_take` for all 35 test posts: 18 were correct, while all 8
`analysis` posts and all 9 `reaction` posts were misclassified. Those two
classes have few test examples, so their estimates are unstable, but the
one-class prediction pattern is also evidence of model collapse on this split.
It does not establish whether the underlying cause is inconsistent training
labels, a genuinely hard boundary, or a task the model cannot learn from this
dataset. The independent staff labels and agreement check are unavailable, so
there is no evidence to diagnose a labeling problem. The confidence criterion
passed on seed 42, but is marked MISSED in the run log because the required
seeds 7 and 2024 were not run and the all-seed target was not demonstrated.

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Status: provisional — the staff repository was not working, so I could not
obtain `data/staff_labels.csv`.**

I labelled all 30 posts in `my_staff_labels.csv` under the staff taxonomy:
10 `analysis`, 10 `hot_take`, and 10 `reaction`. I verified that every text
matches `data/staff_posts.csv` exactly. Without the staff answer-key CSV, I
could not run the comparison, calculate an agreement rate, or identify
disagreements. I am treating these as my own provisional labels, not as
verified staff answers.

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

Not available yet. Without the staff labels there are no independently
identified disagreements to adjudicate or sort into piles (a), (b), and (c).
Comparing my labels to themselves would not be an agreement check.

**What the pattern in my disagreements tells me:**

Pending the independent staff labels; no disagreement pattern can be diagnosed
yet.


---

## The Improvement

**Status: pending — no improvement has been applied or evaluated yet.**

The only saved training result is seed 42, and the three-seed before run is
missing. That run shows the model predicted `hot_take` for every test post, but
it is not enough to distinguish a training-setting problem from a data or label
problem. The staff answer key and agreement check are also unavailable, so I
cannot justify relabelling a boundary from agreement evidence. I will first
need a working project environment and a complete three-seed before run before
choosing one evidence-based improvement.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Held-out accuracy | ≥ 0.70 | Not run | Not run | Not run | Not run |
| 2. Held-out macro F1 | ≥ 0.65 | Not run | Not run | Not run | Not run |
| 3. F1 for each label | ≥ 0.60 each | Not run | Not run | Not run | Not run |
| 4. No label exceeds 65% of the dataset | ≤ 65% | Not rechecked | Not rechecked | Not rechecked | Not run |
| 5. Confidence: most-confident third ≥ 0.70 accurate and ≥ 0.20 above least-confident third | Both conditions on each seed | Not run | Not run | Not run | Not run |

**Did it help, and how do I know:**

Not evaluated. No improvement or three-seed after run has been completed.
<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**Criterion 1 — held-out accuracy ≥ 0.70 on every seed:** Seed 42 reached
0.514. I would run sections 6's seeds 7 and 2024 before drawing a three-seed
conclusion, then select one improvement supported by that complete run. I
stopped because the project environment still blocks NumPy's compiled
extension, and the notebook runner did not execute cells with the selected
kernel.

**Criterion 2 — held-out macro F1 ≥ 0.65 on every seed:** Seed 42 reached
0.226. I would collect all three seed results and compare the per-label scores
and confusion matrices to see whether this is stable or seed-specific. I
stopped for the same environment and notebook-kernel blockers; only one
seed's result is saved.

**Criterion 3 — F1 ≥ 0.60 for every label on every seed:** Seed 42 gave
`analysis` and `reaction` F1 scores of 0.000, while `hot_take` was 0.679.
Their supports were only 8 and 9 test posts, so I would first inspect a valid
three-seed run and, once available, the independent staff agreement results;
then I would consider adding carefully labeled examples to a thin class. I
stopped because the other seed results and staff answer key are unavailable,
so I cannot distinguish low support from a label-boundary or training issue.

**Criterion 5 — confidence target on every seed:** Seed 42 met both thresholds
(0.909 accuracy in the most-confident third, 0.273 in the least-confident
third, a 0.636 gap). I would check the same measures on seeds 7 and 2024. I
stopped because those seeds have not been run, so the across-seed target is
not established.

Criterion 4 (label balance) is met: the largest class is 51.1% of the 233-row
dataset.

**The gap between what I meant my labels to capture and what the model
learned:**
I meant the labels to distinguish evidence-backed `analysis`, unsupported
`hot_take`, and personal `reaction`, but on seed 42 the model predicted
`hot_take` for all 35 held-out posts. The matrix shows all 8 `analysis` and all
9 `reaction` posts were called `hot_take` (while 18 `hot_take` posts were
correct), so this run learned the majority response rather than the intended
three-way distinctions.

**Repository URL for this unit (same existing fork):**
https://github.com/Fienna626/ai201-project3-takemeter-starter-v2026

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
