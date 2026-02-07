# Hepsiburada Address Matching – Hybrid Retrieval Pipeline

This repository contains an end-to-end **hybrid address matching and reranking pipeline**
developed for the **Teknofest Hepsiburada Address Matching Kaggle competition**.

This codebase:
- Achieved the **highest score in the Kaggle leaderboard phase**
- Won 🥇 **1st place** after the final technical presentation and evaluation stage of the competition

---

## General Approach

The task requires matching free-form address texts to the correct address cluster.
Since a single method is not sufficient, a combination of
**dense + sparse retrieval** together with a **learned reranker** is used.

The overall flow is as follows:

1. Turkish-specific address normalization  
2. Dense embedding-based retrieval (BGE-M3)  
3. Sparse lexical retrieval (BM25)  
4. Candidate address generation by combining both methods  
5. Final selection using a BERT-based reranker  

---

## Architecture

- **Dense Retrieval:** BGE-M3  
- **Sparse Retrieval:** BM25  
- **Reranker:** dbmdz/bert-base-turkish-cased  
- **Training Strategy:** Hard Negative Mining  
- **Execution Environment:** GPU-enabled (Colab compatible)

---

## Code Structure

### 1. Configuration (`Config`)
All file paths, model names, batch sizes, and training parameters
are managed from a single configuration class.

Key parameters:
- **30 nearest neighbors** from dense retrieval  
- **30 nearest neighbors** from BM25 retrieval  
- **20 nearest neighbors** for hard negative mining  
- Dense and sparse results are **merged (union)** to form the candidate pool

---

### 2. Text Preprocessing
- Turkish-specific lowercasing  
- Normalization of address abbreviations (mah, cad, sk, etc.)
- Number–word separation

The goal is to improve both semantic and lexical retrieval quality.

---

### 3. Dense Embedding and GPU Search
- All training addresses are embedded using **BGE-M3**
- Cosine similarity–based **GPU-optimized** search is applied
- For each test address, the **top 30 nearest neighbors** are retrieved

Dense retrieval captures semantic similarity even when addresses are
not lexically identical.

---

### 4. BM25 Sparse Retrieval
- Preprocessed addresses are tokenized
- **BM25** is used for term-frequency–based lexical matching
- **Top 30 candidates** are retrieved per test address
- Dense and sparse candidate lists are combined using a union operation

This step ensures that critical tokens such as street names or building numbers
are not missed.

---

### 5. Hard Negative Mining
- Training addresses are searched against each other using dense retrieval
- For each address, **20 nearest neighbors** are selected
- Pairs with the same label are treated as positives
- Semantically similar but label-mismatched pairs are used as **hard negatives**
- A balanced positive/negative training set is created

This significantly improves the discriminative power of the reranker.

---

### 6. BERT Reranker Training
- Input: (query_address, candidate_address)  
- Output: matching probability  
- A Turkish BERT model is fine-tuned from scratch
- The best model is selected based on **F1 score**

The reranker focuses on learning fine-grained distinctions
between highly similar address candidates.

---

### 7. Final Reranking and Submission
- Candidate addresses for each test query are scored by the BERT reranker
- The candidate with the highest probability is selected
- In rare cases with no candidates, the majority label is assigned
- A Kaggle-compatible submission file is generated

---

## Why This Approach?

- **Dense retrieval** captures semantic similarity  
- **Sparse retrieval (BM25)** preserves critical token-level matches  
- **Reranking** resolves subtle ambiguities between close candidates  
- Hard negative mining forces the model to learn from difficult examples  

This hybrid design provides a clear performance advantage
over single-method approaches.

---

## Results

This pipeline:
- Achieved the **highest score** during the Kaggle leaderboard phase  
- Won **first place** after the final technical presentation and evaluation
  in the Teknofest Hepsiburada competition

---

