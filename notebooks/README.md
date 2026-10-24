# Notebook Guide

[English project overview](../README.md) · [Uputstvo na srpskom](../docs/sr/README.md) · [Project summary](../summary.md)

Notebooks 01–06 document the original seminar experiment. They have been reorganized for reading, without rerunning the experiment or changing its executable behavior. Explanations and comments are in English; Serbian prompts, examples, reference answers, docstrings, and printed results remain unchanged. Notebooks 07 and 08 implement the new four-test comparison using the original application workflow; the completed results are archived in [results/](../results/README.md).

## Choose a starting point

- **Review the project:** read [the summary](../summary.md), then the recorded conversations in notebook 05 and the evaluation in notebook 06.
- **Run inference with existing artifacts:** start with notebook 05 after configuring its runtime, model path, and index path.
- **Compare fine-tuning and context availability on a T4:** run notebooks 07 and 08 in separate sessions using the existing Drive artifacts. They use the original prompt and 8-bit loading settings. Running 01–06 again is unnecessary.
- **Reproduce the complete workflow:** follow the order below. Index creation (04) is independent of training (01–03).

| Order | Notebook | Inputs | Outputs |
| --- | --- | --- | --- |
| 01 | [Prepare training data](01_prepare_training_data.ipynb) | Local SQuAD files and Hugging Face datasets | Combined training JSONL |
| 02 | [Explore training data](02_explore_training_data.ipynb) | Combined training JSONL | Missing-value, length, script, and frequency statistics |
| 03 | [Fine-tune and merge](03_finetune_and_merge.ipynb) | Training data, base model, optional checkpoints | LoRA checkpoints and merged model |
| 04 | [Build the retrieval index](04_build_retrieval_index.ipynb) | Local leaflet PDFs | Chunk collection and FAISS index |
| 05 | [Run the assistant](05_run_assistant.ipynb) | Compatible merged model and FAISS index | Retrieved passages and recorded conversations |
| 06 | [Evaluate the system](06_evaluate.ipynb) | Model, index, embedded reference cases | Per-question and aggregate metrics |
| 07 | [Fine-tuned model evaluation](07_evaluate.ipynb) | Merged model, index, shared questions | Tests 1-2: with/without RAG; answers and metrics CSV |
| 08 | [Base model evaluation](08_evaluate_base.ipynb) | Gated base model, index, shared questions | Tests 3-4: without/with RAG; answers and metrics CSV |

## Reading and executing cells

Each notebook begins with its purpose, inputs, outputs, runtime requirements, and a configuration overview. That overview collects the settings to inspect in one place; assignments remain in their original code cells to preserve execution order.

Run code cells from top to bottom in the intended environment. Saved execution counts and outputs in notebooks 01-06 belong to the original experiment; notebooks 07-08 contain the follow-up run. Empty unused cells and routine installation output have been removed; installation errors, training logs, retrieval examples, generated answers, and metric tables are retained.

The notebooks use Google Colab/Drive paths, except that data preparation also contains a local Windows SQuAD path. Consult the [project setup instructions](../README.md#prerequisites-and-paths) before execution. Dependency installation commands are preserved rather than replaced with a newly tested environment.

### Conversation boundaries

Notebook 05 explicitly labels the original history behavior:

- **Session A** resets history before Dovicin in Latin script, then continues through Cyrillic Dovicin and Nimulid examples.
- **Session B** resets history before Pressing, then continues through modified Pressing queries, Zoloft, and out-of-domain/language examples.

These labels describe the existing code. No additional resets were inserted. A self-contained question in a later block is therefore not necessarily an independent conversation. Notebook 06, in contrast, passes empty history for each generated-answer evaluation question.

### Evaluation boundaries

Notebook 06 separates setup, reference labels, metric definitions, execution, aggregate results, and interpretation. The four-question sample, source-file relevance labels, differing Pressing queries, original reference strings, and metric implementations remain unchanged. The four-test comparison was completed in notebooks 07 and 08 on 24 October 2026; see the [findings and CSV exports](../results/README.md).

### Running the new evaluation

Use the same Colab workflow as notebook 05: install dependencies, mount Drive, configure paths, and load one 8-bit model. Notebook 07 loads the merged model and contains test 1 (with RAG) and test 2 (without RAG). Notebook 08 loads the original base model with the same quantization and contains test 3 (without RAG) and test 4 (with RAG). Restart the runtime before switching notebooks. The base model requires your own Hugging Face account with access to the gated repository.

Every test runs the same four reference questions from notebook 06 independently with empty history. Answer text, reference, elapsed time, retrieved sources and context are saved under `NLP_proyect/evaluation_simple/finetuned` or `evaluation_simple/base`. ROUGE-L, BLEU, BERTScore F1 and embedding cosine are then computed using the original metric definitions. BERTScore and the embedding model run on CPU to preserve GPU memory. The reviewed exports are published under [results/](../results/README.md); raw local root folders `base/` and `finetuned/` are ignored by Git. Drive output files are overwritten when the same section is rerun; the notebook does not automatically resume or calculate paired deltas. Compare the CSV tables on the same questions. Optional conversation cells copy the original notebook 05 calls and history resets in all four sections.

The primary fine-tuning comparison is test 1 versus test 4 on the isolated questions. Conversational RAG uses the original history-aware retriever, so its reformulated queries and retrieved context can differ between models. The original prompt is retained verbatim, including its special tokens, refusal and disclaimer. With empty context, refusal is correct; low reference overlap is not evidence of poor instruction following. History is recorded but is not inserted into the no-RAG answer prompt. Quantization is fixed at 8 bits and its effect is not measured. The four cases are a pilot, and similarity metrics do not establish factual correctness.

The installation follows the simple approach used in notebooks 05/06, without the previous dependency-repair framework. Both notebooks completed the supplied runs. The base run still logged a Colab requests-version conflict; completion does not establish that all installed dependencies are consistent. The earlier notebook draft is backed up outside the repository in `pregled_rada/07_evaluate_before_simplification.ipynb`.

## Original filenames in the papers

The PDFs describe the recorded experiment and retain its original filenames. Use this mapping when following those references:

| Original name | Current name |
| --- | --- |
| `dataset_LoRA.ipynb` | [01_prepare_training_data.ipynb](01_prepare_training_data.ipynb) |
| `final_train_dataset_EDA.ipynb` | [02_explore_training_data.ipynb](02_explore_training_data.ipynb) |
| `LLM_finetuning.ipynb` | [03_finetune_and_merge.ipynb](03_finetune_and_merge.ipynb) |
| `vector_database_creation.ipynb` | [04_build_retrieval_index.ipynb](04_build_retrieval_index.ipynb) |
| `app.ipynb` | [05_run_assistant.ipynb](05_run_assistant.ipynb) |
| `evaluation.ipynb` | [06_evaluate.ipynb](06_evaluate.ipynb) |
