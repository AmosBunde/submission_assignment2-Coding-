# Optional Analysis: What to Expect

These supplementary descriptions are for reflection, not graded scoring. Use **your** notebook numbers as evidence. The notes below describe **qualitative expectations and reasoning**—not target accuracies—so you can check whether your interpretation is on solid conceptual ground.

**Overall expectation:** Test-time compute usually helps on hard math, but *how* you spend API calls often matters as much as *how many*. Expect tradeoffs among accuracy, cost, latency, and failure modes—not a single “best” method for every budget.

---

## Task 1: Quantitative Analysis (Optional)

1. **Accuracy Rankings**

   **What to do:** Rank zero-shot, majority voting (report the best $n$ you tried), LLM voting, and self-improvement using the accuracies from *your* run.

   **What to expect (reasoning, not a fixed order):**
   - A single-sample baseline (zero-shot / majority with $n=1$) is usually the floor.
   - Parallel sampling (majority) often improves over the baseline, but gains can saturate.
   - An aggregator that *reads* multiple solutions (LLM voting) can beat raw majority at a similar sample budget when the correct answer is present but not the plurality.
   - A short critique–revise loop (self-improvement) can land between—or occasionally above—many-sample majority, because feedback redirects a wrong first attempt instead of only averaging correlated samples.
   - Absolute ranking can shift with model, temperature, and sampling noise. Prefer explaining *why* your order makes sense over matching a canonical leaderboard.

2. **Cost-Effectiveness**

   **What to do:** Compute accuracy gain **per additional API call** for majority voting ($n=1 \to n=16$) and for self-improvement (typically 3 calls: generate → critique → regenerate). Compare which gives better returns in *your* data.

   **What to expect:**
   - Majority voting often shows **modest** gain per extra call once samples are correlated (same model, same mistakes).
   - Self-improvement often shows **stronger** gain per extra call when the critic catches fixable errors, because two follow-up calls can change the reasoning path.
   - Also check the *peak* along the majority curve (best $n$), not only $n=16$—the last few samples may add cost with little benefit.

3. **Self-Improvement Analysis**

   **What to do:** Report (a) improved vs. zero-shot (and, separately, original vs. improved *within* the RLEF cell), (b) which error types show up most in critiques, (c) what fraction of problems flipped incorrect → correct (and any regressions).

   **What to expect:**
   - Prefer the **within-pipeline** delta (original → improved in the same RLEF run) when judging the method. Comparing to a separate zero-shot cell is useful but noisy—different samples, same model.
   - On AIME-style problems, critiques often highlight **completeness** (missing steps / unfinished arguments) and **conceptual setup** mistakes more than isolated arithmetic; logical gaps and calculation errors still appear.
   - Expect a **subset** of problems to improve, not all; a healthy run often has few or no regressions (correct → incorrect), but that is not guaranteed if the critic is wrong or overconfident.

---

## Task 2: Method-Specific Insights (Optional)

1. **Majority Voting**

   **What to expect:**
   - Plot or list accuracy vs. $n$. Look for where **diminishing returns** begin: the $n$ after which accuracy flattens or wobbles instead of climbing.
   - The **maximum** need not be at the largest $n$. When errors are shared across samples, more votes mostly reinforce the same wrong answer.
   - Reasoning to articulate: independence assumption fails → majority is not a free lunch.

2. **LLM Voting**

   **What to expect:**
   - At the same generation budget (e.g. $n=16$), LLM voting often **matches or beats** majority because selection can favor a correct minority solution.
   - Cost is nearly the same as majority at that $n$, plus **one** aggregation call—so a large accuracy gap (if you see one) is an argument for smarter use of the last call, not just more samples.
   - If LLM voting does *not* help in your run, ask whether candidates were too similar, or whether the aggregator prompt failed to compare carefully.

3. **Self-Improvement**

   **What to expect:**
   - **Vs. parallel test-time compute:** majority / LLM voting spend budget in parallel; RLEF spends it sequentially (solve → critique → revise). RLEF can be competitive at low call count; large parallel budgets (especially with a strong aggregator) may still win on raw accuracy.
   - **Feedback vs. independent samples:** feedback is targeted and often efficient per call, but serial and critic-dependent. Independent samples parallelize easily but waste budget when samples are highly correlated.
   - **Error categories that guide improvement:** conceptual and completeness feedback is usually most *actionable* for regeneration (it tells the model what to change). Calculation flags help mainly when the plan was already right. Prefer specific, category-tagged critiques over “try again.”

---

# My Answers (from my 30-problem run)

## Task 1

### 1. Accuracy Rankings
1. LLM voting: 63.3% (19/30)
2. Self-improvement: 60.0% (18/30)
3. Majority voting at n=16: 46.7% (14/30)
4. Zero-shot: 40.0% (12/30)

Majority voting at n=1 scored 33.3%. It is the same method as zero-shot on a different random sample, so the 6.7-point gap between the two (two problems) is sampling noise, and any difference of one or two problems on this 30-problem set should be read the same way.

### 2. Cost-Effectiveness
Majority voting from n=1 to n=16 gained 13.3 points (33.3% to 46.7%) for 15 extra API calls, about 0.9 points per extra call.

Self-improvement gained 26.7 points over its own first attempt (33.3% to 60.0%) for 2 extra calls, about 13.3 points per extra call, roughly 15 times the return of majority voting. Measured against the separate zero-shot run (40.0%), the gain is 20 points, still 10 points per extra call.

Self-improvement gives far better returns on additional compute in this run, with one caveat: 7 of its 8 gains came from finishing answers that the 4,096-token output cap had cut off, not from correcting mistakes.

### 3. Self-Improvement Analysis
Self-improvement reached 60.0% against 40.0% for zero-shot, a 20-point gain (26.7 points against its own first attempts).

The most frequent error type was completeness. 17 of the 30 zero-shot answers reached the 4,096-token output cap before writing a final answer, and of the 8 problems self-improvement fixed, 7 had a first attempt with no boxed final answer. Only 1 fix corrected an actual mistake in a finished solution, so logical, mathematical and conceptual errors were rare by comparison in this run.

8 of 30 problems improved (26.7%), which is 8 of the 20 problems the first attempt got wrong (40%), with zero regressions.

## Task 2

### 1. Majority Voting
No clear diminishing returns appeared up to 16 samples. Accuracy was 33.3%, 33.3%, 36.7%, 40.0% and 46.7% for n = 1, 2, 4, 8 and 16, and the largest step was the last one (8 to 16, +6.7 points).

The flat step from n=1 to n=2 is built into the method rather than a sign of saturation: with two samples, a disagreement is a 1 to 1 tie that is broken in favour of the first sample, so the n=2 answer always equals the n=1 answer unless the first sample has no readable answer.

Each step is one or two problems out of 30, so these differences are within sampling noise, and a plateau beyond 16 samples cannot be ruled out. The maximum accuracy was 46.7% at n=16.

### 2. LLM Voting
LLM voting reached 63.3% against 46.7% for majority voting at n=16, 16.7 points (five problems) higher.

In API calls the cost is almost the same, 17 against 16. In tokens it is not: the extra aggregation call carries all 16 solutions in its input, so that one call is roughly as long as the 16 sampled solutions put together, and far more expensive than a single sampling call.

A likely reason for the gap is the same output cap: the aggregator restates a solution in full and can complete one that was cut off, while majority voting can only count answers that were finished. This was not measured directly for LLM voting, so it remains a hypothesis.

### 3. Self-Improvement
Self-improvement gave the best accuracy per API call: 60.0% with 3 calls, against 46.7% with 16 for majority voting and 63.3% with 17 for LLM voting.

Its advantage over independent samples is that each call builds on the previous attempt, so it can finish or correct one specific solution. Its disadvantages are that the three calls must run in sequence, which made it the slowest section of the run, and that a critic can talk the model out of a correct answer, although no regressions occurred in this run. Independent samples run in parallel and cannot regress, but they share the same blind spots, including the same tendency to run past the output cap.

The error category that mattered most for guiding improvement was completeness: 7 of the 8 fixes completed an unfinished first attempt, and only 1 corrected an actual mistake.
