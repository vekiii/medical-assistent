# Project Summary

## What I implemented

For a master's-level NLP course, I implemented a Serbian-language question-answering prototype for medicine package leaflets. My contribution was assembling the application pipeline and conducting the experiments using existing pretrained models and libraries, rather than developing a language model from scratch.

I collected and processed 54 leaflet PDFs, built a retrieval index, prepared a combined Serbian instruction/QA dataset, configured and ran QLoRA fine-tuning, and integrated the resulting model with retrieval and a system prompt. I also implemented conversation-history handling for query reformulation and notebooks for qualitative and numerical evaluation. To work around memory limits during model merging, I used a custom shard-by-shard procedure on the CPU.

## Main technical choices

I combined RAG with fine-tuning for different purposes: retrieval supplies pharmaceutical information from external documents, while additional supervised fine-tuning was intended to improve Serbian-language instruction following and QA behavior. I did not use fine-tuning as a substitute for the leaflet knowledge base, and the experiment does not establish its independent contribution.

The retrieval pipeline converts PDFs to Markdown, splits them by headings, and stores 1,769 chunks in FAISS using multilingual sentence embeddings. The generator is Llama-3.1-8B-Instruct. Limited Google Colab resources motivated 4-bit QLoRA training on a T4 GPU, with rank-16 adapters and one epoch over 3,152 examples from Serbian SQuAD, Alpaca, and Aya sources. The merged model is loaded in 8-bit precision for inference. The application supplies the system instructions, so users can ask ordinary questions; however, compliance was inconsistent in the recorded examples.

## How I evaluated the results

I examined example conversations in both Serbian scripts, comparing answers with leaflet excerpts and checking follow-up questions, missing information, and out-of-domain queries. I also compared outputs with those of the original model without RAG.

For four selected questions, I measured retrieval precision, recall, hit rate, and reciprocal rank, and assessed generated answers with ROUGE-L, BLEU, BERTScore, and embedding cosine similarity. Mean Precision@5 was 0.90 and MRR was 0.875, with relevance defined by source filename. These preliminary scores do not establish answer correctness. Qualitative analysis revealed factual contradictions, medicine confusion, grammatical errors, and prompt-following failures. The small test set and combined changes to retrieval, training, and precision limit causal conclusions.

## What I would improve

My first priority would be comparing the base model plus RAG against the fine-tuned model plus the same RAG, holding prompts, retrieved context, precision, decoding settings, and test questions constant. This would isolate the value of additional fine-tuning.

With more time and resources, I would build an independently annotated Serbian pharmaceutical QA test set, obtain expert factual assessment, clean the training data, and align training and inference prompt formats. I would investigate better-targeted training examples, script-aware evaluation, and hybrid retrieval or reranking. These steps would address the prototype's observed limitations before drawing stronger conclusions about reliability.
