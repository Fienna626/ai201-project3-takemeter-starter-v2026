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
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

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

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
