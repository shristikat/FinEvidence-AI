# FinEvidence-AI
FinEvidence AI

An evidence-grounded, evaluation-first system for question answering over financial filings.

FinEvidence AI is a research-style project that studies how AI systems can answer questions about annual reports and public filings using verifiable evidence, and how to measure whether they succeed. The goal is a system that does more than generate fluent answers: it retrieves evidence, cites it at page level, separates reported figures from calculated values, and recognizes when the available evidence is insufficient to answer.
Motivation

Financial filings are long, structured, and dense with numbers. A language model can produce a convincing answer even when the supporting evidence is missing, misread, or taken from the wrong period, and a confident wrong figure is more dangerous than no answer.

Before such a system can be trusted, it has to be understood: which stage fails? Finding the right page, extracting the right figure, doing the arithmetic, interpreting the result, or citing the source? FinEvidence AI is designed to make each of these failure modes separately visible and measurable.

Research question

How do lexical (BM25), dense, and hybrid retrieval methods affect evidence retrieval for financial questions, and can an evidence-sufficiency check reduce unsupported answers when the required evidence is genuinely unavailable?

Supporting questions:

How much of end-to-end error comes from retrieval versus extraction, arithmetic, reasoning, or citation?
Does a deterministic calculation step reduce numerical errors compared with letting the language model do arithmetic?
How reliably can citation support be verified automatically, compared with manual checking?

The project does not claim to have invented retrieval-augmented generation or financial question answering. Its intended contribution is a careful, reproducible, stage-by-stage evaluation on a public benchmark.

Project objectives
Retrieve relevant evidence from financial reports at page level.
Compare lexical, dense, and hybrid retrieval.
Generate answers grounded only in retrieved evidence.
Provide traceable citations (document and page) for every figure used.
Separate reported figures, calculated values, and interpretations in each answer.
Verify whether cited evidence actually supports the answer.
Abstain when the retrieved evidence is insufficient.
Analyse errors by stage and by question type.
Dataset

The initial benchmark is the open-source sample of FinanceBench, a benchmark for open-book financial question answering, with its accompanying document metadata.

Observed during exploration of the sample

Item	Value
Questions	150 (11 columns)
Question types	50 metrics-generated, 50 domain-relevant, 50 novel-generated
Distinct documents referenced by the questions	84
Rows in the document metadata file	361
Evidence per question	A list of evidence items; the 3 questions inspected so far had 1, 1, and 3 items

Observations that shape the evaluation design:

Gold answers mix short numeric strings (with differing units) and long explanatory answers, so simple string matching is unreliable and unit-aware numeric checking is required.
The reasoning-type labels are inconsistent in places (combined labels and missing labels), so they will need documented cleaning before any analysis by reasoning type.
The 50/50/50 split across question types reflects how the sample was constructed, not a natural distribution of analyst questions.

Data handling. This repository does not redistribute FinanceBench data or source PDFs. Obtain them from the original repository and place them in data/ (git-ignored). Anyone reusing them should confirm the licensing and access terms of the benchmark and of the underlying documents.

Possible supplementary data (planned). A small set of additional questions written and manually verified by the author, reported separately from the official sample and never merged with it. Other datasets (for example FinQA) will be considered only if they add something the design needs.

Approach
1. Document ingestion

Extract text from filings page by page, preserving document name and zero-indexed page number so that retrieved text can be compared directly with the benchmark's evidence annotations. Tables and number-heavy pages are expected to be a main source of extraction difficulty and will be handled and documented explicitly.

2. Retrieval

Build and compare three retrieval strategies over the same page-level corpus:

Lexical: BM25.
Dense: embedding-based semantic retrieval.
Hybrid: a fusion of lexical and dense rankings.

Retrieval is evaluated without any language model, by comparing retrieved pages with the annotated evidence pages (Recall@k, MRR, and related measures). The definition of a retrieval "hit" (any gold page versus all gold pages) is fixed before measuring and reported explicitly.

3. Evidence-grounded answering

Generate answers from retrieved pages only, in a structured format that separates:

Reported figures: values quoted from the filing, each with its document and page.
Calculated values: derived numbers, with the formula, inputs, units, and period shown.
Interpretation: any judgement or explanation, clearly labelled as such.
4. Numerical reasoning

Use deterministic calculation for arithmetic (ratios, margins, changes), so that numerical errors can be attributed to extraction or reasoning rather than to the language model's arithmetic. Unit and period handling (millions versus billions, fiscal years) is treated as an evaluation concern in its own right.

5. Citation verification

Check whether the cited pages actually support each stated figure or claim. Automated checks will be validated against manual review on a subset, so their reliability is itself measured.

6. Evidence sufficiency and abstention

Test whether the system can recognize when the evidence is insufficient and decline to answer. One planned experiment removes the gold evidence pages for a set of questions and measures whether the system abstains rather than producing an unsupported answer, while also tracking over-refusal on questions that are answerable.

7. Error analysis

Label failures by stage (retrieval, extraction, arithmetic, reasoning, citation) and by question type, with examples and a written discussion of what each result does and does not show.

Planned experiments
#	Question	Baseline	Changed component
1	Does retrieval strategy change evidence recall?	BM25	Dense, hybrid
2	Does a calculation tool reduce numerical errors?	LLM-only arithmetic	Deterministic calculator
3	How reliable is automated citation checking?	Manual review	Automated verifier
4	Can evidence-sufficiency checks reduce unsupported answers?	Always answer	Abstention policy

No outcome is assumed for any of these. Positive, negative, and inconclusive results will all be reported.

Evaluation principles
Retrieval is evaluated separately from answer quality.
Methods are compared on the same questions (paired comparison), with confidence intervals.
A portion of the questions is held out and not used for tuning until the final evaluation; the split is fixed before experiments begin.
Numeric answers are compared with unit awareness; long-form answers require a documented method and manual review.
Any language-model-based judging is checked against manual review on a subset.
Results on the official sample and on any self-written questions are reported separately.
Cost and latency are recorded alongside accuracy.
Limitations
The open-source sample contains only 150 questions, so only large differences between methods can be trusted, and breakdowns by question type or reasoning type are exploratory.
A single benchmark and a limited set of companies and document types; results will not generalise automatically to other filings or tasks.
Gold answers and reasoning labels are imperfect, and some questions admit more than one valid answer.
Automated citation and answer judging has its own error rates.
Abstention policies trade unsupported answers against refusing answerable questions; neither is free.
This is not a production system and makes no claim of real-world reliability.
Related work
FinanceBench (Islam et al., 2023): the benchmark used here. arXiv:2311.11944
FinQA (Chen et al., EMNLP 2021): a dataset of numerical reasoning over financial reports with annotated reasoning programs. ACL Anthology
ALCE (Gao et al., EMNLP 2023): a benchmark for automatically evaluating language-model answers with citations. arXiv:2305.14627
Financial Report Chunking for Effective RAG (Jimeno Yepes et al., 2024): chunking strategies for financial reports. arXiv:2402.05131

Disclaimer

FinEvidence AI is an academic research project. It is not financial advice, is not affiliated with any employer or with the authors of the benchmark, and must not be used for investment decisions.
