# 📚 PDF Study Buddy

A chatbot that reads a PDF and can **answer questions**, **make quizzes** and **summarize**, using only what is in the document.
Built with RAG (retrieval-augmented generation), a local LLM and an agent that picks the right tool.

**Author:** Hilal  
**Notebook:** `Study_buddy_Chatbot.ipynb` (runs in Google Colab)

## What it does
- **Answers questions from the PDF.** If the PDF does not cover the question, it says "I don't know" instead of guessing.
- **Quizzes.** Ask for any number of questions (1-20). In the web page you answer one by one, see ✅/❌ at once with the sentence from the PDF that proves the answer, and get a final score.
- **Summaries** of the whole PDF or of one topic.
- **Chat memory.** Follow-up questions such as "How is it used in RAG?" are understood.
- **An agent** reads your message and chooses: answer, quiz, summary, or show quiz answers.

## How it works (short)
```
PDF → clean text → chunks (~150 words) → embeddings → Pinecone (vector database)

Question → Pinecone returns 12 close chunks → reranker keeps the best 4
        → Llama 3.1 8B answers using only those chunks (or says "I don't know")
        → a grounding check replaces answers that use words not in the PDF with a real quote
```

## Tech used
| Part | Tool |
|---|---|
| Language model | Llama 3.1 8B through Ollama (local) |
| Embeddings | `all-MiniLM-L6-v2` (384 numbers per chunk) |
| Reranker | `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| Vector database | Pinecone (serverless, cosine) |
| PDF reading | pypdf |
| Web page | Gradio (Chat tab + Quiz tab) |
| Runs on | Google Colab (T4 GPU recommended) |

## How to run
1. Open the notebook in Google Colab and choose **Runtime → Change runtime type → T4 GPU**.
2. Create a free Pinecone account and copy your API key. Add it as a Colab Secret named `PINECONE_API_KEY` (or paste it when asked).
3. Run the notebook **from the top**. It asks you to upload a PDF.
4. In Part 11, run the launch cell. Open the public link to use the Chat and Quiz tabs.

On a CPU-only runtime, set `MODEL = "llama3.2:3b"` in cell 1.4 (the 8B model is too slow on CPU).

## Results
Tested on a 46-question set written for the sample PDF (Wikipedia article "Retrieval-augmented generation"):

| Measure | Result |
|---|---|
| Right page found in top 4 chunks | 100% |
| Correct answers on PDF questions | 97% (37 of 38) |
| Off-topic and trap questions refused | 100% (8 of 8) |
| Overall | 98% |
| Answers using outside knowledge | 0 |

The same score appeared in repeated runs. The one failing question ("What abilities do augmentation modules add...") is answered from a passage with a similar word.
With the smaller 3B model on the same chunks, 3 answers used outside knowledge.

## Limitations
- The test set and the cutoffs were made for one PDF. A second PDF has not been tested yet.
- Scoring uses keywords, so failures are read by hand.
- Quiz questions can still be easy or imprecise, and a quiz may have fewer questions than requested (bad ones are dropped on purpose).
- A topic that the PDF barely mentions gives a short quote instead of a summary.
- A small local model can still make mistakes. The grounding check catches many, not all.
- Quiz answers shown in chat and the summary cache are shared by everyone using the public link.

## More
See `DOCUMENTATION.md` for a part-by-part explanation, the evaluation method, the problems found and how they were fixed.
