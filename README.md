# AI Model as a Search Assistant for Medication Instructions

This repository contains a research and implementation project focused on building an AI-based question-answering system for medication instructions. The system is designed for the Serbian language and combines a retrieval-augmented generation (RAG) pipeline with parameter-efficient fine-tuning using QLoRA.

The goal is to provide accurate, context-aware answers to questions related to drug instructions, dosage, warnings, contraindications, and safe usage, while reducing hallucinations by grounding the model in relevant retrieved sources.

## Overview

Large language models (LLMs) are highly capable, but in domain-specific tasks such as medicine, they can generate answers that sound convincing while being factually incorrect. To address this issue, the project integrates:

- a document retrieval layer,
- a vector database for semantic search,
- a base LLM adapted with QLoRA,
- a generation pipeline grounded in retrieved evidence.

The final system is intended to function as a search assistant that answers medication-related queries with higher factual reliability than a standalone generative model.

## System architecture

The project follows a standard RAG-based design:

1. Relevant pharmaceutical documents are converted into chunks.
2. Text chunks are embedded into a vector space.
3. A retriever searches the vector database for semantically similar content.
4. The most relevant results are injected into the model context.
5. A fine-tuned LLM generates a final answer grounded in the retrieved passages.

This architecture is particularly useful in clinical and pharmaceutical domains, where factual precision and traceability are essential.

## Repository structure

```text
projekat/
├── dataset_LoRA/                            # JSON datasets used for LoRA fine-tuning
├── dataset_RAG/                             # Not included in GitHub uploads; contains PDF documents for retrieval
├── lora_adapters/                           # Not included in GitHub uploads
├── merged_model/                            # Final merged model; to be published on Hugging Face later
├── notebooks/
│   ├── app.ipynb                           # Final model inference and usage
│   ├── dataset_LoRA.ipynb                  # Dataset concatenation and preparation
│   ├── evaluation.ipynb                    # Numerical evaluation and benchmarking
│   ├── final_train_dataset_EDA.ipynb       # Exploratory data analysis for the fine-tuning set
│   ├── LLM_finetuning.ipynb                # QLoRA fine-tuning and LoRA adapter merging
│   └── vector_database_creation.ipynb      # FAISS vector database creation
├── vector_database/
│   ├── index.faiss                         # FAISS index
│   └── index.pkl                           # Embedding / metadata store
├── izvestaj.pdf                            # Serbian version of the project report
├── report.pdf                              # English version of the project report
├── README.md
├── token_Llama.txt                         # Local token file, not intended for GitHub
├── token_LoRA_datasetovi.txt               # Local token file, not intended for GitHub
└── .gitignore                              # Local ignore rules for large or sensitive files
```

## Data and artifacts

### `dataset_LoRA/`
This directory contains the JSON files used for LoRA fine-tuning. The datasets were prepared and combined into a unified training set for domain adaptation of the base LLM.

### `dataset_RAG/`
This folder contains PDFs used for retrieval in the RAG pipeline. It is intentionally excluded from GitHub because it is not intended for public publication.

### `lora_adapters/`
This folder stores the trained LoRA adapter weights. It is excluded from the public repository because it contains model artifacts generated locally.

### `merged_model/`
This directory contains the final merged model, which is expected to be published on Hugging Face after the project is finalized. The model link should be added here once available.

- Hugging Face model: [https://huggingface.co/vekiii/medical-assistent-serbian-model]
- Hugging Face Space: [https://huggingface.co/vekiii/spaces] - Medical Assistant Serbian
### `vector_database/`
The vector database contains the embeddings and index used for semantic retrieval. It stores the FAISS index and metadata required to find the most relevant context for a given user question.

## Notebooks

The project notebooks are organized by functional stage:

- `app.ipynb`: runs inference with the final model and demonstrates the end-to-end usage workflow.
- `dataset_LoRA.ipynb`: merges and prepares data for the LoRA fine-tuning set.
- `evaluation.ipynb`: evaluates model quality using numerical metrics.
- `final_train_dataset_EDA.ipynb`: analyzes the dataset before training.
- `LLM_finetuning.ipynb`: performs QLoRA fine-tuning and merges adapters with the base LLM.
- `vector_database_creation.ipynb`: creates the retrieval index used by the RAG component.

## Methods used

This project incorporates several modern NLP and generative AI techniques:

- Transformer-based language models
- Sentence embedding models for semantic retrieval
- FAISS for efficient vector search
- LoRA and QLoRA for memory-efficient fine-tuning
- Retrieval-augmented generation for factual grounding

The combination of these methods is appropriate for a domain-specific QA use case where the model needs to answer using reliable, context-based evidence rather than relying only on memorized patterns.

## Usage workflow

To reproduce or continue the project locally:

1. Prepare the source datasets and generate the final LoRA dataset.
2. Run the EDA notebook to inspect dataset quality and structure.
3. Execute the QLoRA fine-tuning workflow.
4. Merge the trained adapters into the base model.
5. Build the FAISS vector database.
6. Run inference through `app.ipynb`.

## Project status

This README represents the current public-facing base version of the project. It will be refined after the final model, public artifacts, and links are added.

## License

No explicit license has been declared yet. A license should be added before publishing the repository publicly.

## Notes

This project is intended as a practical implementation of a domain-specific QA system in Serbian, combining research ideas from modern transformer architectures, efficient fine-tuning, and retrieval-based augmentation. The current implementation is a strong foundation for future improvements, including better evaluation, more robust retrieval, and production-quality deployment.
