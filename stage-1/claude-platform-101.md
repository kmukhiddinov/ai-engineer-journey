# Claude Platform 101

The platform breaks down into three layers: primitives, infrastructure, and controls. Primitives are the raw building blocks like the Messages API and Files API. Infrastructure is what runs things at scale, agents, retries, queues. Controls are how you monitor and manage all of it, dashboard, evals, workspaces. The framing that ties it together is build with primitives, scale on infrastructure, run with controls.

**Example:** you build a support bot using the Messages API directly (primitive), then move it onto managed agent infrastructure so it can run retries and handle load on its own (infrastructure), then watch its performance and error rate from the dashboard (controls).

## Model Choice

Model choice comes down to a speed/cost/intelligence tradeoff. Haiku is built for volume, less intelligent but fast and cheap. Sonnet is the daily driver, capable and fast. Opus is for heavy work, most intelligent but slower and expensive.

**Example:** a chatbot answering thousands of simple FAQ messages a day runs on Haiku. The same product's occasional "analyze this 40-page contract" feature runs on Opus instead.

## Agent Loop

The agent loop is observe, decide, act, repeat. Claude requests something, your code executes it, the result goes back to Claude, and that loop continues until the task is done.

**Example:** Claude decides it needs today's weather, requests a `get_weather` tool call, your code runs it and returns "72°F, sunny," Claude reads that result and decides its next step from there.

## Tools

Every tool has three parts, a name, a description, and an input schema. The description is what Claude compares against the user's request to decide whether and how to use the tool, so vague wording there directly produces bad tool calls.

**Example:** a description like "gets data" is too vague. "Looks up the current shipping status for an order given its order ID" gives Claude enough to reliably match it to the right user request.

Tool runner automates the tool-calling mechanism so you're not manually wiring up every request/response cycle by hand.

## Extended Thinking

Extended thinking lets Claude reason step by step before producing a final answer, chain of thought first, then the response. It's suited to math, multi-step logic, debugging, regulatory analysis, complex comparisons, or anywhere there are real tradeoffs to weigh.

**Example:** asking "should we use Postgres or MongoDB for this project" benefits from extended thinking because there are real tradeoffs to weigh, versus "what's the syntax for a Python for loop" which doesn't need any of that.

## Built-in vs Client Tools

Built-in tools are ones Claude calls on its own behalf, server-side, things like web search, file handling, or code execution. Client tools are ones you run yourself on your side, like memory or bash. Both are described as the fastest path to adding real functionality without building everything from scratch.

## Skills

Skills are a packaged set of instructions. You upload docs, name the skill, and attach it to any message, the instructions get set up once and then get called whenever needed.

**Example:** a "brand voice" skill with a document describing tone, banned words, and formatting rules, attached whenever you want an output written by Claude to actually sound like your company.

Tools and skills solve different problems. Tools connect to data, take actions, or run code, things Claude literally cannot do on its own. Skills teach a procedure, plain instructions, no code execution involved.

**Example:** a tool checks live inventory in a database. A skill teaches Claude how your company formats a return-approval email, no database involved, just instructions.

## MCP

MCP connects Claude to third-party services. Regular tools connect to your own internal systems, and you own the code and the maintenance for those yourself.

## Context Management

Context is everything Claude sees on a given turn, every input and output combined. Managing it well means loading only what's actually needed right now instead of dumping everything in, this is called just-in-time context.

**Example:** instead of pasting an entire 200-page document into every message, you retrieve and pass in only the three relevant paragraphs for the current question.

Compaction kicks in when a conversation gets too long, older turns get automatically compressed into a shorter summary block instead of carrying the full history forward. Prompt caching lets you tell the API to remember a piece of content so it doesn't get reprocessed from scratch on every single call.

**Example:** a long system prompt with company policies gets cached once, so the next hundred requests don't pay the cost of reprocessing that same text every time.

## Memory

The memory tool lets Claude remember things across sessions. It writes to a memory store after using a tool call to do so, and at the start of every chat there's a standing instruction to check memory before replying.

## Loop vs Managed Agents

Loop agents run on your own infrastructure, you're responsible for the cycle yourself. Managed agents run on Anthropic's infrastructure instead, you hand off the loop and just tell it what to do.

## Source

https://anthropic.skilljar.com/claude-platform-101
