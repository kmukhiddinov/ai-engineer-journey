# AI Engineer Journey

Notes and projects from learning AI engineering: Claude API, prompt engineering, RAG, agents. Updated as I go through each notebook.

## exercise-on-multi-turn.ipynb

Built my first multi-turn chat with the Claude API.

How it works: user messages get appended to a shared list (role + content) → Claude's replies get appended the same way → that full list gets passed into the chat function along with model and max_tokens → API returns the response text → next turn builds on the same list, so Claude always sees the whole conversation, not just the latest message.

## building-with-the-claude-api.ipynb

**Request flow.** Client sends a message, it goes to your backend (never directly from the frontend), your backend calls the Claude API with your key, Claude responds back to your backend, backend responds to the client. The API key can never live in frontend code, anyone can open dev tools and read it. The backend is the only place that's private, so it has to sit in the middle.

**Tokens and embeddings.** Say 1 word = 1 token (an oversimplification, but useful as a mental model). Each word gets converted into an embedding, a long list of numbers acting as a numeric definition of that word. A word can have different meanings, what actually pins down the meaning is its place in the sentence, the context.

**Before output reaches the user**, the model checks itself: did it hit the max token limit, did it generate a token matching a stop sequence, did it output an end-of-sequence token. `max_tokens` is a ceiling, not a target, Claude can stop earlier if it decides the answer is complete.

## response-streaming.ipynb

**Standard vs streaming responses.** Standard waits for Claude's full response before sending anything back to the user. Streaming sends an initial acknowledgment immediately, then the response arrives as a series of chunks, words appearing one by one.

## system-prompt-temperature.ipynb

**Structured output (JSON, code, lists).** Claude tends to wrap structured output in its own helpful commentary, even when you just want the raw data. The fix is prefilling the assistant's message with the expected start (like &#96;&#96;&#96;json) so Claude sees it's already begun and continues from there, then cutting it off with a stop_sequence at the natural end (like &#96;&#96;&#96;).

**Temperature.** A 0-1 value controlling how deterministic vs creative the output is.
- Low (0-0.3): data extraction, factual responses, content moderation
- Medium (0.4-0.7): summarization, educational content, problem-solving
- High (0.8-1): brainstorming, creative writing, marketing content

Lower temp, less randomness. Higher temp, more creative variation.

## 001_prompt_evals.ipynb

**Typical prompt evaluation workflow:** draft a prompt → create an eval dataset → feed through Claude → feed through a grader → change the prompt and repeat.

**Example walkthrough:** draft prompt ("Please answer the user's questions"), eval dataset (questions like "what's 2+2", "how do I make oatmeal", "how far away is the moon"), feed through Claude to get responses, feed through a grader to get scores per response, average them. If a revised prompt scores higher on average, it's the better prompt. Still a rough signal, not a perfect one, but better than guessing.

**Mini project:** a prompt that needs to assist in writing three specific output types for AWS use cases: Python code, JSON, regular expressions.

1. **Generating test datasets.** Start with a version 1 prompt, ask Claude to generate test objects, store them in a file.
2. **Running the eval.** `run_prompt` merges a test case with the prompt template. `run_test_case` orchestrates running a single test case and grading the result. `run_eval` coordinates the entire evaluation process across all test cases.

**Types of graders:**
- **Code**: programmatic evaluation, checking output length, verifying syntax, checking for certain words
- **Model**: asking a model to assign a score to the output and compare versions, useful for response quality, completeness, safety, helpfulness
- **Human**: asking a person to assign a score, useful for general response quality, depth, relevance, conciseness

**Code grader vs model grader, the actual difference:** code grader checks whether the output is well-formed, does it parse, is the JSON valid, is the regex valid, is the Python syntactically correct, no explanation text mixed in. It's a pass/fail on structure. Model grader checks whether the output is actually right, does it directly and clearly address the task, is it accurate, does it make sense as an answer, not just as valid syntax.

## model-grader1.ipynb

A concrete model grader implementation: give the model a role ("you are an expert code reviewer"), pass it the task and the solution, and ask it to return the evaluation in a structured format:
- strengths
- weaknesses
- reasoning
- score

---

## prompt-engineering-techniques.ipynb

Mini project: a prompt that takes a person's height, weight, goal, and dietary restrictions, and outputs a structured diet/fitness plan.

First eval scored 3/10. The prompt was rough and the model choice was weaker, so before touching anything, the initial prompt and evaluation criteria got written down first. From there, techniques got applied one at a time, checking the score after each pass to see if it actually moved.

**Be clear and direct.** Clear means simple language, stating the task explicitly, leading with a plain statement of what the model needs to do. Direct means using instructions, not questions, action verbs like "write," "create," "generate." Example, clear: "Write 3 paragraphs about how solar panels work." Direct: "Identify three countries that use geothermal energy. Include generation stats for each."

**Be specific.** List the qualities the output should have, then give the steps the model should follow, guidelines or a numbered process. Close to a default technique, worth using on almost every prompt, especially for troubleshooting complex problems, decision-making scenarios, or critical thinking tasks.

**Use XML tags.** Wrap distinct chunks of content so Claude can tell them apart, `<sales_records>`, `<example>`, `<my_code>`, whatever fits the content.

**Provide examples.** Give sample input and sample output pairs, wrapped in XML tags. For a tweet sentiment classifier: `<sample_input>Great game tonight!</sample_input>` paired with `<sample_output>Positive</sample_output>`. Comments can flag edge cases too, like warning the model to watch for sarcasm, for example "oh yeah, I really needed a flight delay tonight, excellent!" reads positive on the surface but is negative. For harder cases, give an `<ideal_output>` and explain why it's the ideal one, not just what it is.

---

## implementing-multiple-tools.ipynb & sending-tool-results.ipynb & web-search-tool.ipynb

Claude only knows what it was trained on. Anything outside that, current data, real-time information, actions in the outside world, requires a tool.

**Setting up a tool.** Tool use starts with writing an actual function that performs the work, then writing a JSON schema that describes it. The schema is not tied to any specific model, it is a general data validation format. It needs a name, a description, and an input schema. The description matters more than it seems: a vague description gives Claude a vague sense of when to call the tool, so it needs to spell out what the tool does, when to use it, and what it returns.

**How the call works.** Once the schema is in place, calling Claude works through multi-block messages. Claude's response comes back as a content list that can include a text block and a tool use block. The tool use block carries the tool's id, name, and the input Claude wants to call it with. The full conversation history has to be passed back on the next request, and the next user message needs to include a tool result block that matches that tool use id, so Claude can connect the call to its result. Tool results get serialized as strings even when the underlying value is a number or boolean.

**Multiple tool calls.** Real conversations often need more than one tool call in a row, so the message handling has to support multiple tool use blocks in a single response, run each one, and return all the tool result blocks together.

**Server-side vs client-side tools.** Not all tools work the same way underneath. Web search is server-side: Claude runs the search itself and hands back a finished result, no implementation needed beyond passing the schema. Text editor is client-side, same as any custom tool: Claude only requests the operation, and the actual read and write logic has to be written and executed on your end.

## RAG

RAG breaks a large document into chunks first, things like strategy outlook, balance sheet, risk factors, so a huge block of text becomes smaller labeled pieces. A user's question gets matched to the most relevant chunk, which then goes into the prompt alongside the question. Asking about risks pulls the risk factors chunk, not the whole document.

Upsides: scales well, keeps the model focused, smaller prompts cost less, and it works across multiple documents. Downsides: needs a preprocessing and search step to find the right chunk, a chunk might miss relevant content outside it, and there's no single best way to chunk a document.

## 001_chunking.ipynb

RAG starts with breaking a source document into chunks, the hardest part of the whole pipeline since how a document gets divided shapes everything downstream. Three chunking strategies apply here.

**Size based** divides text into strings of equal length. Downsides: each chunk can get cut off mid-thought and loses surrounding context. The workaround is giving neighboring chunks some overlap, which creates a bit of duplication but keeps more context inside each piece.

**Structure based** divides text by its existing structure, headers, paragraphs, sections. Sounds clean in theory, but implementation gets shaky whenever the source text doesn't reliably follow that structure.

**Semantic based** groups related sentences or sections together. Requires understanding individual sentences, so it's more computationally expensive, but produces more relevant chunks.

Which method wins depends entirely on the source material. Chunking by character count ends up working for the vast majority of cases.

## 002_embeddings.ipynb

Semantic search finds chunks related to a user's question by using text embeddings, feeding each chunk into an embedding model that turns it into a list of numbers between -1 and 1. Each number scores some quality of the text, though which quality maps to which number isn't something you can pin down directly.

The full RAG flow: chunk the source text, generate embeddings for each chunk, normalize them (handled automatically, and visualizable on a unit circle), then store everything in a vector database, a database built specifically for storing and comparing long lists of numbers like embeddings.

A user's query gets embedded the same way, then the vector database returns whichever stored vector sits closest to it. Closeness is measured with cosine similarity, the cosine of the angle between the query vector and each stored embedding. A value close to -1 means unrelated. Distance is calculated as 1 minus similarity: same direction scores 0.0, perpendicular scores 1.0, completely opposite scores 2.0. Once the closest chunk is found, it gets added into the prompt alongside the user's question before that prompt goes to Claude.

## hybrid.ipynb

Semantic search (embeddings) is strong at context and meaning, lexical search is strong at exact word matches. Combining both gives a better balance than either alone.

Lexical search here uses BM25 (best match 25, the 25th variation of the original formula). It tokenizes the user's query, checks how often each term shows up across all documents, and weights rarer terms higher. The chunk that uses the most high-weighted terms wins.

Combining semantic and lexical search into one pipeline needs a retriever, a class wrapping both the vector index and the BM25 index. Results from each get merged using reciprocal rank fusion, RRF_score(d) = Σ 1/(k + rank_i(d)), where k is a constant, 60 is common but a smaller value like 1 was used here for clearer results. Higher RRF score means more relevant chunk.

## Features of Claude

**Extended thinking.** Gives Claude time to reason through a query before generating a final response, returned as a separate reasoning block. It makes the reasoning deeper and more accurate, but you're charged for the tokens generated during that thinking and it adds latency. More intelligence, more cost and wait. The way to decide is to run an eval on your prompt first, and if accuracy isn't good enough, consider turning it on.

Normally a user message comes back as a text block. With extended thinking enabled, the response contains a thinking block followed by the text block. The thinking block carries a **signature**, a cryptographic token confirming the thinking text was generated by Claude and not modified in any way. **Redacted thinking** happens when Claude generates thinking text that gets flagged by an internal safety system, so the content is hidden while Claude keeps its context about what it already worked through. `thinking_budget` sets the max tokens Claude may spend thinking, and `max_tokens` has to be greater than it.

**Citations.** Let Claude reference an outside source and state directly where the information came from. The response returns `cited_text` (the text being cited), `document_index` (which document, when multiple are provided), `document_title`, `start_page_number`, and `end_page_number`.

## 002_images.ipynb

Claude can answer questions about images, with limits: up to 100 images across all messages in a single request, 5MB max size, 8000px max height or width for a single image, dropping to 2000px when multiple images are sent. Images are billed as tokens and have to be base64 encoded. In code, the user message gets an image block added alongside the usual text block, and the image can be sent raw or by URL.

Accuracy depends heavily on prompting. Send a photo of 12 marbles with a simple prompt and Claude might answer 13. The fix is being concrete and giving guidelines, for example telling it to identify each marble one at a time, assign a number as it goes, start from the bottom-left corner and work row by row, then verify the count using a different method. Multishot examples work too, sending a picture with a known count of 11 first, then asking about the next one. Sending a PDF uses the same code path as sending an image.

## 003_caching.ipynb

**Prompt caching** speeds up responses and cuts cost. Before generating output, Claude does heavy internal work on the prompt, tokenizing, creating embeddings, building context from surrounding text, then discards all of it once the response is sent. On a follow-up request containing the original message plus the new one, that same work gets redone from scratch. Caching stores it in a temporary data store instead so follow-up requests can reuse it.

Nothing is cached automatically. You add a cache breakpoint manually, and cached content only gets used on follow-up requests. Tools and system prompts are the most common places to put breakpoints, though it depends on the application. Content has to be at least 1024 tokens to be cacheable, counted across all messages and blocks.

The cache is sensitive. Change a single character in a tool schema or system prompt and the cache for that part is fully invalidated, so the next request writes to cache from scratch (`cache_creation_input_tokens`) instead of reading from it (`cache_read_input_tokens`). Partial matches don't count, content is either byte-identical or it's a full miss for that block. Granularity works per block, so changing the system prompt while keeping tools identical still reads tools from cache and rewrites only the system prompt.

**Files API and code execution.** The Files API lets you upload a document (PDF, text, image) ahead of time and get back file metadata including a `file_id`, which later requests reference instead of resending the file. Code execution is a server-side tool needing only a predefined tool schema, running code in a Docker container with no network access. Getting data in and out of that container is what pairing it with the Files API solves. The result is a real computed answer, actual numbers and statistics from code that physically ran over the file's data, rather than a guess based on the filename or training patterns.

More notebooks coming, will keep updating this README as I go.
