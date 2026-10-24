# Serbian Medication Leaflet Assistant

**English** | [Srpski](docs/sr/README.md)

A seminar research project combining retrieval-augmented generation (RAG) over Serbian medicine package leaflets with QLoRA fine-tuning of Llama 3.1 8B Instruct. This repository contains the shared implementation, training data, a saved retrieval index, and reports in Serbian and English.

The application works with Serbian queries and documents. The English edition provides documentation for international readers and reviewers; it is not a separately trained English-language application. Notebook explanations and comments are in English; Serbian prompts, examples, docstrings, and recorded outputs retain their original language.

## Reports and resources

| Resource | Description |
| --- | --- |
| [English paper — PDF](docs/en/paper.pdf) | Revised version for review, updated 23 October 2026 |
| [Serbian paper — PDF](docs/sr/seminarski_rad.pdf) | Original submitted report, preserved unchanged |
| [Serbian project guide](docs/sr/README.md) | Overview and reproduction instructions in Serbian |
| [Project summary](summary.md) | Implementation contribution, technical choices, evaluation, and next steps |
| [Follow-up evaluation and CSV results](results/README.md) | Completed four-condition comparison, 24 October 2026; findings and limitations |
| [Hugging Face model](https://huggingface.co/vekiii/medical-assistent-serbian-model) | Model artifact location |
| [Hugging Face Space](https://huggingface.co/spaces/vekiii/medical-assistant-serbian) | Experimental deployment; availability depends on its runtime and hardware quota |

The English paper corrects references, technical descriptions, and interpretation of results. It is therefore not an exact translation of the archived Serbian paper. Recorded numerical results and verbatim model outputs are retained. The English paper has 17 footnotes and the Serbian paper 18. Reports are published as PDFs; editable Word files are excluded from Git.

## Scope and limitations

This is a research prototype. Its examples document factual errors, incomplete answers, confusion between medicines, inconsistent script use, and failures to follow system instructions. The results do not establish clinical reliability or readiness for unsupervised medical use.

The practical contribution is an implemented pipeline under limited GPU resources and an analysis of its failure modes. Numerical evaluation covers only **four questions**:

- Retrieval relevance is assigned by source PDF filename. A chunk from the expected leaflet may count as relevant without containing the answer.
- The Pressing retrieval query includes “sirup”, whereas the generation query does not. These rows evaluate different inputs.
- Semantic similarity does not establish factual accuracy. The default ROUGE tokenizer used by the notebook is not adapted to Serbian scripts.
- The original paper comparison does not isolate retrieval, fine-tuning, and quantization. The [follow-up](results/README.md) compares base/fine-tuned models with/without context at fixed 8-bit precision; quantization effects remain unmeasured.
- **591 input fields contain the literal string `nan`**, which the training formatter includes in prompts. The original data and scores are retained to document the experiment; clean these placeholders before a new training run.

## Repository structure

```text
.
├── README.md                         # English overview and instructions
├── docs/
│   ├── en/paper.pdf                  # Revised English paper
│   └── sr/
│       ├── README.md                 # Serbian overview and instructions
│       └── seminarski_rad.pdf        # Original Serbian paper
├── notebooks/                       # Shared implementation
│   ├── 01_prepare_training_data.ipynb
│   ├── 02_explore_training_data.ipynb
│   ├── 03_finetune_and_merge.ipynb
│   ├── 04_build_retrieval_index.ipynb
│   ├── 05_run_assistant.ipynb
│   ├── 06_evaluate.ipynb
│   ├── 07_evaluate.ipynb
│   └── 08_evaluate_base.ipynb
├── results/                         # Published follow-up findings and CSV exports
│   ├── README.md
│   ├── summary.csv
│   ├── base/
│   └── finetuned/
├── dataset_LoRA/
│   ├── squad-sr-cyr.json
│   ├── squad-sr-lat.json
│   └── final_train_dataset.json      # JSON Lines despite the .json extension
├── vector_database/
│   ├── index.faiss
│   └── index.pkl
└── .gitignore
```

Local directories `dataset_RAG/`, `lora_adapters/`, and `merged_model/` are excluded from Git. The 54 source leaflet PDFs and model weights are not included in this Git repository. The saved FAISS index and its metadata are included.

## Implementation and data

1. **Documents:** ALIMS leaflet PDFs are converted to Markdown using `pymupdf4llm` and split by headings with `MarkdownHeaderTextSplitter`. The recorded run produced 1,769 chunks.
2. **Retrieval:** `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` produces 768-dimensional embeddings. Despite its name, the multilingual checkpoint uses an XLM-RoBERTa backbone. The code uses default FAISS Euclidean-distance search without explicit embedding normalization, retrieving five chunks in the application.
3. **Adaptation:** 3,152 general Serbian instruction/QA examples are used for QLoRA training, rather than a dedicated medical training set. Training uses 4-bit NF4, double quantization, Float16 computation, rank 16, alpha 32, and one epoch (394 steps).
4. **Inference:** the merged model is loaded in 8-bit precision. The saved configuration uses `do_sample=False` and a 512-token output limit. Conversation history supports query reformulation for retrieval; the answer prompt receives the current question and retrieved text.
5. **Instructions:** the prompt requests context-grounded answers, matching script, simpler language, and a disclaimer. These behaviors are not independently enforced. Chunk metadata are stored but are not explicitly inserted into the generation prompt.

The training corpus combines 500 Cyrillic and 500 Latin [SQuAD examples](https://www.kaggle.com/datasets/aleksacvetanovic/squad-sr), 1,000 Cyrillic [saillab Alpaca examples](https://huggingface.co/datasets/saillab/alpaca-serbian-cleaned), 1,000 Latin [datatab Alpaca examples](https://huggingface.co/datasets/datatab/alpaca-cleaned-serbian-full), and 152 Serbian [Aya examples](https://huggingface.co/datasets/CohereLabs/aya_dataset). The instruction-field script split is 52.3% Cyrillic and 47.7% Latin.

## Running the project

The notebooks are the executable entry points. They were developed around Google Colab and Google Drive, rather than as a packaged command-line application. Dependencies are installed in notebook cells. There is no pinned, independently validated environment; a new runtime may require dependency adjustments.

### Prerequisites and paths

- Use a GPU runtime for training or inference. The reported training setup uses an NVIDIA T4 with approximately 15–16 GB VRAM. Model merging also requires sufficient CPU RAM and disk space.
- Obtain access to the gated base model and authenticate with your own Hugging Face account where required. Keep credentials out of committed notebook cells and outputs.
- Mount Google Drive. The default project directory in the Colab notebooks is `/content/drive/MyDrive/NLP_proyect`.
- Change `base_path` in `01_prepare_training_data.ipynb` to the directory containing the two SQuAD JSON files; its original value points to a local Windows cache.
- The training notebook expects `datasets/final_train_dataset.json` under the Drive project directory. Index creation expects `datasets/dataset_RAG`. The repository layout differs from the original Drive layout: copy inputs to those locations or edit the path variables.
- Set `db_path`, `model_id`/`model_path`, adapter paths, and output directories consistently. The application requires a compatible merged model and vector index. To use the Hugging Face model, set its identifier instead of the default Drive model directory.

### Notebook order

See the [notebook guide](notebooks/README.md) for entry points, conversation boundaries, and a mapping from the original filenames cited in the papers.

| Step | Notebook | Purpose |
| --- | --- | --- |
| 1 | [01_prepare_training_data.ipynb](notebooks/01_prepare_training_data.ipynb) | Sample source datasets and create the combined JSONL file |
| 2 | [02_explore_training_data.ipynb](notebooks/02_explore_training_data.ipynb) | Inspect missing values, lengths, scripts, and frequent terms |
| 3 | [03_finetune_and_merge.ipynb](notebooks/03_finetune_and_merge.ipynb) | Train adapters and merge them with base weights shard by shard |
| 4 | [04_build_retrieval_index.ipynb](notebooks/04_build_retrieval_index.ipynb) | Build the FAISS index from local leaflet PDFs |
| 5 | [05_run_assistant.ipynb](notebooks/05_run_assistant.ipynb) | Load the model and index and run conversation examples |
| 6 | [06_evaluate.ipynb](notebooks/06_evaluate.ipynb) | Evaluate retrieval and answers to isolated questions |
| 7 | [07_evaluate.ipynb](notebooks/07_evaluate.ipynb) | Fine-tuned model: test 1 with RAG and test 2 without RAG |
| 8 | [08_evaluate_base.ipynb](notebooks/08_evaluate_base.ipynb) | Base model in the same 8-bit configuration: test 3 without RAG and test 4 with RAG |

Index creation is independent of fine-tuning. With existing model and index artifacts, start with `05_run_assistant.ipynb` after configuring dependencies and paths. The supplied index can be reused without rebuilding the source PDFs. Only load pickle index metadata from a trusted source; the notebook explicitly enables deserialization.

Run cells in order and reset `chat_history` between independent conversations. The documented interface is notebook-based; Gradio imports and the Space link do not establish a verified production interface. New runs, particularly after cleaning data or changing dependencies, may differ from the recorded results.

## Follow-up evaluation

[Notebook 07](notebooks/07_evaluate.ipynb) and [notebook 08](notebooks/08_evaluate_base.ipynb) completed all four tests on 24 October 2026: 16 answers and 16 metric rows. Both RAG conditions received identical saved context and sources for every question; both no-RAG conditions received empty context. The original prompt and 8-bit loading settings were retained. See the [findings and source CSVs](results/README.md) and [execution guide](notebooks/README.md#running-the-new-evaluation). These results supplement the PDFs and do not replace their original experiment.

RAG improved all four mean similarity metrics for both models on this pilot. With RAG, the base model scored higher than the fine-tuned model on all four mean metrics (BERTScore F1: 0.7816 versus 0.7562; ROUGE-L: 0.3758 versus 0.2726). This run does not demonstrate a benefit from the additional fine-tuning under these conditions. Both models still show factual, language and instruction-following failures. Appropriate refusal with empty context must be assessed separately from similarity to answer-bearing references. Four development questions do not support broad conclusions about fine-tuning or medical reliability.

## Further work

Priorities are extending the completed comparison to a larger independent test set, expert factual and instruction-following assessment, script-aware metrics, and training-data cleaning. Further experiments can examine matching training and inference prompt formats, targeted adaptation data, hybrid retrieval, explicit medicine/source metadata, reranking, and adaptive retrieval depth. A separate end-to-end comparison can measure history-aware query reformulation. Record exact environment and checkpoint versions in future runs; the completed base run retains a Colab `requests` dependency warning.

## Licensing

No repository-wide license has been declared. The model, datasets, and source documents have their own terms; their inclusion or links here do not replace those terms.
