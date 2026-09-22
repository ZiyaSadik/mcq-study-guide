# MCQ study guide

Verified against the prompt, measurement, and retrieval articles, and against the instructor’s topic list:

- few-shot versus grounding
- when a result is reproducible
- model comparison and the model decision
- deterministic code versus the model
- embeddings, chunking, and the vector store
- why chunks and queries use the same embedding model
- chunk size, overlap, and structural chunking
- what a chunk row stores
- cosine similarity versus cosine distance
- exact search versus HNSW
- sparse, dense, and hybrid retrieval

Questions about `policy.md` dollar amounts, meal limits, and gym memberships are not in this set. That file is only a lab document.

Each answer is the rule the article states. Distractors are the failure modes from those same articles.

---

## 1. Few-shot versus grounding

Few-shot specifies a judgment by showing an instance. Grounding is a checkable procedure: a source, a restriction to that source, a citation, and a legal way to say the source does not contain the answer. Examples do not create grounding, and a citation in the prompt does nothing if the schema has nowhere to put it.

**Q1.** Few-shot examples are the right tool when:

A. The JSON shape already has a validator
B. A judgment boundary is easier to show than to state
C. You want the twelve scored cases to look familiar
D. The model should repeat the source document

**Answer: B.** Use them for boundaries: a silent field, a contradiction, a superseded document, instruction-like content, and a case outside the task. Structure is already enforced by the schema.

**Q2.** Few-shot is the wrong tool for structure because:

A. Examples cannot contain JSON
B. The schema and the validator already enforce shape, so a full expected object spends input tokens on something you already get
C. Providers reject examples longer than one field
D. Structure can only be taught with chain-of-thought

**Answer: B.** Show only the fields the example is teaching.

**Q3.** Four full policy examples on every request are:

A. Free after the first call
B. A permanent addition to input tokens on every call, including cases the examples do not help
C. Charged only when the model copies them
D. A substitute for the constraint section

**Answer: B.** Each example has to justify its cost. When cost per case is over budget, look at the examples section first. A stable prefix can be cached only if nothing above the examples changes between requests.

**Q4.** Which examples are worth the tokens?

A. A clean, typical policy with every field present
B. Edge cases: silence, self-contradiction, a superseded revision, imperative text aimed at a reader, and a document outside the task
C. Five typical cases and no edge cases
D. The gold labels from the twelve-case set

**Answer: B.** The typical case is the one the model already handles. Two edge examples teach more than five typical ones.

**Q5.** An example is copied from one of the twelve cases you score. What is now true?

A. The metric measures production more fairly
B. That case is no longer evidence. The score can rise while production does not, and the harness will not report it
C. Surface leakage becomes impossible
D. The scorer detects the overlap

**Answer: B.** Keep a separate example pool from the start. Once a scored case has been used as an example, the only clean fix is that it no longer counts as evidence.

**Q6.** Examples leak in two directions. Those directions are:

A. Input tokens and output tokens
B. Outward: surface detail from the example appears in an answer about a different document. Inward: examples taken from the evaluation set contaminate the measurement
C. System layer and user layer
D. Dense retrieval and lexical retrieval

**Answer: B.** Distinctive titles, section styles, and entity names make outward leakage a text search. If “Jersey” or “Section 4” appears in an extraction whose document contains neither, the example is the source.

**Q7.** A wrong output is shown and labeled “bad,” with no corrected output for the same input. What often happens?

A. The model reliably avoids that output
B. The wrong output gets imitated, because the example is the most concrete thing in the prompt
C. Validation rejects the example
D. The scorer version increments

**Answer: B.** If you show a failure, show the input, mark the wrong output, say in one line what is wrong, and show the correct output. Often the better move is to show only the correct handling of that case.

**Q8.** Grounding is in place when the prompt does which four things?

A. Says “be grounded,” sets temperature to 0, requests a trace, and logs it
B. Gives a source, restricts the model to that source, requires a location for each statement, and defines how to report that the source does not contain the answer
C. Retrieves three chunks and asks whether the model is confident
D. Requires a long quotation for every sentence

**Answer: B.** Those four are mechanical, and each one can be checked. “Be grounded” is an adjective, not a procedure.

**Q9.** Where should the citation be produced relative to the claim?

A. Appended after the model has already reached a conclusion
B. Alongside the claim, or before it, so the claim follows the evidence
C. Only in the reasoning field
D. Only when a human later asks for it

**Answer: B.** A citation added afterward tends to decorate a conclusion the model already reached.

**Q10.** The instruction to cite is worth little unless:

A. Temperature is 0
B. The schema has a field for the citation next to the value. Otherwise the model appends it to the value or drops it
C. The example section shows a full object
D. The citation is a quoted span

**Answer: B.** Citation is a schema decision before it is a prompt decision. Cite a section identifier. Quoted spans multiply output tokens, and output tokens are the expensive side of the bill.

**Q11.** Which grounding failure survives ordinary review?

A. `present` with no section, which validation rejects
B. A true statement, carrying a real section identifier, that the cited section does not actually say
C. Invalid JSON
D. An enum value outside the schema

**Answer: B.** Reviewers check whether the answer is right. The check that catches this asks whether the cited section exists and contains the claim. The model can know the fact from training and point at a plausible section.

**Q12.** “If you cannot name a section that contains the value, the field is absent, whatever you may know about policies of this kind.” That sentence prevents:

A. Transport retries
B. A system that reports what is probably true instead of what the document says
C. A missing template variable
D. Lexical search

**Answer: B.** In a review packet, those are different claims.

**Q13.** A citation-correctness metric, for a field with `status = present`, checks:

A. That the `section` string is non-empty
B. That the named section actually appears in the source document
C. That the reasoning trace mentions the section
D. That the quotation is at least twenty characters

**Answer: B.** A citation merely being present is not enough. This metric is deterministic and does not call a model.

**Q14.** Which pairing is right?

A. Few-shot teaches the schema; grounding is a vague quality target
B. Few-shot shows a boundary prose cannot usefully define; grounding is the source, the restriction, the citation, and the absence path
C. Few-shot replaces gold labels; grounding replaces the schema
D. They are the same technique under two names

**Answer: B.**

---

## 2. When a result is reproducible

A result is reproducible only when the exact text and the exact model can be recovered together. Neither half is enough.

**Q15.** Months later, someone asks why a case came back the way it did. You can answer that only if you know:

A. Which prompt file is in the repository today
B. Exactly which text was sent, and which pinned model received it
C. The engineer who wrote the prompt
D. The mean latency of the run

**Answer: B.** The file may have been edited, a template variable may have been empty, or the request may have been assembled at the call site.

**Q16.** Which pair makes `prompt_version` and `model_id` mean what the report assumes?

A. A prompt id, with the model left as a moving alias
B. An immutable prompt version and a pinned model identifier, traveling together on the call record
C. Temperature 0 by itself
D. The latest file at that path

**Answer: B.** The same prompt against a moved alias is a different system. The same model against an edited prompt is a different system.

**Q17.** `summarize.v1` has already produced recorded results. A line in it is wrong. What do you do?

A. Edit `summarize.v1` in place so old records stay attached to the fix
B. Ship a new version. A record that cites `summarize.v1` has to mean the same text next month
C. Overwrite the file and rewrite `prompt_version` on the old records
D. Change only the scorer

**Answer: B.** An edited file leaves two prompts sharing one version string. Nothing errors. The measurements become fiction.

**Q18.** What makes the immutability claim checkable rather than cultural?

A. A code review comment
B. A registry that loads text by id and version and exposes a hash. If the file changes after results exist, the stored hash no longer matches
C. Temperature 0
D. Storing the prompt only in the system layer

**Answer: B.** The same registry is the place that refuses to reload a version from a changed file in the middle of a run.

**Q19.** A prompt assembled by concatenating strings at the call site fails reproducibility because:

A. Concatenation is slower than `format`
B. There is no version, no hash, and no single definition, so two call sites drift and no record identifies which text produced a result
C. The adapter rejects concatenated prompts
D. Concatenation cannot include a schema

**Answer: B.**

**Q20.** A placeholder has no value. Rendering should:

A. Substitute an empty string
B. Refuse to render. An empty document still produces a fluent answer about content that was never sent, and no error appears
C. Use the example document
D. Skip scoring and mark the case passed

**Answer: B.**

**Q21.** The system layer for one prompt version should be:

A. Different on every case, so the case id is always visible
B. Byte-identical across cases, with no case content in it
C. The place where the document is interpolated
D. Rebuilt from the user message

**Answer: B.** A stable system layer is what makes a cached prefix possible, and what makes a diff between two versions show only what changed.

**Q22.** Three kinds of content share a request. Which treatment matches them?

A. All three go in the system layer
B. Standing instruction goes in the system layer. Per-case data you generated may be interpolated. Untrusted third-party content goes inside markers in the user layer, with an explicit statement that it is data
C. Untrusted content goes in the system layer so it has priority
D. Markers are optional if temperature is 0

**Answer: B.** If you cannot tell which category a piece of content is, treat it as untrusted.

**Q23.** A procedure is full of imperative sentences, and a reviewer note says to ignore the extraction rules. Before this is a security issue, it is:

A. A token-pricing issue
B. A correctness issue. Mark the span, repeat your instruction after the span, and give a defined response for content that addresses the model directly
C. A reason to delete the document
D. Something only Week 9 retrieval can handle

**Answer: B.** The defined response in the worked example is `document_kind` `other`, with the attempted instruction described in `out_of_scope_reason`.

**Q24.** Untrusted text contains the literal closing marker. Before render you:

A. Leave it. Markers inside data are ignored
B. Escape the closing marker so the delimited span does not end early and the rest of the document does not sit where your instruction sits
C. Reject the document
D. Move the marker into the system layer

**Answer: B.** The output looks odd rather than obviously broken, which is why this is handled by escaping.

**Q25.** A score can be joined back to the exact call when both records share:

A. Latency only
B. `run_id`, `case_id`, `task`, `model_id`, `prompt_id`, and `prompt_version`
C. The engineer’s name
D. The day of the week

**Answer: B.** You should be able to take a score record and find the call that produced it without guessing.

**Q26.** Which change makes two headline scores incomparable while leaving them looking like a before and after?

A. Running the same frozen twelve cases twice
B. Adding, removing, or editing cases between runs
C. Recording median latency
D. Pinning both model identifiers

**Answer: B.** Freeze the set for the life of a comparison. Adding a case resets the comparison, or you rescore everything.

**Q27.** The scorer changes in the same commit as the prompt, and the number goes up. What do you know?

A. The prompt improved
B. Nothing that separates a more lenient metric from a better prompt. Land scorer changes separately, version the scorer, and rescore old runs when the metric moves
C. The model improved
D. The gold labels were confirmed

**Answer: B.**

**Q28.** For a retrieval result, reproducibility also requires:

A. The top cosine score only
B. The named index build: embedding model, dimension, chunking parameters, corpus manifest hash, and ingestion commit. The corpus is the source of truth, and a rebuild from source is a routine operation
C. A hand-written note that someone reloaded the index
D. The HNSW parameters from a tutorial

**Answer: B.** A hand-patched row exists nowhere in source and disappears on the next ingest.

---

## 3. Model comparison

One table per task. Quality, token usage, latency, and repairs are columns of that table. Every row names the prompt it ran.

**Q29.** Quality, tokens, latency, and repairs are reported in separate sections. What goes wrong?

A. The tables are harder to format
B. Each axis produces its own winner, and the reader keeps whichever table they saw last. The trade disappears
C. Median latency becomes illegal
D. The prompt version is forced into every cell

**Answer: B.** One table per task, because a model that wins extraction may lose triage. An average across tasks describes a system nobody is building.

**Q30.** Model B fails validation more often and looks cheaper if you price only successful calls. Case cost is:

A. The successful call only
B. The sum of every attempt on that case: the first call, transport retries, and schema repairs
C. Mean latency times a dollar rate
D. The final repair only

**Answer: B.** Failed attempts that returned tokens are part of the case. They are not spread evenly across models. A model that fails validation more often can look cheaper on successes and cost more per case.

**Q31.** The latency numbers in the report are:

A. The mean, because it uses every sample
B. The median and the maximum, with the observation count, measured from the first attempt to the final result
C. A p95, even on twelve cases
D. Time to first token of the successful call only

**Answer: B.** The mean hides the retried case. Twelve cases barely describe a distribution. Resist a p95 that a larger sample would be needed to support. A model that succeeds quickly after failing twice is not reported as fast.

**Q32.** Two models both get eight of twelve fields right. One missed four values. The other invented four. They are:

A. Equivalent, because the totals match
B. Different findings. A miss is visible. An invention is something a person may act on without seeing that it is unsupported
C. Both acceptable if latency is lower
D. Comparable only as a percentage

**Answer: B.** Keep missed and invented in separate columns. Collapsing them into one accuracy figure discards the finding a compliance reviewer cares about.

**Q33.** Qwen runs `extract.v2`, written while working with Mistral, and no Qwen-specific version exists. The row should be described as:

A. “Qwen is worse at extraction.”
B. A prompt-transfer result. It measures that prompt on Qwen. It does not measure an adapted prompt, so it does not rule Qwen out
C. A production-quality ranking
D. Unusable, so it is omitted from the table

**Answer: B.** If you tune a prompt for the second model, that is a new version. Preserve the old version. Every result records the version that actually ran.

**Q34.** A one-case gap on twelve cases, such as 11 against 10, supports:

A. A ranking you can repeat to a client as a commitment
B. Nothing you should call a difference. Report the counts and the denominator. One case is more than eight percentage points, which the set cannot resolve
C. A percentage to one decimal place
D. A claim about forty thousand cases a month

**Answer: B.** Twelve cases support direction. They can eliminate a candidate that breaks a hard constraint, or expose a failure that repeats across several cases. They do not support magnitude, a percentage, or behavior at production volume.

**Q35.** Local Ollama models in this comparison are reported with:

A. Invented commercial token prices
B. `cost_usd = 0.0`, plus input tokens, output tokens, median latency, maximum latency, observation count, repair rate, and retry or failure count
C. Mean latency as the headline
D. A blended dollar cost across tasks

**Answer: B.** Both models use `provider = "ollama"` and are distinguished by `model_id` from configuration. No model-name literal belongs at the call site. Do not add cloud credentials for this lab.

**Q36.** The limits section of the comparison has to say:

A. The winner is ready for production volume
B. There are twelve cases per task, the results are directional, transfer rows are labeled, untested combinations are named, no production-volume reliability claim is made, and local latency depends on the lab hardware
C. Percentages are more honest than counts
D. An unadapted prompt is a fair capability ranking

**Answer: B.** Write the limits as though the reader will read nothing else. A caveat left out of the document is left out of every meeting that quotes it.

**Q37.** The recommendation for a task names:

A. The model family only
B. The task, the model, the prompt version the recommendation rests on, the measured reason, and the condition that would reopen the decision
C. A single winner across summarization, extraction, and triage
D. Whichever model felt more fluent when you read the outputs

**Answer: B.** If someone changes the prompt without rerunning, the recommendation no longer rests on anything. Point at the decision record for constraints and rejected alternatives. Do not restate that record as if the report replaced it.

**Q38.** Repair rate belongs next to cost because:

A. Repairs are free on local models
B. A reader who assumes “cost per call” misses the model whose validation failures close the apparent gap
C. Repair rate replaces quality
D. Only the first attempt is billed

**Answer: B.** On a validation failure, send the error back, cap the attempts, and record every attempt. An uncapped loop against a schema the model cannot satisfy is the fastest way to hit a spend ceiling. Do not send the identical request again.

---

## 4. The model decision

The decision record holds evidence, the decision, rejected alternatives, and review triggers. Every evidence row names the prompt version. Earlier constraints are not rewritten after the results arrive.

**Q39.** A model decision made from this lab can rest on:

A. An impression from reading twenty outputs
B. A frozen case set, gold labels written from the source, deterministic scores, full attempt cost, median and maximum latency, and the prompt version on every row
C. A vendor table that separates quality from cost and omits the prompt
D. A mean latency and a percentage

**Answer: B.** The number you give will be repeated in rooms you are not in. If it came from an impression, you cannot say what would change it.

**Q40.** Which result is strong enough to drop a candidate?

A. A one-case difference on routing
B. A hard constraint that fails, such as a committed draft that promises a refund, approves or denies a claim, says the issue is resolved, or implies a final customer outcome
C. A lower median latency by itself
D. A transfer row that scored below the model the prompt was written for

**Answer: B.** Run that human-boundary check on every model you recommend. Also treat a systematic repeated failure as evidence, such as a model that never reports a document as superseded. State which models were tested.

**Q41.** Personal-data leakage, as a decision input, is checked by:

A. Asking the model whether the answer contains personal data
B. Applying the configured patterns for synthetic account numbers, national IDs, emails, and telephone numbers to free-text outputs
C. Blocking any answer that cites a section
D. A second model acting as a judge

**Answer: B.** The check is deterministic. It does not call a model.

**Q42.** You should reopen a recommendation when:

A. Someone prefers another vendor
B. The condition you named occurs: for example a missed escalation on a later scheduled run, or an adapted prompt is finally measured for a model that previously only had a transfer result
C. The mean latency changes slightly
D. The prompt file is edited in place

**Answer: B.** An unmeasured adapted prompt means the model is not recommended and is also not ruled out.

**Q43.** `docs/model-decision.md` after the run should contain:

A. New constraints written to match whichever model won
B. Evidence, decision, rejected alternatives, and review triggers, with the prompt version on every evidence row, and the constraints you set before seeing the results left as they were
C. Cloud prices for the local models
D. Only the winning percentage

**Answer: B.**

**Q44.** A published comparison you are handed omits the prompt version, quotes mean latency, and separates quality from cost. You should treat it as:

A. Ready to adopt
B. Incomplete. Ask which prompt each model ran, whether repairs are in the cost, whether latency is median and maximum from first attempt to final result, and whether a small case set is being reported as a percentage
C. Authoritative because it came from a vendor
D. Usable if the winner matches your preference

**Answer: B.**

**Q45.** Whole-object exact match as the quality number for a twelve-field extraction is:

A. The right headline, because it is one figure
B. Almost useless. Score each field, then aggregate, and report per field. Otherwise you cannot see that the effective date broke, and a change that fixes two fields and breaks one looks only like a net loss
C. Required for the join to the call record
D. The same thing as required-evidence recall

**Answer: B.**

---

## 5. Deterministic code versus the model

If a competent person, given the same facts and the same written procedure, would always produce the same answer, the answer is determined and it belongs in code.

**Q46.** The placement test is:

A. Whether the model usually gets the step right
B. Whether the same facts and the same written procedure always determine the answer
C. Whether the step is cheap in tokens
D. Whether the client asked for a model

**Answer: B.** A model that usually produces a determined answer, and occasionally produces something else, is worse than a function that produces it every time.

**Q47.** Which step stays with the model?

A. Adding business days across a holiday calendar
B. Reading a narrative and judging whether inconsistent evidence shows a prior merchant relationship
C. Reading a sanctions-list entry from the system that owns that list
D. Choosing which extracted revision is in force on a given date

**Answer: B.** The model reads. Code decides. Summarizing for a person who will act, recognizing a contradiction, and drafting text a person will review also stay with the model, because no rule encodes them.

**Q48.** Arithmetic and dates are:

A. Safe to leave in the prompt, because the model gets them right most of the time
B. Never the model’s job. A high hit rate is what lets a weekend or bank-holiday error survive a demo and a twelve-case set
C. The model’s job when temperature is 0
D. A retrieval problem

**Answer: B.** Procedural deadlines, business-day counts, policy-period boundaries, threshold comparisons, and holiday calendars are computations. The weekend case becomes a unit test once it lives in a calendar function.

**Q49.** A model has already seen a customer-master value earlier in the same request. Using the model to repeat that value is:

A. A lookup
B. A recollection. If a fact has an owning system, the fact comes from that system. There is no guarantee the model reproduces it unchanged
C. Required for grounding
D. Cheaper than a function call

**Answer: B.**

**Q50.** `select_current_version(extractions, as_of)` exists so that:

A. The two local models can vote on which document is current
B. Currency is decided in Python after a validated extraction. A wrong outcome is then either a bad extraction or a bad rule
C. The embedding can store the effective date
D. The prompt can include the holiday calendar

**Answer: B.** Do not ask the model which document is current. If version currency is scored, score the function’s result, not a free-text opinion.

**Q51.** The current revision on a date is:

A. The row where `superseded_by` is null
B. The revision effective on or before that date which has not been superseded by another revision that is itself effective on or before that date
C. The highest version string
D. The chunk with the smallest cosine distance

**Answer: B.** A successor that is published but not yet in force does not make the older revision withdrawn. A null check on `superseded_by` returns nothing in that situation, and it returns a withdrawn document when the link was never populated.

**Q52.** A routing table lives in Python and is also restated in the prompt. What follows?

A. The two copies reinforce each other
B. They eventually disagree, and the output does not say which one produced the answer. One owner remains, and the other copy is removed
C. The scorer averages them
D. The registry rejects the prompt

**Answer: B.** If the prompt mentions the rule at all, it says that code applies it.

**Q53.** Moving a determined step into code changes the bill because:

A. Python calls are billed as output tokens
B. The rule is no longer sent on every request, and the model no longer spends output tokens stating its conclusion. At tens of thousands of cases a month, that saving compounds
C. Embeddings become unnecessary
D. Repairs become impossible

**Answer: B.** A rule in a function costs microseconds and nothing per case.

**Q54.** A model-risk review asks which behavior is specified and which is learned. You can show a test that the rule holds when:

A. The rule is a sentence in the prompt and temperature is 0
B. The rule is a function with a diff, a test, a reviewer, and a release
C. Five samples at temperature 0 agree
D. The reasoning trace mentions the rule

**Answer: B.** The honest answer for a prompt rule is that it usually is applied. A wrong rule in code is a one-line change with a regression test. A wrong rule in a prompt is an experiment.

**Q55.** The failure surface after the split is:

A. One outcome called “the model got it wrong”
B. Either a bad extraction, which you check against the citation, or a bad rule, which you check against the procedure and the unit test
C. A retrieval miss only
D. A scorer bug only

**Answer: B.** A dispute manager can read the routing function and say it is wrong. They cannot do that with a paragraph whose effect is probabilistic.

---

## 6. Embeddings

An embedding is a fixed-length vector from one specific model. Similarity is the only thing it is for. It is not a summary, not invertible, and not a confidence score.

**Q56.** You send text to an embedding model and get back:

A. A short natural-language summary
B. A vector of fixed length for that model. A forty-word clause and a four-thousand-word procedure produce vectors of the same length
C. A sparse keyword list
D. A citation

**Answer: B.** The long input has to average more material into the same space. A chunk covering many subjects sits near none of them.

**Q57.** A cosine of 0.61 means:

A. 61 percent relevant, on every question, including after a model change
B. A rank within this query’s candidate set. It is not calibrated, not a probability, and not comparable across questions
C. The document is in force
D. The lexical score after min-max normalization

**Answer: B.** Scores order candidates for one query. Reading 0.61 as confidence is how a cutoff gets hard-coded and later fails with no error.

**Q58.** Someone sets 0.75 as the relevance cutoff after looking at a few scores. A later model change:

A. Preserves the meaning of 0.75
B. Moves the whole distribution. The cutoff can discard good candidates with no error and no log line
C. Is rejected at insert because the cutoff is in the schema
D. Only affects lexical search

**Answer: B.**

**Q59.** Cosine similarity ignores magnitude and asks whether two vectors point the same way. A dot product equals that cosine when:

A. Always, for any embedding model
B. The vectors are unit length. Check that. Do not assume it
C. The vectors come from different models
D. You sort with `<=> DESC`

**Answer: B.**

**Q60.** Retrieval is asymmetric because:

A. Questions are always longer than clauses
B. You want the passage that answers the question, and a question resembles other questions more than it resembles an answer
C. Cosine distance is lower-better
D. Lexical search is case-sensitive

**Answer: B.** Models built for retrieval are trained to place a question near its answer. That is a different objective from general text similarity.

**Q61.** A query prefix or instruction is applied when the corpus is embedded, and forgotten when the question is embedded. What happens?

A. Insert fails
B. Everything runs, and recall collapses. The same function and the same model embed both sides
C. Only the lexical index breaks
D. HNSW repairs it

**Answer: B.** A query embedded even slightly differently from the corpus produces no error.

**Q62.** Dimensionality is:

A. A quality dial you raise whenever recall dips
B. A cost. Storage, index size, and query time scale roughly with length. Pin the length beside the model id, and create the vector column at that length so a mismatch fails at insert
C. Equal to the chunk-size limit
D. Irrelevant if you normalize

**Answer: B.** Higher dimension is not proportionally better retrieval. The length in the migration is rendered from configuration. It is not typed by hand.

**Q63.** Embeddings are reliably weak at:

A. Paraphrase between a question and a clause that share little vocabulary
B. Negation, numeric thresholds, identifiers, version strings, and dates
C. Section titles prepended to a chunk
D. Short unrelated clauses, which they rank above the real answer

**Answer: B.** “Shall not accept” sits near “shall accept.” A 25 percent threshold sits near a 10 percent threshold. Those facts belong in metadata and are queried as predicates.

**Q64.** A test asks similarity to rank the current 25 percent clause clearly above the superseded 10 percent clause. The gap is a few thousandths. You should:

A. Tune a threshold until the assertion passes
B. Remove that assertion. Currency is a date rule in code, then a filter. Keep the test that the question ranks its answering clause above an unrelated clause
C. Repeat the version string inside the vector until the gap exceeds 0.05
D. Switch to HNSW

**Answer: B.** The model is reporting that two nearly identical sentences are nearly identical. There is no threshold that separates them, and a different embedding model does not fix that structural fact.

**Q65.** An analyst pastes a version string or a section number and expects an exact hit. Dense retrieval:

A. Is the right tool, because identifiers are high-information tokens
B. Treats the string as short and low-information, and can return thematically similar prose from the wrong document
C. Guarantees an exact match on tokens
D. Applies the currency filter by itself

**Answer: B.** That exact match is what lexical retrieval is for.

---

## 7. Same embedding model for chunks and queries

**Q66.** Stored chunks and user queries must use the same embedding model because:

A. It keeps the prompt cache warm
B. Vectors from two models are not comparable, and there is no conversion. A mismatch produces no error and destroys recall
C. The database rejects every mixed pair at insert time, even when the lengths match
D. Cosine distance requires both sides to be sparse

**Answer: B.** The model is part of the index’s identity.

**Q67.** A provider quietly changes the model behind a deployment name. Inserts and queries still succeed. What is true of the old vectors?

A. They are converted automatically
B. They are invalid. Ranking degrades in a way that looks like a badly worded question. The defense is a pinned identifier, the model recorded on the build, and a full rebuild from source
C. They remain valid if the dimension is unchanged
D. Only new documents are affected, and mixed rows are safe

**Answer: B.** A different model that happens to emit the same dimension is still silent at the database. If rebuilding the index feels frightening, that feeling is the defect.

**Q68.** Part of the corpus is re-embedded after a model change and part is not. This is:

A. A normal rolling upgrade
B. A mixed-vintage index. The join succeeds, the query succeeds, and vectors from two models are compared as though they were comparable
C. Caught by the primary key
D. Safe because `chunk_id` is stable

**Answer: B.** `chunk_id` stability is what makes the bad join succeed. The build id is what makes the mixture detectable.

**Q69.** “The same model” in this rule includes:

A. The model name only. Prefixes, normalization, and the embedding function may differ
B. The same model, the same function, and the same query-versus-passage handling at index time and at query time
C. Any model with the same vector length
D. The generation model

**Answer: B.**

**Q70.** The generation model and the embedding model:

A. Must be the same checkpoint
B. Are different roles. Generation reads retrieved text. Embedding only places text for ranking. Changing the generator does not invalidate vectors. Changing the embedding model does
C. Must have the same dimension
D. Are both stored in the vector column

**Answer: B.**

---

## 8. Chunk size, overlap, and structural chunking

A chunk is the retrieval unit and the citation unit. Those jobs pull against each other. Structure comes first. Size comes second.

**Q71.** The primary split is:

A. A fixed character count, because it is predictable
B. The document’s own numbered sections. Subdivide only a section that exceeds the configured maximum
C. One chunk per document, so the citation is always the title
D. Token windows that ignore headings

**Answer: B.** A character split cuts sentences, tables, and defined terms, most often in the normative text that matters. A document with no usable structure is a fact to record, not a reason to abandon the rule.

**Q72.** Why does size matter, given that every embedding has a fixed length?

A. A larger chunk produces a longer vector
B. A chunk that covers many subjects is averaged into one position, so it ranks moderately for every question and strongly for none
C. Size only affects lexical search
D. Size is the same thing as overlap

**Answer: B.** Retrieval wants a small, single-subject chunk. Answering wants the exception still attached to the requirement. Citation wants a unit a person can open.

**Q73.** A fixed-size cut lands in the middle of “25 percent or more,” and the next window starts at the exception. The quiet failure is:

A. The database rejects the row
B. The next chunk states the exception without the rule, has no heading, and can still receive a plausible section citation downstream
C. Cosine distance becomes negative
D. The chunk id collides

**Answer: B.** The severed threshold is obvious on inspection. The exception without its rule retrieves well and answers falsely with a real-looking citation. That is a grounding failure created at ingestion.

**Q74.** A section shorter than the maximum produces:

A. Two overlapping chunks
B. Exactly one chunk, with ordinal 0
C. No chunk, because it is too small to embed
D. A chunk whose id is a hash of the text

**Answer: B.** Overlap is not applied to a section that already fits.

**Q75.** Overlap is for:

A. Merging two documents so the index is smaller
B. Keeping a condition and the requirement it governs together when a long section has to be subdivided. It inflates the index and can fill the candidate set with near-copies of one passage
C. Raising recall until the gold passage appears in several chunks
D. Replacing structural boundaries

**Answer: B.** Apply a modest overlap only at boundaries inside a subdivided section. Record it as configuration. Do not move it because a metric improved. That improvement often means the gold text now exists in more than one chunk, so a question that needed two sources got worse while the headline went up.

**Q76.** A minimum size merges the tail of one document with the head of the next. What validates it?

A. The primary key
B. Nothing. Both halves are real text, and half the chunk now carries the wrong document id and version
C. The embedding dimension check
D. The lexical match predicate

**Answer: B.**

**Q77.** No chunk may contain:

A. A heading in `embed_text`
B. Text from more than one section, or text from more than one document
C. An effective date
D. An ordinal of 0

**Answer: B.**

**Q78.** Chunking parameters sit on the build record because:

A. They are a prompt
B. The index is a function of those parameters as well as of the model. Two builds can be compared, and a query that mixes them is detectable
C. Postgres requires them as a primary key
D. They change the cosine operator

**Answer: B.**

---

## 9. What is stored in a chunk

What you embed is not what you cite. Metadata needed at query time is on the chunk row, because the chunk is the row the retriever returns.

**Q79.** `embed_text` and `text` differ in that:

A. They are the same string
B. `embed_text` prepends the document title, version, and section heading. `text` is the unmodified section and is what a citation resolves to
C. `text` includes the heading, and `embed_text` is the heading only
D. The vector replaces both

**Answer: B.** The prefix is applied at ingestion, not improvised at call sites. A chunk that begins mid-sentence has no subject in its vector and ranks for nothing.

**Q80.** `text_sha256` is computed over:

A. `embed_text`, including the prepended heading
B. `text` alone
C. The chunk id
D. The whole document file

**Answer: B.**

**Q81.** The chunk id is:

A. A hash of `text`
B. The insert sequence
C. `doc_id:version:section:ordinal`
D. A timestamp

**Answer: C.** It contains no content hash, no insert sequence, and no timestamp. A full re-ingest of unchanged source produces the same identifiers, which is what gold labels name. The hash stays in `text_sha256`, so an edit does not rename the chunk and silently invalidate those labels.

**Q82.** Which facts are copied onto every chunk, rather than left only on a parent document?

A. None. The vector carries them
B. Document id, version, effective date, superseded-by, jurisdiction, entity types, and section
C. The user’s question and the fused score
D. The reasoning trace

**Answer: B.** Version, date, threshold, and jurisdiction are exactly what a vector cannot represent, so a predicate has to reach them on the row that comes back.

**Q83.** The chunk row in the store also carries:

A. Only the vector
B. The chunk fields, a foreign key to the build, the vector at the configured dimension, and a generated lexical column over `embed_text`
C. A hand-edited correction, as the source of truth
D. The gold label

**Answer: B.** The lexical column is generated from `embed_text` so a query can match the title, version, and heading that were prepended for the embedding. Indexing `text` alone drops section-number and version matches, which is the thing lexical search was added to provide.

**Q84.** A positional chunk id, assigned in insert order, breaks scoring because:

A. Insert order is always stable
B. Re-ingesting with the files enumerated differently makes every id name different text. The metrics still compute, and the numbers mean nothing
C. Postgres rejects insert-order keys
D. The embedding dimension changes

**Answer: B.**

**Q85.** An unparsed effective date is stored as null because:

A. Dates cannot be filtered
B. Rejecting the document would drop it silently. A null is a recorded fact about the source, and the ingest report counts and names those documents
C. Null means the document is current
D. The parser should guess a date from a similar policy

**Answer: B.** No parsed value appears that is not in the source. Do not guess. A document that cannot be fully parsed is still ingested.

---

## 10. Cosine similarity versus cosine distance

**Q86.** Cosine similarity and pgvector cosine distance agree in this way:

A. Both are higher-better
B. Similarity is higher-better. The `<=>` operator is cosine distance, lower-better, and the two are related by subtraction from one
C. Both are lower-better
D. Distance is the lexical score

**Answer: B.** Code that sorts the wrong way still runs, still returns a full set, and returns the least relevant chunks.

**Q87.** The lab ordering for “three closest chunks” is:

A. `ORDER BY embedding <=> :query_vector DESC LIMIT 3`
B. `ORDER BY embedding <=> :query_vector ASC LIMIT 3`
C. `ORDER BY embedding LIMIT 3`
D. Order by similarity using `<=>` and take the largest values

**Answer: B.** A smaller cosine distance is a closer match. Distances in an API response are numbers, not formatted strings.

**Q88.** You treat a distance of 0.37 as “63 percent confidence” by subtracting from one. That number is:

A. A calibrated probability
B. Still only a rank within one candidate set. It is not a confidence, and it is not comparable to 0.37 on a different question
C. The BM25 score
D. Proof the chunk is current

**Answer: B.**

**Q89.** Which statement mixes the two conventions?

A. Sort similarity descending
B. Apply a cutoff that was chosen on a similarity scale, such as 0.75, directly to a distance
C. Sort `<=>` ascending
D. Use one operator consistently inside one query

**Answer: B.** The query returns rows either way. The set is simply the wrong end of the corpus, and it looks like a catastrophic retrieval failure rather than a sign error.

---

## 11. Exact search versus HNSW

**Q90.** Without a vector index, a nearest-neighbor query:

A. Returns an approximation and can miss the true neighbor
B. Computes a distance for every row in scope and returns the true nearest neighbors
C. Is illegal in Postgres
D. Uses BM25

**Answer: B.** Exact search is correct by construction.

**Q91.** HNSW or IVFFlat:

A. Returns the true nearest neighbors by construction
B. Reads a fraction of the table and returns nearly the right neighbors, faster. A chunk that exists and is genuinely nearest can fail to be returned
C. Is a filter on effective date
D. Makes vectors from two models comparable

**Answer: B.** The recall you actually get depends on build parameters and on query-time parameters. It is not one decision.

**Q92.** At the size of this week’s corpus, the default is:

A. An HNSW index, because tutorials create one during setup
B. An exact scan, which is fast enough that the approximate index buys nothing. Adopt an approximate index later, against a measurement
C. IVFFlat with the tutorial’s parameters
D. No distance operator

**Answer: B.** An approximate index adopted by default drops recall below one for reasons nobody measured. A missing chunk then gets investigated as a chunking or embedding bug.

**Q93.** A selective filter such as “current, this jurisdiction, this entity type” interacts poorly with an approximate vector index. That is a reason to:

A. Post-filter the approximate results
B. Stay on exact search while the corpus is small, and put the predicate in the same statement as the ordering
C. Drop the filter
D. Raise the similarity cutoff

**Answer: B.**

---

## 12. Sparse, dense, and hybrid retrieval

In this course, sparse means lexical term search. Dense means embedding search. They fail in opposite directions. Hybrid combines ranks, not raw scores.

**Q94.** Dense retrieval is the one that:

A. Matches the section number `6.4` because that token is rare
B. Places a paraphrased question near a clause that shares almost none of its words
C. Knows which revision is in force
D. Returns an empty list when nothing is relevant

**Answer: B.** That vocabulary bridge is the reason to use embeddings. It is also why negation, thresholds, and identifiers barely move the vector.

**Q95.** Lexical retrieval is the one that:

A. Bridges “documents we need” to “required documentation”
B. Matches a section number, a version string, a defined term, or a proper name as written
C. Returns k rows even when no term matched
D. Removes superseded revisions

**Answer: B.** It has no capacity to bridge vocabulary. A question phrased entirely in the analyst’s words will not find a clause phrased entirely in the drafter’s words.

**Q96.** Postgres full-text ranking and BM25:

A. Are the same formula, so the scores can be copied from a paper
B. Are both lexical, and they will not agree even on identical text. Postgres uses its own term-frequency functions
C. Are both dense
D. Are combined with a weight of 0.5

**Answer: B.**

**Q97.** A real analyst question contains a paraphrase and a literal section number. Running only dense retrieval:

A. Is enough, because the embedding includes all tokens
B. Predictably buries the named section behind thematically similar chunks. The result looks like an answer, not like a broken system
C. Is enough if k is 3
D. Applies the currency rule

**Answer: B.** Neither retriever is a fallback for the other. The failures do not overlap, which is the argument for hybrid retrieval.

**Q98.** Min-max normalizing both score lists and adding them with a fitted weight fails because:

A. Ranks are illegal
B. The scores are not comparable. The weight is specific to this corpus and this embedding model, and an empty lexical list still receives a 1 at the top, so a query with no lexical hits still contributes a maximum-scoring candidate
C. `chunk_id` cannot be joined
D. Distance cannot be sorted

**Answer: B.** There is no principled weight. Whatever you pick has to be refitted when the corpus or the model changes, and it will not be.

**Q99.** Reciprocal rank fusion scores a chunk as:

A. The average of cosine distance and `ts_rank`
B. The sum, across retrievers that returned it, of `1 / (60 + rank)`
C. Sixty times the dense rank
D. The maximum of the two raw scores

**Answer: B.** The constant, conventionally 60, dampens the advantage of being first, so a respectable rank in both lists beats first place in one list and absence from the other. No weight is tuned. Magnitude is discarded: an outstanding match and an adequate one at the same rank count the same.

**Q100.** Fusion can do that only because:

A. Both scores lie on the same 0-to-1 scale
B. Both retrievers query the same table and the same build, and return chunk ids built the same way. Fusion is a join on a string
C. Lexical search lives in a separate engine with its own ids
D. The model chooses the weight

**Answer: B.** A second store with its own identifiers turns fusion into a mapping. Every failed mapping is a silently dropped candidate, and the two stores drift.

**Q101.** The three widths are:

A. One k for dense, lexical, and the fused list
B. A k for each retriever, and a separate size for the fused list you keep. Retrieve wide, fuse, then truncate
C. Always three
D. Unlimited lexical and one dense result

**Answer: B.** Two lists of five rarely overlap enough for agreement to mean anything. Truncating too early throws away the agreement fusion exists to capture.

**Q102.** Ties in the fused list are broken by:

A. Whichever row the database returned first
B. `chunk_id`, so a later scoring run does not reorder tied candidates with nothing else changing
C. A random seed
D. The superseded-by flag

**Answer: B.**

**Q103.** Dense and lexical behave differently when the query’s terms appear nowhere:

A. Both return exactly k rows
B. Lexical has a match predicate, so it can return an empty list. Dense has no “no match” outcome and fills its limit from the nearest rows that remain
C. Dense returns empty and lexical fills k
D. Both raise

**Answer: B.** After a filter, dense also returns fewer than k if fewer rows satisfy the predicate. “Always k” describes an unfiltered nearest-neighbor query over a non-empty table.

**Q104.** The lexical index is built on `embed_text` because:

A. `text` is too long to stem
B. The prepended title, version, and heading are what let a query naming `6.4` or `v4.2` match at all
C. The vector column requires a `tsvector`
D. `text` includes stop words and `embed_text` does not

**Answer: B.**

**Q105.** Fusion promotes a superseded chunk that both retrievers found. That means:

A. Fusion is broken
B. Fusion is working. Neither retriever knows what a withdrawn document is. Currency is not a ranking problem
C. The weight on lexical search is too high
D. The chunk id is unstable

**Answer: B.**

**Q106.** One retriever queries a stale build id, or a query omits the build predicate. Because ids are stable across builds:

A. The join fails closed
B. The join succeeds and the fused list silently mixes two corpora
C. Postgres rejects the query
D. The hash check deletes the stale rows

**Answer: B.**

---

## 13. The vector store

The store is a table with an extra column type. Primary keys, foreign keys, nulls, and migrations still mean what they meant before the column was a vector.

**Q107.** A nearest-neighbor query against a corpus that has nothing on the subject:

A. Returns no rows and an explicit “no match”
B. Returns k ranked rows anyway, with distances a bit worse than usual, and no signal that the answer is absent
C. Raises a dimension error
D. Falls back to keyword search automatically

**Answer: B.** Deciding that the answer is not here is a job above the store. “No matching documents” after a filter and “no relevant documents” among documents that matched are different findings.

**Q108.** The connection succeeds, the migration ran, and ingestion did not. A retrieval path with no branch for that case:

A. Returns a clean empty answer
B. Generates an answer from nothing. That answer is fluent and invented
C. Rebuilds the index by itself
D. Switches to the generation model’s memory

**Answer: B.** Up, down, slow, and up-and-empty are different states. The third one needs a defined behavior.

**Q109.** A vector of the wrong length should:

A. Be padded with zeros and stored
B. Fail in the embedding function and fail again at insert, because the column was created at the configured dimension
C. Succeed, and show up later as a bad question
D. Be truncated

**Answer: B.**

**Q110.** Rows are repaired by an `UPDATE` when a chunk looks wrong. That is a defect because:

A. Updates are slower than inserts
B. The correction exists nowhere in source, cannot be explained from the repository, and vanishes on the next ingest
C. pgvector forbids updates
D. It changes the dimension

**Answer: B.** Every row is reconstructible from the corpus plus the configuration that produced it. Nothing is repaired by editing rows.

**Q111.** The build row is what makes two ingests attributable. It carries:

A. Only the latest user question
B. Embedding model, dimension, chunk strategy, overlap, corpus manifest hash, ingestion commit, and when it ran. Every chunk references that build
C. The gold labels
D. A nullable build id, filled in later by hand

**Answer: B.** Ingestion writes a new build rather than mutating an existing one. No chunk row has a null or unmatched build id.

**Q112.** Two ingests of unchanged source under different build ids should produce:

A. Different chunk ids, because the build id is inside the chunk id
B. Identical chunk-id sets. If they do not, the identifiers are not stable
C. Identical build ids
D. A new hash inside every chunk id

**Answer: B.**

**Q113.** The store’s connection configuration:

A. Is committed next to the migration
B. Comes from the environment and stays out of the commit, the same discipline used for model credentials
C. Is hard-coded in the retriever
D. Is stored in the vector column

**Answer: B.** The database is infrastructure the service depends on. It has its own lifecycle. It is not started by hand as part of a request.

---

## 14. Filtering is not ranking

Ranking orders candidates by degree. Filtering removes them by fact. A document either was in force on the review date or it was not. A preference can be outvoted. A fact cannot.

**Q114.** The review date is 15 March. v4.2 has been in force since November. v4.3 is already published and becomes effective on 1 July. The analyst is entitled to:

A. v4.3, because a higher version reads as a better answer
B. v4.2. No similarity score makes a policy that is not yet in force the right answer
C. Both, fused
D. Whichever chunk is nearest

**Answer: B.** Boosting recent documents, or adding “use the current version” to the query text, turns a determined answer into a preference. A strong lexical match on the withdrawn text can outvote the boost.

**Q115.** Retrieval should apply currency by:

A. Reimplementing the date rule in SQL because the ingest type differs from an extraction
B. Projecting chunk metadata into the shape `select_current_version` already accepts, and delegating. This module owns the projection and owns no version logic
C. Asking the generator which version sounds current
D. Sorting by version string

**Answer: B.** A second implementation of one rule will diverge.

**Q116.** A document whose effective date could not be parsed:

A. Is treated as effective today
B. Cannot be placed on the supersession chain. The filter decides explicitly whether undated documents are excluded and counted, or included and flagged
C. Is dropped by three-valued logic, with the report still saying every document was searched
D. Is current because `superseded_by` is null

**Answer: B.** Comparing a null date yields unknown, not false. A null must not disappear quietly.

**Q117.** Pre-filtering means:

A. Take the top k, then drop rows that fail the predicate
B. Put the same predicate in the same statement as the ordering, so a request for twenty means twenty candidates that already satisfy the facts
C. Filter only the dense retriever
D. Filter after fusion

**Answer: B.** Post-filtering returns however many of those twenty happened to be current, which might be four. Recall then depends on how many superseded near-duplicates crowded the top. The loss is worst on the documents with the longest version histories. Candidates that ranked just below those duplicates never existed in the result, and nothing in a post-filtered list tells you they were missing.

**Q118.** The two retrievers are filtered:

A. Independently, with a stricter predicate on lexical search
B. Identically, with the same filter specification, before fusion
C. After fusion, so each retriever can reward a chunk the other never saw
D. Only when the lexical list is empty

**Answer: B.** If only one list is filtered, fusion rewards a chunk for appearing in exactly one list, for a reason that has nothing to do with relevance.

**Q119.** The filter specification carried with the query and returned with the result includes:

A. The raw vectors
B. Build id, as-of date, optional jurisdiction, optional entity type, and an explicit flag for undated documents
C. A similarity cutoff
D. The model’s reasoning

**Answer: B.** An empty result is a correct outcome for a jurisdiction you do not hold, or a date before any document was in force. Return nothing, and carry the filter that emptied the set, so the next layer can tell “no matching documents” from “no relevant documents.”

**Q120.** v4.2 carries `superseded_by = v4.3`, and v4.3 is not yet effective on the as-of date. A predicate of `superseded_by IS NULL`:

A. Correctly keeps v4.2
B. Excludes v4.2 and can leave no current version of that policy at all
C. Is the definition used by `select_current_version`
D. Is equivalent to two date comparisons

**Answer: B.** Both halves of the date rule matter. Dropping the “successor must itself be in force” half returns nothing useful here. Dropping the “effective on or before the date” half returns a policy that is not yet in force.

---

## 15. Retrieval in more than one step

One query assumes the analyst’s wording lands near the corpus, and that the whole answer sits in one region. Both assumptions fail. A second retrieval is informed by the first. Code decides what runs.

**Q121.** A retrieved clause says to apply Section 6.4, and 6.4 is not in the candidate set. The right mechanism is:

A. A better embedding model
B. A coverage check: code pattern-matches cross-references in text already retrieved and issues a specific next query. That check costs no model call
C. Asking the model whether it has enough context
D. Raising k until the score feels sufficient

**Answer: B.** The downstream answer would otherwise be confident, cited, and wrong, because the cited section really does state the rule whose exception is missing.

**Q122.** A rewrite may:

A. Insert a jurisdiction, an entity type, or a percentage that makes the query more natural
B. Turn the analyst’s words into the corpus’s words. It may add phrasing. It may not add facts. Facts stay in the filter specification
C. Change the as-of date
D. Replace the filter

**Answer: B.** Supplying the surviving section titles helps the rewrite use the corpus’s vocabulary. An acceptance check can assert that no rewrite introduced a jurisdiction token. An instruction that merely says “do not add facts” is only a request.

**Q123.** One sentence asks whether 6.4 applies and what documents are required. Decomposition:

A. Splits it into three paraphrases of the whole sentence
B. Produces separate steps, each with its own query text and its own filters, retrieved independently. The plan is data you can print and compare
C. Averages the two topics into one embedding
D. Replaces filtering

**Answer: B.** High overlap between step result sets is the sign that decomposition restated one question. The cost tripled and the candidate set still holds one passage.

**Q124.** Within a step, and across steps, you combine results by:

A. Reciprocal rank fusion in both places
B. Reciprocal rank fusion inside a step. Across steps, the union, keeping each chunk’s best within-step fused rank, with ties broken by chunk id
C. A weighted sum in both places
D. Keeping only the last step

**Answer: B.** Steps target different sub-questions. Fusing across them penalizes a chunk for being absent from a list that was never looking for it.

**Q125.** The loop is bounded by:

A. The model’s judgment that it has enough
B. Configured constants: a maximum number of passes, a maximum number of steps, and a maximum number of retrievals. Stopping because a bound was reached is a recorded outcome
C. A cosine cutoff of 0.75
D. The spend ceiling alone

**Answer: B.** An unbounded loop that continues until something feels sufficient is a cost and latency incident. The spend ceiling will stop it, which means enforcement worked and the design failed. `stop_reason` is a small set: coverage satisfied, pass limit, step limit, or empty after filter. A run that stops at the pass limit has a known gap. It is not a broken run that hides the gap.

**Q126.** Coverage, before another pass, is decided by code asking:

A. Whether the model feels ready to answer
B. Whether every step returned a non-empty filtered set, whether named sections are present, and whether any retrieved chunk cross-references a section that is absent
C. Whether the fused score exceeds 0.6
D. Whether five chunks have been collected

**Answer: B.** A model asked whether it has enough context will generally say yes.

**Q127.** Who drives the loop?

A. The model selects tools and calls the store
B. The model contributes rewrite and decomposition text. Code decides which steps run, in what order, under which filters, whether another pass happens, and when to stop
C. The database trigger
D. Fusion

**Answer: B.** Nothing selects a tool and nothing acts. When control later moves to the model, that is a change in who is driving.

**Q128.** A multi-step run that keeps only the final merged candidate set cannot be explained because:

A. Chunk ids are random
B. The plan, each step’s query and filter, the candidates, the coverage results, and the stop reason were not recorded. The rewrite was generated, and temperature 0 does not make it deterministic
C. Fusion forbids logging
D. The gold labels contain the plan

**Answer: B.** Without that record, a miss cannot be attributed to the rewrite, the decomposition, the filter, or the retriever.

---

## 16. The prompt as a specification

A production prompt is written once and run against inputs nobody has read. Ambiguity that a person would resolve by asking becomes a coin flip.

**Q129.** The stable section order is:

A. Constraints, task, output, input, failure
B. Task, input, constraints, output, and what to do when the task cannot be completed
C. Role, examples, schema, temperature
D. Output, examples, task, input

**Answer: B.** A reviewer knows where the constraints are. A diff of the output section stays a diff of the output section. A missing section is a gap in a familiar shape.

**Q130.** Case text goes inside explicit markers because:

A. Markers reduce output tokens
B. The model otherwise has no reliable way to separate your instruction from imperative sentences in a procedure or a customer message
C. JSON mode requires markers
D. Markers replace an out-of-scope path

**Answer: B.** This is a correctness requirement. The security and adversarial treatments come later.

**Q131.** Which constraint can be checked from the output alone?

A. “Be accurate and do not make anything up.”
B. “Do not hallucinate.”
C. “Draw every statement from the marked text and cite the section heading.”
D. “Use your best judgment.”

**Answer: C.** A constraint that fails that test is a wish. Two or three checkable constraints beat a dozen aspirational ones. State them positively: name where the material comes from, or what the output must contain.

**Q132.** A document does not state a threshold, and the model returns 25. The usual cause is:

A. Temperature above zero
B. The prompt implied that a number was the expected shape and gave no legal representation for absence
C. The embedding filled the gap
D. The field was an enum

**Answer: B.** Absence, ambiguity, and out-of-scope are three findings and three human actions. How they are encoded is a schema decision.

**Q133.** A bereavement notice is handed to a dispute-summary prompt that describes only the successful path. The model will:

A. Return an empty string
B. Map it onto the nearest summary, with the same confidence as a correct one
C. Fail validation automatically
D. Skip the case because it has no deadlines

**Answer: B.** Name the out-of-scope condition and state what to return. The failure becomes a routing decision.

**Q134.** A prompt grows past a page because every observed failure added a line. The usual fix is:

A. A stronger negative instruction at the top
B. Restructuring. A constraint is doing work that belongs in the output specification or in the case set
C. A higher temperature
D. Moving the prompt into the user message

**Answer: B.** Pile-up creates unnoticed contradictions and spends input tokens on every call.

**Q135.** Where a document states a version or an effective date, and where it indicates it has been superseded, the summary prompt:

A. Leaves both to the model’s general knowledge
B. Requires both version and effective date, and requires the superseded status to be said before anything else
C. Asks the model to pick the latest revision it remembers
D. Drops the old revision silently

**Answer: B.** This week that is a reading instruction. The same failure shape later becomes a retrieval filter. Do not resolve a contradiction in the document. Report both readings.

---

## 17. Structured outputs

The schema is the contract. The prompt is how you obtain it. Everything downstream imports the schema.

**Q136.** The output description in the prompt should be:

A. Typed by hand beside the Pydantic model
B. Generated from the Pydantic v2 model, so a renamed or removed field cannot linger in the prompt and get discarded on parse
C. Owned by the scorer
D. Different for each case

**Answer: B.** Two hand-maintained descriptions drift, and the drift is invisible until a field is always absent or never validates.

**Q137.** One provider can constrain decoding to your schema. The other returns unconstrained text. The harness:

A. Deletes validation, because the strong provider makes it redundant
B. Keeps parse, repair, and recording on every path. A stronger guarantee reduces how often that path runs. It does not let you delete it
C. Uses only the strong provider’s errors as the schema
D. Skips repair on the weak provider to save tokens

**Answer: B.** Check the pinned model versions rather than assuming. The three levels are: decoding constrained to the schema; syntactically valid JSON that may not match the schema; and instruct-then-validate. Build for the weakest you support.

**Q138.** A value a downstream system branches on is:

A. Free text, so the model can explain it
B. An enum that includes the trouble values as well as the success values
C. A float confidence
D. Omitted when the case is out of scope

**Answer: B.** `card disputes` is not `card_dispute`. If the only legal outputs describe cases the task handles, out-of-scope has nowhere to land.

**Q139.** `status = present` arrives with a value and no section. Validation:

A. Stores it and cites “unknown section”
B. Fails. Present requires both a value and a section
C. Coerces the field to absent
D. Passes, because the value is enough

**Answer: B.** The repair attempt carries that message. The model then either produces a citation or the case is recorded as a failure. `0` and `false` are values. The check is “missing,” not “falsy.”

**Q140.** `status = absent` arrives with a value. Validation:

A. Keeps the value and drops the status
B. Fails. Absent must not carry a value
C. Treats the field as ambiguous
D. Passes if section is null

**Answer: B.**

**Q141.** `status = ambiguous` is valid when:

A. The value is null and the note is empty
B. A note describes the conflict. A value may be present. The note is what the validator requires
C. The model picked one of the two readings
D. `document_kind` is `other`

**Answer: B.** The worked example reports one reading in `value`, cites a section, and describes the unresolved conflict in `note`. Collapsing absent, ambiguous, and not-found into one null destroys the information the extraction existed to produce.

**Q142.** `extra = forbid` means:

A. Lower latency
B. An invented field is an error rather than something silently dropped. The invented field is often where the model put the real answer
C. Repair is disabled
D. Citations are stripped

**Answer: B.**

**Q143.** A threshold field is a float whose name states the unit because:

A. Floats embed more accurately
B. “twenty-five percent,” “25%,” and “0.25” cannot be consumed by the code that branches on the field. The type settles the question once
C. JSON cannot represent strings
D. The scorer only compares floats

**Answer: B.**

**Q144.** `document_kind = other` with no `out_of_scope_reason`:

A. Passes, and the extraction fields are invented so the object looks complete
B. Fails validation. Out of scope is a legal, explained output
C. Is rewritten to the policy kind
D. Skips the repair path

**Answer: B.**

**Q145.** A schema that is deeply nested, has a huge enum, or asks the model to echo the source document will:

A. Improve adherence
B. Reduce how reliably the model conforms, and inflate output tokens. The usual cause of a growing schema is one prompt doing two tasks
C. Be required for citations
D. Remove the need for repair

**Answer: B.** Prefer a flat structure with a handful of required fields. Never ask the model to echo input you already have.

**Q146.** A validation failure is:

A. A signal to resend the same request
B. Data. Return the error for a capped repair, write a call record for every attempt, and watch the failure rate. One failure in eight says something about the schema
C. Discarded so the case cost stays low
D. A reason to call a judge model

**Answer: B.** The repair says to return a corrected response and not to change fields the error does not concern. Both attempts count toward the case and toward the spend ceiling.

---

## 18. Reasoning

Chain-of-thought does not open the model. The final answer is generated after intermediate tokens and attends to them. That is the whole mechanism.

**Q147.** Chain-of-thought helps when:

A. The value is already in the input and only needs to be copied
B. The answer depends on combining several facts that have to be brought together first
C. The task is field extraction
D. The task is a single obvious classification signal

**Answer: B.** On extraction it often makes things worse, because it invites the model to elaborate on what a document probably means. Arithmetic and dates still belong in code even when the task looks like reasoning.

**Q148.** The price of requested reasoning is:

A. A one-time prompt-engineering cost
B. Output tokens and latency on every call, including the cases that did not need it. A task whose reasoning triples the output has roughly tripled the expensive half of its cost
C. Input tokens only
D. Nothing at temperature 0

**Answer: B.** It is a per-task purchase. Measure the change on your own cases before adopting it.

**Q149.** Extended thinking, where the provider manages a reasoning mode, is:

A. Portable across both providers behind one adapter, and always returns the full trace
B. Not portable. Availability differs by pinned model. Some providers return a trace, some a summary, some only the answer. A design that depends on the trace works on one provider and quietly degrades on the other
C. A replacement for citations
D. Free

**Answer: B.** Anything provider-specific stays behind the adapter. Anything the comparison depends on has to be available from both.

**Q150.** The `analysis` paragraph is fluent and says the merchant record shows two prior orders. It is:

A. Evidence, because it is specific
B. Not evidence. It is generated by the same process as the decision. If the detail is wrong, the paragraph still reads the same way. A case note gets a citation a person can open, or a review reason that names a conflict in two sources
C. Required in the routing enum
D. The gold label

**Answer: B.** Models produce justifications that omit the factors that actually influenced them. Reasoning is a debugging artifact. If it has no dedicated field, it lands inside a value a downstream system reads.

**Q151.** Five samples at temperature 0.8 split three to two between two queues, on a decision where the wrong queue sends a possible fraud victim into a goods-and-services process. The useful output is:

A. The majority queue
B. The disagreement. Escalate. A majority vote is a confident answer to a question the model did not settle
C. The first sample
D. The average of the queue names

**Answer: B.** Self-consistency can improve accuracy on genuinely hard decisions, and it costs a full call per sample, so it is reserved for consequential decisions. The disagreement is usually worth more than the majority.

**Q152.** Five samples at temperature 0:

A. Are the standard grounding metric
B. Produce near-identical answers, at five times the cost, and add no information. Sampling for agreement requires a distribution wide enough for disagreement to mean something
C. Are required before schema repair
D. Remove the human boundary

**Answer: B.**

---

## 19. Measuring

Gold labels are written from the source before you measure. The score is a deterministic comparison. The families keep the same names every week.

**Q153.** Gold labels are written by:

A. Correcting the model’s output
B. Reading the source, before the run. Where two people disagree, the disagreement is resolved and the reason is recorded
C. A second model
D. Copying the example pool

**Answer: B.** Labels written from system output drift toward the system’s habits, and there is no record of what the labels should have been. A contested label produces an argument every time the number moves.

**Q154.** This week’s metrics:

A. Use a judge model
B. Compare the parsed object to the gold object in code: exact match on an enum, set comparison on a list, citation checks, and pattern search. They do not call a model
C. Score fluency
D. Use cosine similarity between the answer and the gold text

**Answer: B.** A metric that needs the raw text, the token counts, or a second model call is a different kind of metric. Evaluation frameworks and model-as-judge are later. Do not reach for them now.

**Q155.** Required-evidence recall compares:

A. Token counts with a budget
B. Fields the gold label says are recoverable with fields the model returned as present. Report a count with a denominator, such as 6/8
C. Latency across models
D. How often the answer quotes the question

**Answer: B.** Do not report that figure as a percentage.

**Q156.** A field the document does not state, returned as a plausible value, and a field the document does state, returned as absent, are:

A. The same error
B. An invention and a miss. Count them separately. Invention on a field someone will act on is the more damaging one, because a miss is visible
C. Both ignored if the rest of the object matches
D. Both scored as ambiguous

**Answer: B.** A document that contradicts itself has a gold label of ambiguous. A system that confidently picks one reading has failed even though that reading appears in the document.

**Q157.** The metric families used across the use cases are:

A. Fluency, tone, and brevity
B. Required-evidence recall, correct version selection, evidence grounding and citation correctness, routing and escalation accuracy, and personal-data leakage. Tool-call correctness and idempotency arrive when agents do
C. Cosine, BM25, and HNSW recall only
D. A single whole-object accuracy

**Answer: B.** Name them the same way every time.

**Q158.** Jurisdictions: one of two ambiguous cases called correctly. You report:

A. 50 percent ambiguity accuracy
B. The count, 1 of 2, and you treat two cases as a note to revisit when the set is larger
C. A pass for the whole extraction
D. Nothing, because ambiguity is unscored

**Answer: B.**

**Q159.** An effective-date miss on one case is:

A. Proof the prompt must be rewritten today
B. A one-case movement. It might be a defect and it might be noise. Run the same version again before changing something
C. A reason to edit the prompt in place
D. Larger than a three-case invention pattern

**Answer: B.** When a change matters, it moves several cases, or it moves the same case consistently across repeated runs.

**Q160.** Version currency in the final score is:

A. Whatever the model says is the current document
B. The result of `select_current_version`. A failure is then attributable to the extraction or to the rule
C. The nearest embedding
D. The highest version string in the prompt

**Answer: B.**

---

## Answer key

| Q | Answer | Q | Answer | Q | Answer | Q | Answer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | B | 41 | B | 81 | C | 121 | B |
| 2 | B | 42 | B | 82 | B | 122 | B |
| 3 | B | 43 | B | 83 | B | 123 | B |
| 4 | B | 44 | B | 84 | B | 124 | B |
| 5 | B | 45 | B | 85 | B | 125 | B |
| 6 | B | 46 | B | 86 | B | 126 | B |
| 7 | B | 47 | B | 87 | B | 127 | B |
| 8 | B | 48 | B | 88 | B | 128 | B |
| 9 | B | 49 | B | 89 | B | 129 | B |
| 10 | B | 50 | B | 90 | B | 130 | B |
| 11 | B | 51 | B | 91 | B | 131 | C |
| 12 | B | 52 | B | 92 | B | 132 | B |
| 13 | B | 53 | B | 93 | B | 133 | B |
| 14 | B | 54 | B | 94 | B | 134 | B |
| 15 | B | 55 | B | 95 | B | 135 | B |
| 16 | B | 56 | B | 96 | B | 136 | B |
| 17 | B | 57 | B | 97 | B | 137 | B |
| 18 | B | 58 | B | 98 | B | 138 | B |
| 19 | B | 59 | B | 99 | B | 139 | B |
| 20 | B | 60 | B | 100 | B | 140 | B |
| 21 | B | 61 | B | 101 | B | 141 | B |
| 22 | B | 62 | B | 102 | B | 142 | B |
| 23 | B | 63 | B | 103 | B | 143 | B |
| 24 | B | 64 | B | 104 | B | 144 | B |
| 25 | B | 65 | B | 105 | B | 145 | B |
| 26 | B | 66 | B | 106 | B | 146 | B |
| 27 | B | 67 | B | 107 | B | 147 | B |
| 28 | B | 68 | B | 108 | B | 148 | B |
| 29 | B | 69 | B | 109 | B | 149 | B |
| 30 | B | 70 | B | 110 | B | 150 | B |
| 31 | B | 71 | B | 111 | B | 151 | B |
| 32 | B | 72 | B | 112 | B | 152 | B |
| 33 | B | 73 | B | 113 | B | 153 | B |
| 34 | B | 74 | B | 114 | B | 154 | B |
| 35 | B | 75 | B | 115 | B | 155 | B |
| 36 | B | 76 | B | 116 | B | 156 | B |
| 37 | B | 77 | B | 117 | B | 157 | B |
| 38 | B | 78 | B | 118 | B | 158 | B |
| 39 | B | 79 | B | 119 | B | 159 | B |
| 40 | B | 80 | B | 120 | B | 160 | B |

**Q81 = C. Q131 = C.** Every other item is B.

That letter pattern is an artifact of this file: the correct rule was placed second while the questions were being checked against the articles. On the real paper the same rule will sit under A, C, or D. Learn the rule. Do not learn the letter.

---

## Rules worth memorizing

1. Prompt order: task, input, constraints, output, when the task cannot be completed.
2. A constraint is real only if the output alone shows it was followed.
3. Absent, ambiguous, and out-of-scope are three findings. One null cannot carry them.
4. After a prompt version has recorded results, the next change is a new version.
5. Reproducible means immutable prompt text plus a pinned model id, with the hash and the call record to prove it.
6. Few-shot shows a boundary and costs input tokens forever. It is not drawn from the scored cases.
7. Grounding is source, restriction, citation, and a legal absence. A true statement with a plausible citation can still be unsupported.
8. Cite a section id beside the value. Present requires value and section. Absent carries no value. Ambiguous requires a note.
9. Reasoning is not evidence. Extended thinking is not portable. Self-consistency needs a nonzero temperature, and a split is a reason to escalate.
10. If the written rule determines the answer, the step is code. Dates, lookups, thresholds, and version currency are code.
11. One comparison table per task. Counts, not percentages. Median and maximum latency. Repairs included. Transfer rows labeled.
12. An embedding is a position used for ranking inside one model. Same model, same function, and same prefixes for chunks and queries.
13. Chunk on sections first. `embed_text` has the title and heading. `text` does not. Overlap only inside a section you had to split.
14. `chunk_id` is `doc_id:version:section:ordinal`. The content hash is a separate field.
15. Similarity is higher-better. `<=>` is distance and is lower-better. Exact search is correct. HNSW can miss a true neighbor.
16. Dense bridges paraphrase. Lexical matches the token. Fuse ranks with `1 / (60 + rank)`. Do not blend scores.
17. Filter both retrievers before fusion. Current means in force on the date, not “latest” and not `superseded_by IS NULL`.
18. The model may rewrite wording. It may not add facts. Code decides the next retrieval. Record the plan.
