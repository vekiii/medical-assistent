# Four-condition evaluation — 24 October 2026

This follow-up supplements the seminar PDFs; their original results are unchanged. [Notebook 07](../notebooks/07_evaluate.ipynb) evaluates the fine-tuned model and [notebook 08](../notebooks/08_evaluate_base.ipynb) evaluates the base model. The CSV files in [base](base/) and [finetuned](finetuned/) are exact copies of the supplied Colab exports. Generated text is experimental evidence, not verified medical guidance.

## Protocol and completion

Both models use the original system prompt, 8-bit loading with automatic device placement and permitted FP32 CPU offload, greedy decoding, and a 512-token output limit. The base checkpoint is `meta-llama/Meta-Llama-3.1-8B-Instruct`; the fine-tuned checkpoint is the saved merged model. Each condition evaluates the same four questions independently with empty conversation history. The no-RAG branch supplies an empty context. Embeddings and BERTScore run on CPU.

All four conditions completed: eight CSV files contain 16 answer rows and 16 metric rows. All answers are nonempty, numeric scores are finite, and question/reference pairs match across conditions. Saved RAG context strings and source lists are identical between the two models for all four questions. Both no-RAG conditions have empty context and source fields. No exception output was found in the saved notebooks.

The base notebook still records a pip dependency conflict: Colab requires `requests==2.32.4`, while `2.34.2` was installed. The completed outputs demonstrate successful execution of these tests, not a fully consistent dependency environment. Package versions and checkpoint hashes were not captured in a run manifest, limiting exact reproduction. Notebook code records the intended settings; the CSV files are not a complete execution provenance record.

## Mean scores

Arithmetic means over four questions, rounded to four decimal places. Full precision is retained in the source CSVs and [summary.csv](summary.csv).

| Test | Model | RAG | ROUGE-L | BLEU | BERTScore F1 | Cosine similarity |
| --- | --- | --- | ---: | ---: | ---: | ---: |
| 3 | Base | No | 0.0894 | 0.0047 | 0.6652 | 0.7172 |
| 2 | Fine-tuned | No | 0.1359 | 0.0096 | 0.6762 | 0.6974 |
| 4 | Base | Yes | 0.3758 | 0.0130 | 0.7816 | 0.8995 |
| 1 | Fine-tuned | Yes | 0.2726 | 0.0113 | 0.7562 | 0.8529 |

RAG increases all four mean similarity metrics for both models on this sample. With identical RAG context, the fine-tuned model scores lower than the base model on all four mean metrics. This run provides **no evidence of an average similarity-score improvement from this additional fine-tuning when RAG is present**. It does not establish that fine-tuning is generally ineffective or that the base model is always more factually accurate.

Without RAG, fine-tuning increases three mean metrics but decreases embedding cosine similarity. Under the original prompt, however, empty context should lead to refusal. Similarity to an answer-bearing reference is therefore not an adequate measure of success in that condition.

## Observed answer behavior

- **Nimulid with RAG:** both models produce exactly the same answer. Both benefit from the supplied dosage passage, while grammatical errors remain.
- **Pressing with RAG:** the fine-tuned model says it lacks information; the base model uses the under-two-years restriction present in the retrieved text. The context also includes passages about other medicines, making this a noisy retrieval case. This one question contributes substantially to the average gap.
- **Dovicin with RAG:** the fine-tuned answer omits the reference's warning about the period of tooth development. The base answer changes the reference's “should not be used” wording to “use with caution” and contains severe lexical distortions. Similarity scores alone do not adequately capture these problems.
- **Without RAG:** neither model consistently follows the instruction to refuse when evidence is absent. The fine-tuned model refuses the Dovicin question but answers the other three; its Nimulid dosage differs from the supplied reference. Both models make claims not supported by supplied context, which is empty in these tests.
- **Language and instructions:** both models show grammatical errors and inconsistent inclusion of the required disclaimer. This sample contains only Latin-script questions and cannot establish behavior on Cyrillic queries.
- **Output length:** the base/no-RAG Zoloft answer ends mid-sentence. This is consistent with reaching a generation limit, but token counts and finish reasons were not exported, so the cause cannot be confirmed from the CSV alone.

These observations compare saved model outputs with the provided references and retrieved text. They are not an independent medical review, and assertions made by the models should not be treated as established facts.

## Interpretation limits and next steps

The four cases were already used during development; they are a pilot, not an independent held-out benchmark. References contain original typos and are not exhaustive gold answers. ROUGE uses the original tokenizer, which is not adapted to Serbian Cyrillic. BLEU, BERTScore and embedding similarity measure text resemblance rather than factual correctness. No significance tests, repeated runs or expert ratings were performed.

The prompt, reference questions, intended generation settings and precision are controlled, and equality of the saved RAG context was verified. This supports the narrow within-run comparisons above. It does not isolate inference quantization, which is fixed at 8 bits, or quantization during training. Optional multi-turn notebook examples are not included in these aggregate metrics; their history-aware query rewriting can yield different context across models.

Elapsed times are preserved in the exports, but output lengths, CPU offload placement, warmup and session conditions were not controlled as a latency benchmark. Shorter answers or refusals can be faster. Do not interpret the timing difference as a model-speed advantage.

Next steps are a larger independent Serbian pharmaceutical QA benchmark, expert correctness and instruction-following assessment, both scripts, analysis of retrieval noise, training-data cleanup, and better-targeted adaptation data. Evaluate prompt/training-format changes as new experiments. Preserve these results as evidence of the current implementation, including its failures.
