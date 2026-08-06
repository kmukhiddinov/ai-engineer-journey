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

More notebooks coming, will keep updating this README as I go.
