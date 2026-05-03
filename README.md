# Different Ways to Chunk Podcast and PDF

Lab for exploring how to split podcast transcripts and PDF documents for use in a **Retrieval-Augmented Generation (RAG)** system. 
---

# Main file
* chunking_strategies.ipynb

Source files:
- Podcast: The_Blueürint_For_Trustworthy_AI_transcript.txt
- PDF: ethics_guidelines_for_trustworthy_ai_text.txt

---
## Chunking Strategies Covered

### Fixed-Size Chunking (`CharacterTextSplitter`)  -> Step 2 (in main file). Refer to Fixed_size_chunking_output.md for results analysis 
- Splits text at a fixed character count (e.g. 500, 1000, 2000 chars)
- Overlap (0–100 chars) helps reduce context loss at boundaries

### Recursive Character Chunking (`RecursiveCharacterTextSplitter`) -> Step 3 (in main file). Refer to recursive_character_chunking_output.md for results analysis.
- Tries to split on natural boundaries in priority order: `\n\n` → `\n` → `. ` → ` ` → `""`

### Token-Based Chunking (`TokenTextSplitter`)  -> Step 4 (in main file). Refer to Token_base_chunking_output.md for results analysis.
- Splits by token count rather than character count — more accurate for LLM context windows
- Uses `tiktoken` (e.g. `cl100k_base` encoding) to verify actual token counts

* **Comprehensive analysis:** chunking_recommendations.md
