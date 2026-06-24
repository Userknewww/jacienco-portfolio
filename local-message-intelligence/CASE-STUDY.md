# Case Study — Local Message Intelligence: On-Device Retrieval & Analysis

## The challenge
I wanted to run analytical queries against a personal message archive of more than 28,000 records without any of it touching the cloud. The privacy requirement was absolute: the data had to stay on one device. That ruled out the easy path of uploading exports to a hosted model, and it turned a simple "ask questions about my data" idea into a real systems problem — local ingestion, local storage, local search, and a local model, all working together on a single machine.

## Discovery
I started from a proposed architecture that looked reasonable and was, on inspection, broken in several ways. It re-embedded the entire archive every sixty seconds, which would never keep up. It reached for a semantic vector database for questions that were actually statistical and lexical. It hardcoded the wrong system user in its scheduling configuration. And it stored plaintext message exports inside a cloud-synced folder — the exact opposite of the privacy goal. Catching those before building anything saved the project; each was the kind of flaw that looks fine on paper and fails in practice.

## The solution I designed
I replaced the proposed design with a lean on-device pipeline. A SQLite database with an FTS5 full-text index stores the corpus in a permission-locked directory outside any synced folder. The ingester reads only new rows using a stored row-id watermark, and it decodes the binary `attributedBody` blobs that modern macOS uses for message bodies, so messages with empty plain-text fields are captured rather than lost. A local Ollama model answers questions grounded in SQL-derived statistics and full-text excerpts. There is no Docker, no vector store, and no cloud call anywhere in the path. The architecture diagram in the README shows the full flow.

## The technical decisions
First, full-text search over embeddings: the queries were counts, dates, and term matches, so FTS5 was both lighter and more accurate than a vector database would have been. Second, incremental ingestion with a watermark instead of full re-indexing, which made each update cheap rather than reprocessing tens of thousands of rows every cycle. Third, decoding the binary body blob, without which a large share of modern messages would have been silently dropped. Fourth, when the first local model returned thin, one-word answers on analytical prompts, I rewrote the query path to use a chat-style call with explicit system and user roles and a larger output budget, and identified a move to a larger local model as the durable fix.

## The results
The pipeline ingested 28,360 of 28,761 source rows; the remaining rows were reactions and attachments with no text content, which is expected. The system runs entirely on-device, in a permission-locked directory outside cloud sync, and answers aggregate questions about the archive without any external API call. The privacy goal held end to end.

## What I would do differently
I would have profiled model capability against the actual analytical prompts before settling on the smallest local model, because that choice was the main source of weak answers and the cause of a mid-project rewrite. Choosing the right-sized local model up front, and budgeting the memory for it, would have removed an entire debugging cycle.
