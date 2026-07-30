# AI Capabilities and Limitations

## Regular AI vs Generative AI

Regular AI works with data that already exists. It runs operations on
what's already there, it doesn't go out and search the internet or
verify new claims, it just processes. Generative AI is different, it
produces new outputs that didn't exist before, built from patterns
learned during training rather than pulled from a fixed dataset.

## How generative AI gets built

Two stages. Pretraining is where the model learns from a massive
amount of data, this is where it picks up language, facts, patterns.
Fine tuning comes after, and this is where it gets shaped to actually
be safe and helpful rather than just a raw pattern matcher.

## Next token prediction and hallucinations

At its core the model is statistically predicting what text comes
next. Most of the time this works fine because it's grounded in
patterns it saw a lot during training. The problem shows up when a
topic is niche or narrow, the model has less to draw on so it drifts
into territory it's basically guessing at. That's where hallucination
comes from, it starts fabricating names, citations, authors that
sound plausible but don't exist.

The tell for this: if an answer gets suspiciously specific with names
and citations on a narrow topic, that's exactly when to double check
it rather than trust it at face value.

## The four Ds

A way to think about working with AI reliably. Delegation is
deciding what actually makes sense to hand off to the model versus
doing yourself. Description is being specific and clear about what
you want instead of vague. Discernment is being able to judge whether
what came back is actually good or just sounds good. Diligence is
following through and verifying instead of taking the first answer
and moving on.

## Embeddings and vector search

Embeddings turn words (or chunks of text) into vectors, numbers that
capture meaning. Vector search then takes those vectors and compares
them to find what's actually similar in meaning, not just similar in
spelling. This is the backbone of how retrieval works, which matters
a lot once RAG comes into play in stage 2.

## Lost in the middle

Information placed in the middle of a long context tends to get
underweighted. The model pays more attention to the beginning and
the end, and less to what's buried in the middle. This is sometimes
described as a U shaped curve, attention is high at the start, dips
in the middle, comes back up at the end.

How to actually work around this:
- avoid dragging out a conversation for too long
- put the most important material or instruction at the beginning
- treat middle content as the part most likely to get overlooked
- use context saving features instead of letting one thread run forever
- start a fresh conversation when the old one has drifted too far

## Real world failures

Most real failures aren't caused by just one of these limitations
on its own, they're usually two of these properties interacting at
once, for example a niche topic plus a long conversation where the
key detail got buried in the middle. Being able to spot which two
are combining is what actually lets you fix the problem instead of
guessing.

The general defense against all of this comes down to three things:
verification, managing context, and checkpoints along the way instead
of waiting until the end to check the work.

## Source:

https://anthropic.skilljar.com/ai-capabilities-and-limitations
