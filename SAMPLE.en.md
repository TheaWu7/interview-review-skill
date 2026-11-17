# Mock Interview
Interviewer: Tell me about a project you’re proud of.

Candidate: I led the frontend for an AI writing feature in an online editor. Users could select text and ask AI to rewrite, polish, or continue it. We used React and TypeScript, consumed an SSE stream, and rendered generated text incrementally. I owned streaming rendering, editor integration, cancellation, and race handling. After launch, daily active penetration reached about 35%.

Interviewer: How did you handle streaming rendering performance?

Candidate: The main risks were too many React commits and re-parsing the whole generated text on every chunk. We buffered chunks and committed them in batches with requestAnimationFrame, isolated the streaming preview from the rest of the editor state, memoized completed paragraphs, and only re-·parsed the active paragraph. We also kept an AbortController for cancellation.

Interviewer: What if the user types or changes selection during generation?

Candidate: That’s a race and state-consistency problem, not just a debounce problem. Each request had a requestId and a snapshot of the document version and selection. When a new request started, we aborted the old one. When chunks arrived, we checked whether the request was still active and whether the selection was still compatible. We never let stale chunks write into a new selection.

Interviewer: Coding question: given an array and a target, return indices of two numbers that sum to target. What do you need to clarify?

Candidate: Can the array contain duplicates? Can the same element be used twice? Is there guaranteed to be one solution, or should I return any valid pair? Should I return all pairs or just one?

Interviewer: Duplicates allowed, same element cannot be reused, return any one pair, empty array if none.

Candidate: Brute force is O(n²). With a hash map from value to index, I check target - current before inserting, so I don’t reuse the same element. That’s O(n) time and O(n) space. For [3, 3] and target 6, it returns [0, 1], not [0, 0]. If multiple pairs are required, I’d confirm that first and use a different approach.

Interviewer: What is React key, and why is index risky?

Candidate: key is an identity hint for reconciliation. It tells React which new element corresponds to which old element. With index, insertion or deletion shifts identities, so React may reuse the wrong component state—input values, focus, or expanded rows. I prefer stable business IDs; index is acceptable only for static lists that never reorder.

Interviewer: Design a URL shortener. Start with clarification.

Candidate: What’s the write scale, read-to-write ratio, retention period, and whether we need custom aliases and analytics? Also, if the same long URL is submitted twice, should it return the same short link or a new one?

Interviewer: 100M new links per day, read-heavy, indefinite retention, optional aliases, basic analytics, duplicate URLs should be idempotent.

Candidate: 100M/day averages about 1,157 writes/sec, with higher peaks. I’d use a segment allocator to pre-allocate ID ranges to nodes, generate Base62 keys locally, and keep a unique constraint in the database as the final guard. For duplicate long URLs, I’d store a normalized URL hash with a unique constraint and return the existing short key. I’d shard by short-key hash, cache hot mappings in Redis, and use the database as the source of truth. For redirects, I’d default to 302 for analytics and flexibility.

Interviewer: Tell me about a time you learned something quickly for a production issue.

Candidate: We had intermittent SSE disconnects in the AI writing feature. I reproduced it with a minimal client, read the SSE and browser networking docs, and found we didn’t distinguish network errors from normal stream end and weren’t reconnecting with Last-Event-ID. I added reconnect logic, cancellation, and a test checklist. Incomplete-generation reports dropped significantly, and another team reused the pattern.

Interviewer: Any questions for me?

Candidate: How does the team measure editor performance—do you have SLOs for input latency and streaming render time? And how do you handle cache consistency and failure recovery in services like the short-link system?

# Interview Review

## 1. Question List

1. Tell me about a project you're proud of.
2. How did you handle streaming rendering performance?
3. What if the user types or changes selection during generation?
4. Coding question — Two Sum: what do you need to clarify?
5. Two Sum — solve it (duplicates allowed, no element reuse, return any one pair).
6. What is a React `key`, and why is using index risky?
7. Design a URL shortener — start with clarification.
8. URL shortener — design it (100M/day, read-heavy, indefinite retention, optional aliases, basic analytics, idempotent duplicates).
9. Tell me about a time you learned something quickly for a production issue.
10. Any questions for me?

## 2. Question Intent

#### Question 1: Project you're proud of

- **What it tests**: Experience validation — did you actually build this, and can you articulate your own contribution (not the team's)?
- **The deeper signal**: Whether you own a real, non-trivial piece of a system and can speak to its difficulty and measurable impact. The interviewer is looking for "I" ownership, technical specifics, and a result with a number.
- **What a strong candidate covers**: Business context → your specific role → architecture/data flow → the hard part → quantified result → honest reflection. You hit most of this, though the "hard part / reflection" was only lightly touched.

#### Question 2: Streaming rendering performance

- **What it tests**: Frontend performance knowledge under a genuinely hard constraint (incremental text rendering with high-frequency updates).
- **The deeper signal**: Do you understand React's rendering model well enough to know *why* many commits hurt (reconciliation cost, layout thrash), not just that they do? Can you name concrete, correct mitigation techniques rather than buzzwords?
- **What a strong candidate covers**: The actual failure modes (re-parse per chunk, excessive commits, layout thrash), and layered mitigations — buffering/batching, memoization boundaries, isolating state, rAF vs. idle scheduling, plus flush-on-idle and backpressure so UX latency doesn't regress. You covered the core well; the scheduling/flush nuance was the gap.

#### Question 3: User types / changes selection mid-generation

- **What it tests**: Race-condition and state-consistency awareness under real concurrent UI.
- **The deeper signal**: Whether you distinguish a true race (stale async writes landing after state changed) from a simple debounce problem. This is a classic "senior vs. mid" differentiator.
- **What a strong candidate covers**: Request identity + versioned snapshot, aborting the previous request, validating each chunk against current selection/version before committing, and guarding against stale writes. You nailed this — one of your strongest answers.

#### Question 4 & 5: Two Sum (clarify + solve)

- **What it tests**: Problem-solving discipline and basic algorithm fluency, but the *clarify* phase is the real tell — it shows you scope ambiguity before coding (per the framework's "ask clarifying questions" pattern).
- **The deeper signal**: Do you think about edge cases (duplicates, self-reuse, multiple solutions) before they're handed to you, and can you explain *why* the hash-map approach doesn't reuse an element?
- **What a strong candidate covers**: All the ambiguities you listed, plus the "check `target - current` before insert" trick that cleanly solves the duplicate/self-reuse problem. Excellent answer.

#### Question 6: React key / index risk

- **What it tests**: Framework internals — do you understand reconciliation, not just "add a key to silence the warning"?
- **The deeper signal**: Whether you can articulate the *concrete failure* of index keys (state association breaking on insert/delete/reorder), which proves real experience debugging it.
- **What a strong candidate covers**: Key as identity hint → index shifts identities → React reuses the wrong component instance and its state (inputs, focus, expansion), plus when index is acceptable. You covered all of this.

#### Question 7 & 8: URL shortener (clarify + design)

- **What it tests**: System design — scoping under ambiguity, back-of-envelope estimation, and trade-off reasoning at scale.
- **The deeper signal**: Can you drive a design from requirements rather than jumping to "use a hash + Redis"? Do you know the *collision problem* with naive random Base62, the standard ID-allocation solutions, and the idempotency + redirect-status trade-offs? The clarification about "same URL twice → same short link?" was a strong, senior-level question.
- **What a strong candidate covers**: QPS/storage estimation, unique-ID generation (range/segment allocator, or hash-vs-counter trade-off), collision handling, idempotency via normalized-URL unique constraint, sharding key, cache-aside with DB as source of truth, 302 vs 301, and failure modes (allocator losing a range, cache invalidation). You covered the core well; the missing pieces were failure-mode and storage-cost detail.

#### Question 9: Learned quickly for a production issue

- **What it tests**: Behavioral (STAR) — real experience, debugging methodology, learning speed, and whether you share credit honestly.
- **The deeper signal**: Can you demonstrate a concrete methodology (reproduce → isolate → research → fix → verify) rather than "I asked a senior"?
- **What a strong candidate covers**: Full STAR with a quantified result. You had strong Situation/Action, and the "another team reused the pattern" is a great signal; the Result was the weak point ("dropped significantly" without a number).

#### Question 10: Any questions for me?

- **What it tests**: Genuine engagement and whether you think about how teams operate (not just compensation/logistics).
- **The deeper signal**: Technical, specific questions signal seniority and that you're evaluating them, too. Your two questions were excellent — concrete and domain-aware.

## 3. Interview Results Analysis

- **Overall assessment**: A strong, senior-leaning frontend interview. Answers were technically accurate, well-structured, and showed genuine experience — especially the race-condition handling and the two-sum clarification discipline. Gaps are in depth-of-edge-cases (system design failure modes, quantified behavioral results), not correctness.
- **Correctness rate**: ~90% (all questions acceptable or better; no incorrect answers; 2–3 partial).

**Breakdown**:

- **Technical depth**: 8.5/10 — Strong on React reconciliation, keys, and race conditions; slightly thinner on rendering-scheduling nuance (rAF vs. idle, flush-on-idle/backpressure).
- **Problem-solving ability**: 9/10 — Excellent clarify-first discipline, clean complexity reasoning, correct edge-case handling.
- **System design ability**: 8/10 — Solid scoping, correct estimation (1,157 writes/sec), and the right core components (segment allocator, Base62, unique constraint, shard, Redis cache, 302); missing storage estimation, allocator failure modes, and cache-invalidation depth.
- **Communication**: 9/10 — Clear, structured, concise; the one weakness is the unquantified result in the behavioral answer.

**Strengths / areas to improve**:

- **Strengths**: Clarifies before coding; explains *why* not just *what*; strong mental model of React reconciliation and async race handling; asks specific, technical reverse-questions.
- **To improve**: Back-of-envelope storage/bandwidth estimates in system design; naming failure modes proactively; quantifying behavioral results with numbers; a touch more depth on scheduling strategies for high-frequency UI updates.

## 4. Error Correction

No answers were incorrect. The following were **partial** and worth strengthening.

#### Question 2: How did you handle streaming rendering performance?

- **Your answer**: Batching commits with `requestAnimationFrame`, isolating the streaming preview state, memoizing completed paragraphs, re-parsing only the active paragraph, and an `AbortController`.
- **The gap**: Strong on *what*, lighter on the *scheduling trade-off*. Batching with rAF caps update rate at frame rate but says nothing about what happens when the user has scrolled away, the tab is backgrounded, or chunks arrive faster than the render loop drains.
- **Stronger answer**: Add the scheduling layer — rAF for visual commits, but flush on idle or on chunk-boundary timeout so latency doesn't regress; consider virtualizing long outputs; drop/coalesce work for off-screen paragraphs; discuss backpressure (buffer growth → memory) and why you bound the buffer.
- **Advice**: When asked about performance, structure the answer as "failure mode → technique → trade-off." Always name the trade-off of each optimization.

#### Question 8: URL shortener design

- **Your answer**: ~1,157 writes/sec, segment allocator + local Base62 + DB unique constraint, normalized-URL hash for idempotency, shard by short-key hash, Redis cache, DB as source of truth, 302 for analytics.
- **The gap**: The design is correct but stops short of the numbers and failure modes that separate "good" from "strong" system design answers.
- **Stronger answer**: Estimate storage (~100M/day × ~1KB × retention → TB-scale, mention tiering/compaction); explain what happens if an allocator node crashes mid-range (you lose a range — safe, just wasteful — or you persist checkpoints); cover cache invalidation (short TTL vs. delete-on-write) and hot-key/celebrity-link skew; state the 302-vs-301 rationale explicitly (302 lets you update target + track analytics at the cost of one extra hop); mention rate limiting and URL abuse/security.
- **Advice**: For system design, adopt the fixed pass — *scope → estimate (QPS/storage/bandwidth) → high-level → deep-dive on one component → failure modes*. Your answer skipped the storage estimate and failure-mode pass.

#### Question 9: Learned quickly for a production issue

- **Your answer**: Reproduced with a minimal client, read SSE/browser-networking docs, found we conflated network errors with normal stream-end and weren't reconnecting with `Last-Event-ID`, added reconnect + cancellation + test checklist, reports dropped "significantly," team reused the pattern.
- **The gap**: STAR is mostly there, but the Result isn't quantified.
- **Stronger answer**: Replace "significantly" with a number ("incomplete-generation reports dropped ~70% over two weeks," or "reconnect success rate went from ~60% to ~98%"). The "another team reused the pattern" line is excellent — keep it and pair it with a metric.
- **Advice**: Before the interview, pre-write one quantified result per project story. A specific number is what makes an anecdote feel real.

## 5. Summary

#### 1. Topics to strengthen

- **Rendering & scheduling**: `requestAnimationFrame` vs. `requestIdleCallback` trade-offs, flush-on-idle, backpressure/buffering, list virtualization, `content-visibility`, time-slicing for long blocking work.
- **React internals**: deeper reconciliation mechanics (fiber, lanes/priority, `useMemo` vs. `React.memo` boundaries), controlled vs. uncontrolled components, when `key`-less/index-keyed lists are safe.
- **System design fundamentals**: back-of-envelope estimation (QPS, storage, bandwidth) as a first-class step; ID-generation schemes (range/segment allocator vs. Snowflake vs. hash+collision-retry vs. DB counter); cache-aside + invalidation (TTL, delete-on-write, write-back); sharding key selection and hot-key skew; 301 vs. 302; rate limiting and abuse handling.
- **Behavioral storytelling**: pre-written quantified results for every project story; explicit STAR with a measurable "R" and one-line reflection.

#### 2. Strategic advice

1. **Put a number on every result.** Your project and incident stories are technically rich but the outcomes are qualitative ("35%" was good; "dropped significantly" was not). For each of your 2–3 core stories, lock in one metric you can state in a sentence.
2. **Lead system design with the estimate, not the architecture.** You have the intuition (1,157 writes/sec was right and impressive); make it explicit and then hang the design off it. Interviewers read an un-prompted storage/bandwidth estimate as seniority.
3. **Name failure modes before you're asked.** You handle races brilliantly when prompted — do the same proactively in system design (allocator crash, cache staleness, hot key). One unprompted "here's what breaks and how I handle it" per design earns disproportionate credit.
4. **Keep asking technical reverse-questions.** Your two closing questions were a genuine strength; they signal you're evaluating the team's engineering maturity, which is exactly what senior interviewers look for.
