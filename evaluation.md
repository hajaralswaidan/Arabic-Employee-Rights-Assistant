# Evaluation and Error Analysis

[Back to Project Overview](README.md)

## 1. Evaluation Setup

The retrieval methods were compared using 15 Arabic development questions with manually identified reference PDF pages.

The questions cover contracts, wages, working hours, overtime, leave, probation, workplace responsibilities, resignation, and dispute resolution.

Both methods use the same document chunks, questions, and top-three cutoff.

One wage-payment question was clarified during development to ask specifically about cash versus bank transfer. This is a development comparison, not an untouched held-out test.

---

## 2. Retrieval Methods

| Method | Ranking Approach |
|---|---|
| BM25 Baseline | Lexical matching with term weighting |
| Dense Retrieval | Similarity between question and document embeddings |

For BM25, tokenization removes Arabic diacritics, normalizes selected alef forms, lowercases text, and extracts word tokens. The displayed source text remains unchanged.

---

## 3. Evaluation Metric

**Page Hit@3** checks whether at least one of the top three retrieved chunks comes from a reference page.

Each question receives:
- `True` if a reference page is retrieved.
- `False` otherwise.

The score is the number of successful questions divided by the total number of questions.

This metric does not check whether the retrieved chunk contains the complete answer or whether the generated response is correct.

---

## 4. Results and Interpretation

Both methods retrieved a reference page for all 15 development questions. Neither method outperformed the other on Page Hit@3.

The result should be interpreted carefully:

- The evaluation set is small.
- Several chunks can come from the same page.
- A reference-page hit may not contain the exact evidence needed.
- A generator can misunderstand correctly retrieved information.

The comparison evaluates retrieval, not two complete answer-generation pipelines. No numeric end-to-end answer-accuracy, citation-support, latency, or cost benchmark was established.

---

## 5. Manual Answer Review

Responses were checked against the available document excerpts for factual correctness, important conditions, and supporting page references.

| Test | Observed Result |
|---|---|
| Contract information question | Listed the requested information and cited PDF page 5 |
| General working-hours question | Initially confused daily and weekly values |
| Working-hours question after formatting | Correct numerical values, but omitted the Muslim-specific Ramadan condition |
| Explicit Ramadan eligibility question | Correct condition, values, and citation to PDF page 6 |
| Probation question in Saudi dialect | Answered the contract requirement and cited PDF page 10 |
| Personal leave-balance question | Abstained without inventing a balance |
| Passport-renewal procedure question | Abstained because the retrieved passages did not answer it |

These are individual development checks, not a comprehensive accuracy evaluation.

---

## 6. Main Error: Working-Hours Interpretation

The assistant initially reported **48 hours per day**, although the retrieved text contained **8 daily hours** and **48 weekly hours**.

The citation pointed to the correct page, so checking the page number alone would not have detected the incorrect claim.

The extracted text placed numbers before their labels. Fixed-window chunking could also separate headings and conditions, making the context harder to interpret.

### Changes Tested

| Change | Observed Outcome |
|---|---|
| Added instructions to verify values, units, and labels | The incorrect response continued |
| Applied `format_work_hours` inside `ask` | A later response correctly reported daily and weekly values |
| Asked directly about Ramadan eligibility | The response included the Muslim-specific condition |

Formatting improved the tested numerical response. However, the broad answer still omitted a condition, so the issue is not considered fully resolved across all question types.

---

## 7. Failure Types

| Failure Type | Example or Meaning |
|---|---|
| Extraction / Context | PDF layout disrupts the relationship between values and labels |
| Retrieval | The required evidence is not included in the selected chunks |
| Generation | Relevant evidence is available, but the answer misinterprets it or omits a condition |
| Service | Rate limits or empty model responses prevent an answer |

A service-status message is not counted as a correct answer. A correct citation is also not sufficient evidence that every statement is supported.

---

## 8. Limitations

- Only one document and a small development question set were used.
- Reference pages may not capture every alternative location containing an answer.
- Page-level evaluation can overestimate useful retrieval.
- Fixed character windows can split words, tables, and conditions.
- The formatting helper handles one observed pattern rather than arbitrary PDF tables.
- Factual claims and citations are not automatically validated.
- Model outputs may vary or omit important conditions.
- Conversation history is not used when answering follow-up questions.
- Personal HR records are unavailable.
- Model availability and API quota can interrupt testing.
- The document is not independently checked for later regulatory updates.

---

## 9. Future Improvements

| Area | Possible Improvement |
|---|---|
| Document Preparation | Preserve headings, tables, and complete conditions |
| Retrieval Labels | Annotate exact supporting chunks rather than pages alone |
| Evaluation Data | Add a larger held-out set with dialects, paraphrases, and unanswerable questions |
| Answer Review | Measure factual correctness, condition completeness, and citation support |
| Retrieval Methods | Compare hybrid search and reranking against the current baselines |
| API Usage | Cache question embeddings and avoid repeated requests |
| Reproducibility | Add automatic checkpoint restoration |
| Operations | Measure latency and API usage |
