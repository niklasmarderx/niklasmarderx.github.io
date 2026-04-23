---
title: "Building Ember: An AI Agent Framework in Rust"
description: "Why I chose Rust to build an AI agent framework, what I learned about performance and safety in agentic systems, and where Ember is heading next."
pubDate: 2026-04-15
tags: ["rust", "ai", "agents"]
---

When I started building [Ember](https://github.com/niklasmarderx/Ember), the immediate question was: why Rust? Most AI tooling is written in Python. The ecosystem is there, the libraries are mature, and the LLM providers all have Python SDKs. So why write a new agent framework in a systems language?

The short answer: because I think the next generation of AI infrastructure needs to be fast, safe, and embeddable — and Python doesn't fit all three.

## What is an agent framework, really?

An "AI agent" is roughly: an LLM that can take actions, observe results, and decide what to do next. In practice, this means orchestrating a loop:

1. Build a prompt from context + tools
2. Call the LLM API
3. Parse the response — did it call a tool?
4. Execute the tool, capture output
5. Feed output back, repeat

This loop sounds simple. But when you're running multiple agents concurrently, streaming responses, managing tool call lifetimes, and dealing with retries and timeouts — the complexity compounds fast.

## Why Rust?

**Concurrency without the footguns.** Tokio's async runtime lets Ember run many agents in parallel with predictable scheduling and minimal overhead. In Python, you'd be managing GIL contention or spinning up processes. In Rust, you just `tokio::spawn`.

**Memory predictability.** Agent loops can run for minutes or hours. Python's garbage collector is fine for short-lived scripts but GC pauses and memory fragmentation are real concerns in long-running processes. Rust gives you deterministic memory behavior.

**Embeddable as a library.** Because Rust compiles to native code with a small binary footprint, Ember can be embedded in other tools — a CLI, a VS Code extension, an IDE plugin — without dragging in a Python runtime.

**Type safety catches protocol errors early.** LLM APIs have complex response formats: tool calls, streaming deltas, finish reasons. Modeling these as Rust enums and structs means the compiler catches mismatches before they surface as runtime panics at 2am.

## The tradeoffs

It's not all upside. The Rust ecosystem for AI tooling is still young. There's no equivalent of LangChain's integrations or LlamaIndex's connectors. I've had to write a lot of the plumbing myself — HTTP clients, JSON parsing for tool call formats, streaming response handling.

The compile times are also real. During early development when the API was changing constantly, waiting 30 seconds for a build after every structural change was painful. I've gotten better at organizing the codebase to minimize recompilation, but it's a real workflow cost.

## What's next for Ember

The immediate priorities are:

- **Better tool ergonomics**: right now defining tools requires too much boilerplate. I want a derive macro that generates tool schemas from Rust structs.
- **Persistent memory**: a simple key-value store that agents can write to and read from across invocations.
- **Streaming first**: full support for streaming responses with backpressure — so you can pipe agent output to a terminal or UI as it arrives.

If you're building something with AI agents and want to try Ember, the repo is at [github.com/niklasmarderx/Ember](https://github.com/niklasmarderx/Ember). Issues and PRs welcome.
