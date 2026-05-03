# Lab Summary: Chunking Strategies

This lab compared three chunking approaches for two different source types: a podcast transcript about trustworthy AI and a PDF document, *Ethics Guidelines for Trustworthy AI*. The notebook first prepared the inputs by transcribing the audio and extracting the PDF text, then tested fixed-size character chunking, recursive character chunking, and token-based chunking across several chunk sizes and overlaps.

## Strategy Summary

**Fixed-size chunking** split the text by character count using sizes of 500, 1000, and 2000 characters with overlaps of 0, 50, and 100. This method was simple and predictable, but it often cut through sentences and even words because it did not understand natural boundaries. It worked somewhat better for the structured PDF than for the conversational podcast, but boundary quality remained weak.

**Recursive character chunking** improved the results by trying to split on larger natural separators first, such as paragraphs, new lines, and sentences. This preserved sentence and section boundaries better than fixed-size chunking, especially in the PDF where headers and page structure gave the splitter useful signals. For the podcast, it was still only a modest improvement because the transcript had less formal structure.

**Token-based chunking** split content by model tokens instead of characters, using `tiktoken` and `TokenTextSplitter`. This gave the most predictable control over how much text would be sent to an embedding or language model. It also produced fewer chunks than character-based methods while keeping chunks close to the intended token limits.

## Result Analysis

The fixed-size results showed that chunk size had the largest impact on the number of chunks: smaller chunks created much more granular retrieval units, while larger chunks reduced total chunk count. Overlap increased chunk count, but its effect became smaller at larger chunk sizes. A reasonable fixed-size choice was 1000 characters with 50-100 overlap for the podcast, and 500 characters with 100 overlap for the PDF when fine-grained retrieval was preferred.

Recursive chunking was a clear upgrade for structured text. It preserved sentence boundaries better, respected PDF section headers more effectively, and reduced the problem of arbitrary cuts. However, for the podcast transcript, recursive splitting was limited by the lack of paragraph-like structure, so it improved readability but did not fully solve boundary issues.

Token-based chunking appeared to be the strongest production choice. It offered better efficiency, lower embedding cost, and tighter alignment with how models consume input. The podcast chunks had strong fill rates, around 92% at 500 tokens and about 85% at 2000 tokens. The PDF filled less efficiently at smaller token sizes because of page breaks and sparse header regions, but at larger sizes both documents converged around 84-85% fill rate.

Overall, the lab showed that fixed-size chunking is useful as a baseline, recursive chunking improves semantic boundaries, and token-based chunking is best suited for production RAG because it balances context preservation, model compatibility, and cost control.
