# Project Summary

## What I implemented

For a master's-level NLP course, I implemented a Serbian-language question-answering prototype for medicine package leaflets. My contribution was assembling the application pipeline and conducting the experiments using existing pretrained models and libraries, rather than developing a language model from scratch.

I collected and processed 54 leaflet PDFs, built a retrieval index, prepared a combined Serbian instruction/QA dataset, configured and ran QLoRA fine-tuning, and integrated the resulting model with retrieval and a system prompt. I also implemented conversation-history handling for query reformulation and notebooks for qualitative and numerical evaluation. To work around memory limits during model merging, I used a custom shard-by-shard procedure on the CPU.

## Main technical choices

I combined RAG with fine-tuning for different purposes: retrieval supplies pharmaceutical information from external documents, while additional supervised fine-tuning was intended to improve Serbian-language instruction following and QA behavior. I did not use fine-tuning as a substitute for the leaflet knowledge base. A later controlled pilot did not show an improvement from this additional training when identical RAG context was supplied.

The retrieval pipeline converts PDFs to Markdown, splits them by headings, and stores 1,769 chunks in FAISS using multilingual sentence embeddings. The generator is Llama-3.1-8B-Instruct. Limited Google Colab resources motivated 4-bit QLoRA training on a T4 GPU, with rank-16 adapters and one epoch over 3,152 examples from Serbian SQuAD, Alpaca, and Aya sources. The merged model is loaded in 8-bit precision for inference. The application supplies the system instructions, so users can ask ordinary questions; however, compliance was inconsistent in the recorded examples.

## How I evaluated the results

I examined example conversations in both Serbian scripts, comparing answers with leaflet excerpts and checking follow-up questions, missing information, and out-of-domain queries. I also compared outputs with those of the original model without RAG.

For four selected questions, I measured retrieval precision, recall, hit rate, and reciprocal rank, and assessed generated answers with ROUGE-L, BLEU, BERTScore, and embedding cosine similarity. Mean Precision@5 was 0.90 and MRR was 0.875, with relevance defined by source filename. These preliminary scores do not establish answer correctness. Qualitative analysis revealed factual contradictions, medicine confusion, grammatical errors, and prompt-following failures. The small test set and combined changes to retrieval, training, and precision limit causal conclusions.

After the seminar experiment, I ran a [four-condition pilot](results/README.md): base and fine-tuned models, each with and without RAG, at the same 8-bit precision. All 16 answers and metric rows were produced, and saved context was identical across the two RAG conditions. RAG improved all four average similarity metrics for both models. With RAG, the base model exceeded the fine-tuned model on all four averages (BERTScore F1: 0.7816 versus 0.7562). This sample therefore did not demonstrate the intended benefit of additional fine-tuning. Refusal and factual correctness require separate assessment; these are four development questions rather than an independent benchmark.

## What I would improve

My first priority would be extending that comparison to a larger held-out set with expert ratings and exact environment/checkpoint provenance.

With more time and resources, I would build an independently annotated Serbian pharmaceutical QA test set, obtain expert factual assessment, clean the training data, and align training and inference prompt formats. I would investigate better-targeted training examples, script-aware evaluation, and hybrid retrieval or reranking. These steps would address the prototype's observed limitations before drawing stronger conclusions about reliability.
