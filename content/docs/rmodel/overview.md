+++
title = "Overview"
weight = 2
description = """
This page tells you how to get started with the Compose theme.
"""
+++

[RModel](https://github.com/rModel/rModel) is a workflow programming framework designed for constructing agentic applications with LLMs. It implements by the scheduling of computational units (`Neuron`), that may include loops, by constructing a `Brain` (a directed graph that can have cycles) or support the loop-less DAGs. A `Brain` consists of multiple `Neurons` connected by `Link`s. Inspiration was drawn from [LangGraph](https://github.com/langchain-ai/langgraph). The `Memory` of a `Brain` leverages [ristretto](https://github.com/dgraph-io/ristretto) for its implementation.

- Developers can build a `Brain` with any process flow:
    - Sequential: Execute `Neuron`s in order.
    - Parallel and Wait: Concurrent execution of `Neuron`s with support for downstream `Neuron`s to wait until all the specified upstream ones have completed before starting.
    - Branch: Execution flow only propagates to certain downstream branches.
    - Looping: Loops are essential for agent-like behaviors, where you would call an LLM in a loop to inquire about the next action to take.
    - With-End: Stops running under specific conditions, such as after obtaining the desired result.
    - Open-Ended: Continuously runs, for instance, in the scenario of a voice call, constantly listening to the user.
- Each `Neuron` is a concrete computational unit, and developers can customize `Neuron` to implement any processing procedure (`Processor`), including LLM calls, other multimodal model invocations, and control mechanisms like timeouts and retries.
- Developers can retrieve the results at any time, typically after the `Brain` has stopped running or when a certain `Memory` has reached an expected value.