# Notebook Guide

[English project overview](../README.md) · [Uputstvo na srpskom](../docs/sr/README.md) · [Project summary](../summary.md)

These notebooks document the original seminar experiment. They have been reorganized for reading, without rerunning the experiment or changing its executable behavior. Explanations and comments are in English; Serbian prompts, examples, reference answers, docstrings, and printed results remain unchanged.

## Choose a starting point

- **Review the project:** read [the summary](../summary.md), then the recorded conversations in notebook 05 and the evaluation in notebook 06.
- **Run inference with existing artifacts:** start with notebook 05 after configuring its runtime, model path, and index path.
- **Reproduce the complete workflow:** follow the order below. Index creation (04) is independent of training (01–03).

| Order | Notebook | Inputs | Outputs |
| --- | --- | --- | --- |
| 01 | [Prepare training data](01_prepare_training_data.ipynb) | Local SQuAD files and Hugging Face datasets | Combined training JSONL |
| 02 | [Explore training data](02_explore_training_data.ipynb) | Combined training JSONL | Missing-value, length, script, and frequency statistics |
| 03 | [Fine-tune and merge](03_finetune_and_merge.ipynb) | Training data, base model, optional checkpoints | LoRA checkpoints and merged model |
| 04 | [Build the retrieval index](04_build_retrieval_index.ipynb) | Local leaflet PDFs | Chunk collection and FAISS index |
| 05 | [Run the assistant](05_run_assistant.ipynb) | Compatible merged model and FAISS index | Retrieved passages and recorded conversations |
| 06 | [Evaluate the system](06_evaluate.ipynb) | Model, index, embedded reference cases | Per-question and aggregate metrics |

## Reading and executing cells

Each notebook begins with its purpose, inputs, outputs, runtime requirements, and a configuration overview. That overview collects the settings to inspect in one place; assignments remain in their original code cells to preserve execution order.

Run code cells from top to bottom in the intended environment. Saved execution counts and experimental outputs belong to the original run, not a fresh verification run. Empty unused cells and routine installation output have been removed; installation errors, training logs, retrieval examples, generated answers, and metric tables are retained.

The notebooks use Google Colab/Drive paths, except that data preparation also contains a local Windows SQuAD path. Consult the [project setup instructions](../README.md#prerequisites-and-paths) before execution. Dependency installation commands are preserved rather than replaced with a newly tested environment.

### Conversation boundaries

Notebook 05 explicitly labels the original history behavior:

- **Session A** resets history before Dovicin in Latin script, then continues through Cyrillic Dovicin and Nimulid examples.
- **Session B** resets history before Pressing, then continues through modified Pressing queries, Zoloft, and out-of-domain/language examples.

These labels describe the existing code. No additional resets were inserted. A self-contained question in a later block is therefore not necessarily an independent conversation. Notebook 06, in contrast, passes empty history for each generated-answer evaluation question.

### Evaluation boundaries

Notebook 06 separates setup, reference labels, metric definitions, execution, aggregate results, and interpretation. The four-question sample, source-file relevance labels, differing Pressing queries, original reference strings, and metric implementations remain unchanged. The proposed controlled base-plus-RAG versus fine-tuned-plus-RAG comparison is documented as future work, not as a completed experiment.

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
