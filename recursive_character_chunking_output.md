### Analysis questions:

* Does recursive chunking preserve sentence boundaries better?
Yes, is preserving it better when I see the results. 
* How does it handle the podcast's conversational structure?
It's partially improved compared to fixed-size. But maybe recursive still is not the best way to handle content that has no structure.
* Does it respect PDF section headers?
Yes, it handles them better compared to the fixed-size. The separator aligns here with the pdf text.

In conclusion: recursive chunking is an upgrade for structured text like the PDF, and a modest improvement for the podcast.

Available recursive chunk sets:
  podcast_default_size_500: 53 chunks
  podcast_default_size_1000: 23 chunks
  podcast_default_size_2000: 12 chunks
  pdf_default_size_500: 500 chunks
  pdf_default_size_1000: 209 chunks
  pdf_default_size_2000: 104 chunks
  podcast_paragraph_size_500: 54 chunks
  podcast_paragraph_size_1000: 23 chunks
  podcast_paragraph_size_2000: 12 chunks
  pdf_paragraph_size_500: 500 chunks
  pdf_paragraph_size_1000: 209 chunks
  pdf_paragraph_size_2000: 104 chunks
  podcast_sentence_size_500: 51 chunks
  podcast_sentence_size_1000: 20 chunks
  podcast_sentence_size_2000: 9 chunks
  pdf_sentence_size_500: 499 chunks
  pdf_sentence_size_1000: 205 chunks
  pdf_sentence_size_2000: 96 chunks

--- Example PDF chunk | strategy: default | size: 500 ---
[Page 1]
INDEPENDENT  
HIGH-LEVEL EXPERT GROUP ON  
ARTIFICIAL INTELLIGENCE  
SET UP BY THE EUROPEAN COMMISSION  
 
 
 
 
 
 
ETHICS GUIDELINES  
FOR TRUSTWORTHY AI

--- Example Podcast chunk | strategy: default | size: 500 ---
[Part 1]

--- Example PDF chunk | strategy: paragraph | size: 500 ---
[Page 1]
INDEPENDENT  
HIGH-LEVEL EXPERT GROUP ON  
ARTIFICIAL INTELLIGENCE  
SET UP BY THE EUROPEAN COMMISSION  
 
 
 
 
 
 
ETHICS GUIDELINES  
FOR TRUSTWORTHY AI

--- Example Podcast chunk | strategy: paragraph | size: 500 ---
[Part 1]

--- Example PDF chunk | strategy: sentence | size: 500 ---
[Page 1]
INDEPENDENT  
HIGH-LEVEL EXPERT GROUP ON  
ARTIFICIAL INTELLIGENCE  
SET UP BY THE EUROPEAN COMMISSION  
 
 
 
 
 
 
ETHICS GUIDELINES  
FOR TRUSTWORTHY AI

[Page 2]
ETHICS GUIDELINES  FOR  TRUSTWORTHY  AI 
 
High -Level Expert Group on Artificial Intelligence  
 
 
 
 
 
 
 
 
This document was written by the High -Level Expert Group on AI (AI HLEG)

--- Example Podcast chunk | strategy: sentence | size: 500 ---
[Part 1]
So, imagine for a second you're driving across a, I don't know, a massive suspension bridge. Okay. You don't pull over halfway across, get out, and demand to see the blueprints, right? You don't interview the welding crew. No, you just trust it. You just drive. You trust the bridge. You trust the engineering standards, the inspections, the laws that say this thing won't fail. Right. It's trust in the infrastructure. It's invisible, but it's there. Exactly. But now, let's switch gears
