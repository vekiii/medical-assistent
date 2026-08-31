# AI Model as a Search Assistant for Medication Instructions

This project explores the development of an AI-based assistant for searching and answering questions about medication instructions. The system combines a retrieval-augmented generation (RAG) pipeline with a LoRA-fine-tuned large language model to provide contextual and domain-aware responses based on pharmaceutical documentation.

The goal is to help users find relevant drug guidance quickly and reliably by leveraging a vector database and a specialized LLM adapted to the task.

## Project overview

The repository contains the main data, notebooks, and supporting artifacts used for:

- preparing and cleaning the fine-tuning dataset,
- performing EDA on the training data,
- fine-tuning a base LLM with QLoRA,
- combining the LoRA adapters with the base model,
- creating and indexing a vector database,
- evaluating the system,
- running the final model for inference.

## Repository structure

```text
projekat/
├── dataset_LoRA/                 # JSON files used for LoRA fine-tuning
├── dataset_RAG/                  # Not included in GitHub uploads; contains PDF documents for RAG
├── lora_adapters/                # Not included in GitHub uploads
├── merged_model/                 # Final merged model; hosted on Hugging Face (insert link later)
├── notebooks/
│   ├── app.ipynb                 # Runs the final model for inference and usage
│   ├── dataset_LoRA.ipynb        # Concatenates datasets and creates the files inside dataset_LoRA
│   ├── evaluation.ipynb          # Performs numerical evaluation of the model
│   ├── final_train_dataset_EDA.ipynb  # Exploratory data analysis for the fine-tuning dataset
│   ├── LLM_finetuning.ipynb      # Fine-tunes the base LLM via QLoRA and merges LoRA adapters
│   └── vector_database_creation.ipynb  # Builds the vector database used for retrieval
├── vector_database/
│   ├── index.faiss               # FAISS vector index
│   └── index.pkl                 # Metadata / embedding index file
├── report.pdf                    # Theoretical project report in English (to be added later)
├── izvestaj.pdf                  # Theoretical project report in Serbian (to be added later)
└── README.md
```

## Important notes

- The `dataset_RAG/` directory contains PDF documents used for retrieval and is intentionally excluded from GitHub uploads.
- The `lora_adapters/` directory is not included in the public repository.
- The `merged_model/` directory is also excluded from GitHub, but the final model is expected to be published on Hugging Face. Add the model link here once available.
- `report.pdf` and `izvestaj.pdf` are currently not present in the folder but will be added later.

## Data

### `dataset_LoRA/`
This directory contains the JSON datasets used for LoRA fine-tuning of the LLM. These files are the primary training artifacts used for adapting the base model to the medication-instruction domain.

## Notebooks

The notebooks in `notebooks/` are organized by task:

- `app.ipynb`: runs inference with the final model and demonstrates the end-to-end usage flow.
- `dataset_LoRA.ipynb`: combines the source datasets and produces the final files stored in `dataset_LoRA/`.
- `evaluation.ipynb`: computes numerical metrics to assess model quality and performance.
- `final_train_dataset_EDA.ipynb`: performs exploratory analysis of the fine-tuning dataset to understand structure, quality, and balance.
- `LLM_finetuning.ipynb`: fine-tunes the base LLM with QLoRA and merges the trained adapters into the base model.
- `vector_database_creation.ipynb`: builds the vector database used for retrieval-based grounding.

## Vector database

The `vector_database/` folder stores the FAISS index and the corresponding metadata used to retrieve relevant documents during the question-answering pipeline.

## Model

The final model is intended to be available on Hugging Face and can be linked here once the publishing step is completed.

Example placeholder:

- Hugging Face model: [Add model link here]

## Usage

1. Open the notebooks in `notebooks/` in order of the pipeline.
2. Run the data preparation steps and create the LoRA dataset.
3. Execute QLoRA fine-tuning and adapter merging.
4. Build the vector database.
5. Use `app.ipynb` to run inference on the final model.

## Project status

This README represents the base version of the project structure and documentation. It will be updated and refined as the project evolves and additional files, reports, and model links are added.

## License

This project does not currently declare a license. If needed, a license should be added before publishing the repository publicly.
