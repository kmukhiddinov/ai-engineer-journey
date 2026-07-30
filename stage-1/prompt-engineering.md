# Prompt Engineering (docs)

Went through Prompting best practices for Claude. A good chunk of this
overlapped with stuff that already came up in Claude 101 and AI
Capabilities and Limitations, being specific and giving context wasn't
new information at this point. But there were a handful of techniques
in here I hadn't actually seen laid out properly before.

The employee framing stuck with me the most. Think of Claude as a new
employee who's very capable but has zero context on your company,
your norms, your workflows. Being clear and direct isn't about dumbing
things down, it's about not assuming shared context that was never
actually given.

Related to that: it's not enough to just give an instruction, you get
better results explaining the reasoning or motivation behind it too.
Telling Claude why a behavior matters changes how it handles edge
cases it wasn't explicitly told about.

XML tags came up as the backbone of how to structure a complex prompt.
Wrapping different parts in tags like instructions and context lets
Claude parse what's what instead of guessing where one section ends
and another begins. Examples specifically benefit from this too,
wrapping a single example in an example tag, or multiple ones inside
an examples tag, actually improves steerability compared to just
listing them as plain text.

Quotes work the same way. Pulling relevant quotes into their own tag
and asking Claude to extract first, then reason based on what got
extracted, gives a cleaner result than asking it to reason over a raw
wall of text directly.

Framing instructions positively matters more than I expected, telling
Claude what to do lands better than telling it what not to do.

Another one that's easy to overlook is matching the format of your
input to the format you want back. If you want structured output,
your prompt should already look structured going in.

Last thing, for anything involving steps, like connecting tools or a
multi-step process, it's worth explicitly asking for documentation of
each step along the way if those steps actually matter for the task.

Didn't go through the whole sidebar. Tool use and agentic systems are
sitting there for stage 2, no point reading them without anything to
apply them to yet. Model-specific pages outside Sonnet 5, LaTeX
output, document creation, and migration considerations got skipped
entirely, none of that applies to what I'm building right now.

## Source:

https://platform.claude.com/docs/en/about-claude/use-case-guides/overview
