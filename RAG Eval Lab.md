3-day build · Databricks · MLflow 3

# RAG Eval Lab: measure first, *then* optimize.

Which RAG setup works best, and can we trust the judge that says so?

Not another chatbot. A benchmark-driven framework that tests one setting at a time, scores every run against known answers, and checks two AI judges against ground truth.

HotpotQA · distractorDelta LakeMLflow tracingLLM judge vs Jev

Recall@5 by config · illustration

cfg_s1_k3

0.52 cfg_s2_k3

0.61 cfg_p_k5

0.58 cfg_s3_k5

0.74 baseline

0.48

Placeholder numbers. Your real ones replace these on Day 1.

The finish line

## What exists at the end of Day 3

If the notebooks run but nothing is published, almost none of the career value lands. These five things are the deliverable.

1 · Repo

### GitHub repo

Notebooks driven by one config table. Anyone can rerun it end to end.

2 · Data

### Delta tables

Raw data to chunks to results to metrics. Every experiment is queryable with SQL.

3 · Observability

### MLflow experiment

One run per config, with traces, latency, tokens and judge scores.

4 · Findings

### Results write-up

Best config, failure attribution, judge agreement and cost.

5 · Reach

### LinkedIn post

Your words, linking the repo and the write-up.

The one sentence the project must earn:\
Config \_\_\_\_ beat the baseline by \_\_\_\_ Recall@5 and \_\_\_\_ answer F1. Jev agreed with ground truth \_\_\_% of the time at $\_\_\_\_ per 1,000 judgments, versus \_\_\_% for the LLM judge.

| Table | Layer | Holds |
| --- | --- | --- |
| bronze_hotpot_raw | bronze | HotpotQA records as downloaded |
| silver_paragraphs | silver | Deduplicated Wikipedia paragraphs, split into sentences |
| silver_eval_questions | silver | Question, gold answer, supporting paragraphs and sentences |
| exp_configs | silver | One row per config: chunk size, overlap, Top-K, retrieval strategy |
| silver_chunks | silver | Chunks and embeddings, keyed by chunking config |
| exp_retrievals | gold | Ranked chunks returned per question and config |
| exp_generations | gold | Answers with trace id, latency and tokens |
| exp_judgments | gold | Score per question, config and judge, with cost |
| exp_metrics | gold | Recall@K, NDCG, MRR, EM, F1, agreement rates per config |

A starting point for your schema session. The final design is yours.

The design

## A cheap filter before the expensive part

Retrieval can be scored with no LLM calls, so every config gets tested there first. Only the best few pay for generation and judging.

STAGE 0**Data prep**

**HotpotQA**\~300 questions, \~10 paragraphs each

**Delta**bronze → silver

**Eval set**answers + supporting facts

STAGE 1**Retrieval grid**

**\~36 configs**chunk · overlap · Top-K · strategy

**Embed + retrieve**in notebook

**Recall@K · NDCG · MRR**zero LLM cost

36 configs → **top 3–4 + baseline**

STAGE 2**Generate + judge**

**Generate**MLflow traces, tokens, latency

**Ground truth**exact match, F1

**LLM judge**rationale, slower

**Jev**probabilities, cheap

STAGE 3**Analysis**

**SQL on Delta**compare configs and judges

**Optimized RAG**best config + why it fails

The core idea

## Where does a bad answer come from?

Every wrong result gets traced to one of three causes. This is what separates an evaluation framework from a demo.

Did retrieval return the supporting paragraphs?No → retrieval problem

Fix chunking, Top-K or the embedding strategy. Stage 1 metrics show this directly.

Context was right, but the answer fails ground truth?Yes → generation problem

The model had the facts and still missed. Fix the prompt or the model, not the retriever.

Ground truth and the judge disagree?→ evaluation problem

The measuring tool is wrong. You read these cases yourself; that is where the insight is.

Success criteria

## Done means all seven are true

Tick them off as you go. Your ticks stay in this browser only.

0 / 7

[ ] **Clean HotpotQA subsample in Delta**\~300 questions with answers and supporting-fact labels; row counts verifiedDAY 1

[ ] **Retrieval grid scored**\~36 configs with Recall@K, NDCG, MRR in one metrics tableDAY 1

[ ] **Shortlist run end to end with traces**3–4 configs plus baseline, each an MLflow run with latency and tokensDAY 2

[ ] **Three evaluators scored every answer**Ground truth, LLM judge and Jev, in one judgments tableDAY 2–3

[ ] **Judge agreement and cost reported**Agreement rate, disagreement examples you read, cost per 1,000DAY 3

[ ] **Best config named, failures attributed**Retrieval vs generation vs evaluation, backed by SQLDAY 3

[ ] **Published**Repo, write-up and LinkedIn post are liveDAY 3

Trade-offs

## What we chose, and what it costs

We trade breadth for a result we can finish, explain and trust in three days.

HotpotQA original, not BEIR or SciFact

Gold answers and supporting sentences, multi-hop questions

Short paragraphs, so chunking is in sentences, not tokens

Two-stage experiment

Full grid with zero LLM spend

A config weak at retrieval but strong at generation can be missed

One variable at a time

Clean cause and effect

No interaction effects; stated in the write-up

\~300 questions

Fits rate limits and the time box

Small gaps are noise; close configs are reported as ties

In-notebook retrieval, Vector Search once

Many configs, full control

Less like production; the single Vector Search run is the bridge

Jev as second judge

Cheap, fast, calibrated probabilities

No rationale; weaker on reasoning-heavy checks

### Limits we design around

Time-limited workspace → commit to Git every sessionFoundation model rate limitsJev needs network access + API key3 days, no extensionsPublic data only, never ADP work

Division of labour

## Where AI works, where you think

THE RULEEvery line in the repo is something you can explain in an interview without notes. If you can't, it doesn't ship.

Loading, chunking, embedding codeYou read and run every cell

AI drafts

you

MLflow tracing and judge wiringYou debug what breaks

AI boilerplate

you

Metric code (Recall@K, NDCG, F1)You hand-check one example

AI drafts

you verify

Experiment designConfigs, baseline, shortlist rule

ideas

you decide

Write-up and postStructure and editing only

edits

your words

Delta schema designYour strength and your interview story

100% you

Reading results, drawing conclusions

100% you

### Four moments you must be present for

**Before any code**You design the Delta schema on paper.

**Stage 1 results land**You pick the shortlist and write down why.

**Judge disagrees**You read 20 of those cases yourself.

**The last paragraph**What you learned, what you'd change.

The schedule

## Tonight plus three days

TONIGHT

- Lesson 1: tracing
- Smoke test: dataset, model list, Jev call
- Study one HotpotQA record
- Connect a Git folder

Out: **nothing blocked**

DAY 1 · DATA

- Schema on paper
- Bronze → silver load
- Chunking + embeddings
- Run the retrieval grid

Out: **retrieval metrics table**

DAY 2 · GENERATE

- Lesson 2: evaluate + scorers
- Pick the shortlist
- Traced RAG chain
- Ground-truth scoring

Out: **traced runs, EM/F1**

DAY 3 · JUDGE + SHIP

- LLM judge + Jev
- Agreement analysis
- Write-up, repo README
- LinkedIn post

Out: **published**

Behind at the end of Day 2? Drop Jev before you drop the write-up. A published project with one judge beats an unfinished one with two.

The method

## How to approach any problem like this

The project is one instance of a method you can reuse for every AI system you touch.

### Define "good" before building

Pick the metric and the ground truth first. Without them, every improvement is an opinion.

Here: Recall@K, F1, judge agreement

### Run the cheap test first

Find the part you can measure for free and use it to shrink the expensive part.

Here: retrieval grid before any LLM call

### Change one thing at a time

If two things change at once, you learn nothing about either.

Here: one config variable per comparison

### Test your measuring tool

A judge is a model too. Check it against known answers before trusting it.

Here: LLM judge and Jev vs ground truth

### Store everything as data

Results in tables can be queried, compared and audited later.

Here: every run is a Delta row

### Finish and show it

Unpublished work has no career value. Shipping is part of the job.

Here: repo + write-up + post

Career value

## What it does for you, honestly

### Where it helps

- **Proof for your positioning.** "AI Data Platform, Databricks-native" stops being a claim.
- **A concrete interview story.** Numbers, trade-offs and a validated judge. Few candidates have one.
- **Good timing.** Evaluation cost is a live topic and Jev is a month old.
- **Reusable.** This framework later measures your learning-coach flagship.

### Where it doesn't

- **It won't fix screening alone.** The published write-up does more than a resume line.
- **It doesn't train DE fundamentals.** Spark internals and Delta concurrency still need their own time.
- **It is a benchmark study.** Present it as public data, never as production work.

### What to carry into the next ten years

**Evaluation is the scarce skill**

Building a RAG app takes an afternoon now. Knowing whether it works, and proving it, is what teams pay for.

**Data engineering is the backbone of AI quality**

Eval sets, lineage and metrics tables are data problems. That is your home ground.

**Think in cost and latency, not just accuracy**

A judge that is 2% better at 400× the price is usually the wrong choice. Say that with numbers.

**Judgment beats typing**

AI writes code faster than you. Your value is deciding what to build, what to measure and what the results mean.

RAG Eval Lab · 3-day build plan · October 2026 · Figures in the hero are placeholders until Day 1.