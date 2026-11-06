# FinEvidence AI

**An evidence-grounded, evaluation-first system for question answering over financial filings.**

FinEvidence AI is a research-style project that studies how AI systems can answer questions about annual reports and public filings using **verifiable evidence**, and how to measure whether they succeed.

The goal is to build a system that does more than generate fluent answers: it retrieves relevant evidence, cites it at page level, separates reported figures from calculated values, and recognizes when the available evidence is insufficient to answer.

---

## Motivation

Financial filings are long, structured, and dense with numbers. A language model can produce a convincing answer even when the supporting evidence is missing, misread, or taken from the wrong period. In financial analysis, a confident but incorrect figure can be more harmful than no answer.

Before such a system can be trusted, it is important to understand *which stage fails*: finding the right page, extracting the right figure, performing the arithmetic, interpreting the result, or citing the source.

FinEvidence AI is designed to make these failure points separately visible and measurable.

## Research Question

> How do lexical (BM25), dense, and hybrid retrieval methods affect evidence retrieval for financial questions, and can an evidence-sufficiency check reduce unsupported answers when the required evidence is genuinely unavailable?

### Supporting Questions

- How much of end-to-end error comes from retrieval versus extraction, arithmetic, reasoning, or citation?
- Does a deterministic calculation step reduce numerical errors compared with letting the language model perform arithmetic?
- How reliably can citation support be verified automatically, compared with manual checking?

The project does not claim to have invented retrieval-augmented generation or financial question answering. Its intended contribution is a careful, reproducible, stage-by-stage evaluation on a public benchmark.

## Project Objectives

- Retrieve relevant evidence from financial reports at page level.
- Compare lexical, dense, and hybrid retrieval methods.
- Generate answers grounded only in retrieved evidence.
- Provide traceable citations (document and page) for every figure used.
- Separate **reported figures**, **calculated values**, and **interpretations** in each answer.
- Verify whether cited evidence actually supports the answer.
- Abstain when the retrieved evidence is insufficient.
- Analyse errors by stage and by question type.

## Dataset

The initial benchmark is the open-source sample of [FinanceBench](https://github.com/patronus-ai/financebench), a benchmark for open-book financial question answering, together with its accompanying document metadata.

### Dataset Exploration

The following observations were obtained during the initial exploration of the provided dataset files.

| Item | Observed Value |
|---|---:|
| Questions | 150 |
| Columns in the question dataset | 11 |
| Metrics-generated questions | 50 |
| Domain-relevant questions | 50 |
| Novel-generated questions | 50 |
| Distinct documents referenced | 84 |
| Rows in the document metadata file | 361 |
| Total annotated evidence items | 189 |

### Evidence Distribution

| Evidence Items per Question | Number of Questions |
|---:|---:|
| 1 | 115 |
| 2 | 31 |
| 3 | 4 |
| **Total questions** | **150** |

### Observations

- Gold answers include both short numerical strings and longer explanatory responses. This makes simple string matching insufficient for evaluation.
- Financial values require unit-aware comparison, including differences such as millions versus billions, percentages versus absolute values, and fiscal-year differences.
- Some questions require multiple evidence passages, making multi-evidence retrieval an important part of the evaluation.
- Reasoning-type labels contain missing and combined values, so any analysis based on these labels will require documented cleaning.
- The equal distribution across question types reflects how the sample was constructed, not necessarily the natural distribution of financial analyst questions.
- Evidence page annotations use zero-indexed page numbers, which will be preserved during document ingestion and retrieval evaluation.

### Data Handling

This repository does not redistribute FinanceBench data or source PDFs. The benchmark files and underlying documents remain subject to their original access and licensing terms.

### Possible Supplementary Data

A small set of additional questions may be written and manually verified to test scenarios not sufficiently covered by the benchmark. These questions will be reported separately and will not be merged with the official FinanceBench sample.

Other datasets, such as FinQA, will be considered only if they add a meaningful evaluation dimension.

## Approach

### 1. Document Ingestion

Extract text from financial filings page by page, preserving document names and zero-indexed page numbers so that retrieved content can be compared directly with the benchmark's evidence annotations.

Tables and number-heavy pages are expected to be important sources of extraction difficulty. Their handling and limitations will be documented explicitly.

### 2. Evidence Retrieval

Build and compare three retrieval strategies over the same page-level document corpus:

- **Lexical Retrieval:** BM25, matching financial terms, names, and exact figures.
- **Dense Retrieval:** Embedding-based semantic search to capture meaning beyond exact keyword matches.
- **Hybrid Retrieval:** A fusion of lexical and dense rankings.

Retrieval will be evaluated independently of answer generation by comparing retrieved pages with the benchmark's annotated evidence pages.

Planned metrics include:

- **Recall@k:** Whether relevant evidence appears within the top-k retrieved pages.
- **MRR (Mean Reciprocal Rank):** How highly the first relevant page is ranked.
- **Multi-evidence retrieval:** Whether all required evidence pages are retrieved, rather than only one.

The definition of a retrieval hit (any gold evidence page versus all required gold evidence pages) will be fixed before evaluation and reported explicitly.

### 3. Evidence-Grounded Answering

Generate answers using retrieved evidence only, in a structured format that distinguishes:

- **Reported Figures:** Values quoted directly from the filing, accompanied by document and page citations.
- **Calculated Values:** Derived numbers with the formula, inputs, units, and reporting period shown.
- **Interpretation:** Explanations or judgments based on the evidence, clearly distinguished from reported facts.

The system should not introduce unsupported financial figures or claims that cannot be traced to the retrieved source material.

### 4. Numerical Reasoning

Use deterministic calculation for arithmetic operations such as ratios, margins, and percentage changes.

This allows numerical errors to be analysed more precisely: an incorrect result may come from retrieving the wrong figure, extracting a value incorrectly, selecting the wrong reporting period, or applying the wrong formula.

Unit and period handling, including conversions such as millions to billions and comparisons across fiscal years, will be treated as evaluation concerns in their own right.

### 5. Citation Verification

Check whether cited pages actually support each stated figure or claim.

Automated citation checks will be validated against manual review on a subset of answers. This makes citation verification itself an object of evaluation rather than assuming that a generated citation is correct.

### 6. Evidence Sufficiency and Abstention

Test whether the system can recognize when the available evidence is insufficient and decline to answer.

One planned experiment removes the annotated gold evidence pages for selected questions and measures whether the system abstains instead of generating an unsupported answer.

However, removing an annotated evidence page does not automatically make a question unanswerable. The same information may appear elsewhere in the filing. The experiment will therefore validate whether the required evidence is genuinely unavailable.

The evaluation will also track **over-refusal**: cases where the system abstains even though sufficient evidence is available.

### 7. Error Analysis

Analyse failures by stage:

- Retrieval
- Extraction
- Arithmetic
- Reasoning
- Citation

Failures will also be examined by question type, with representative examples and a discussion of what each result does and does not demonstrate.

## Planned Experiments

| # | Research Question | Baseline | Changed Component |
|---:|---|---|---|
| 1 | Does retrieval strategy change evidence recall? | BM25 | Dense and hybrid retrieval |
| 2 | Does a calculation tool reduce numerical errors? | LLM-only arithmetic | Deterministic calculator |
| 3 | How reliable is automated citation checking? | Manual review | Automated verifier |
| 4 | Can evidence-sufficiency checks reduce unsupported answers? | Always-answer policy | Abstention policy |

No outcome is assumed for any of these. Positive, negative, and inconclusive results will all be reported.

## Evaluation Principles

- Retrieval is evaluated **separately** from answer quality.
- Methods are compared on the same questions using paired comparisons and confidence intervals.
- A portion of the questions is held out from tuning until final evaluation. The split is fixed before experiments begin.
- Numerical answers are compared with unit awareness; long-form answers require a documented evaluation method and manual review.
- Any language-model-based judging is checked against manual review on a subset.
- Results on the official benchmark and any self-written questions are reported separately.
- Cost and latency are recorded alongside accuracy.

## Limitations

- The open-source sample contains only 150 questions, so only large differences between methods can be trusted. Breakdowns by question type or reasoning type are exploratory.
- The benchmark covers a limited set of companies and document types; results will not automatically generalise to other filings or tasks.
- Gold answers and reasoning labels are imperfect, and some questions admit more than one valid answer.
- PDF text extraction may lose table structure or introduce errors in number-heavy content.
- Automated citation and answer judging have their own error rates.
- Abstention policies involve a trade-off between unsupported answers and refusing answerable questions.
- This is an academic research project, not a production system, and makes no claim of real-world reliability.

## Related Work

- **FinanceBench** (Islam et al., 2023): The benchmark used in this project. [Paper](https://arxiv.org/abs/2311.11944)
- **FinQA** (Chen et al., EMNLP 2021): A dataset of numerical reasoning over financial reports with annotated reasoning programs. [ACL Anthology](https://aclanthology.org/2021.emnlp-main.300)
- **ALCE** (Gao et al., EMNLP 2023): A benchmark for automatically evaluating language-model answers with citations. [Paper](https://arxiv.org/abs/2305.14627)
- **Financial Report Chunking for Effective RAG** (Jimeno Yepes et al., 2024): Research on chunking strategies for financial reports. [Paper](https://arxiv.org/abs/2402.05131)

## Disclaimer

FinEvidence AI is an academic research project. It is not financial advice, is not affiliated with any employer or with the authors of the benchmark, and must not be used for investment decisions.
