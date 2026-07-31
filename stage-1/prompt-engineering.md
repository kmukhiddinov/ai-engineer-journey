# Prompt Engineering (docs)

Went through Prompting best practices for Claude. A good chunk of this
overlapped with stuff that already came up in Claude 101 and AI
Capabilities and Limitations, being specific and giving context wasn't
new information at this point. But there were a handful of techniques
in here I hadn't actually seen laid out properly before.

Claude works best when you treat it like a new employee with zero
context on your company, your norms, your workflows. Being clear and
direct means spelling out things you'd normally assume are obvious.

Example: instead of "write a project update," give it "write a weekly
update for the engineering team, keep it under 150 words, lead with
blockers first, no marketing language."

It's not enough to just give an instruction, explaining the reasoning
or motivation behind it gets better results. Telling Claude why a
behavior matters changes how it handles edge cases it wasn't
explicitly told about.

Example: "Don't reveal internal pricing numbers" gets followed
literally. "Don't reveal internal pricing numbers because this chat
log may get shared with external partners" gets followed even when
the situation isn't exactly pricing, because Claude understands why
the rule exists.

XML tags are the backbone of how to structure a complex prompt.
Wrapping different parts in tags like instructions and context lets
Claude parse what's what instead of guessing where one section ends
and another begins. Examples benefit from this too, wrapping a single
example in an example tag, or multiple ones inside an examples tag,
improves steerability compared to listing them as plain text.

Example:
```
<instructions>Summarize the ticket in one sentence.</instructions>
<context>This is a support ticket from a subscription service.</context>
<examples>
<example>Input: "cancelled last week but still got charged" → Output: "Customer was charged after cancellation."</example>
</examples>
```

Quotes work the same way. Pulling relevant quotes into their own tag
and asking Claude to extract first, then reason based on what got
extracted, gives a cleaner result than reasoning over a raw wall of
text directly.

Example: dump a long contract into `<document>` tags, then ask
"first extract the quotes relevant to termination clauses into
`<quotes>`, then explain what they mean" instead of "summarize the
termination clauses" cold.

Framing instructions positively matters more than I expected, telling
Claude what to do lands better than telling it what not to do.

Example: "Answer only using the provided context" works better than
"Don't make anything up."

Matching the format of your input to the format you want back is easy
to overlook. If you want structured output, the prompt should already
look structured going in.

Example: if you want a JSON list of action items back, format your
notes as a rough list going in instead of one paragraph of text.

For anything involving steps, like connecting tools or a multi-step
process, it's worth explicitly asking for documentation of each step
along the way if those steps actually matter for the task.

Example: "As you call each tool, log what you called and why before
moving to the next step" instead of just asking for the final answer.

Didn't go through the whole sidebar. Tool use and agentic systems are
sitting there for stage 2, no point reading them without anything to
apply them to yet. Model-specific pages outside Sonnet 5, LaTeX
output, document creation, and migration considerations got skipped
entirely, none of that applies to what I'm building right now.

## Source:

https://platform.claude.com/docs/en/about-claude/use-case-guides/overview
