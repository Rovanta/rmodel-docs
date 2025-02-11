+++
title = "Quick Start"
weight = 4
description = """
This page tells you how to get started with the Compose theme.
"""
+++

Let's use `rModel` to build a `Brain` as shown below.

<img src="https://github.com/Rovanta/rmodel/blob/main/examples/chat_agent/chat_agent_with_function_calling/chat-agent-with-tools.png?raw=true" width="476" height="238">


### Defining a Brainprint

Define the graph's topology by outlining a brainprint (a shorthand for brain blueprint).

#### 1. Create a brainprint

```go
bp := rModel.NewBrainPrint()
```

#### 2. Add `Neuron`s

Bind a processing function to a neuron or custom `Processor`. In this example, a function is bound, and its definition is omitted for brevity. For more details, see [examples/chat_agent_with_function_calling](https://github.com/rModel/rModel/blob/main/examples/chat_agent/chat_agent_with_function_calling).

```go
// add neuron with function
bp.AddNeuron("llm", chatLLM)
bp.AddNeuron("action", callTools)
```

#### 3. Add `Link`s

There are three types of `Link`s:

- Normal Links: Include the `source Neuron` and `destination Neuron`
- Entry Links: Only have the `destination Neuron`
- End Links: The `Brain` will automatically go into a `Sleeping` state when there are no active `Neuron`s and `Link`s, but you can also explicitly define end links to set an endpoint for the `Brain` to run. You only need to specify the `source Neuron`, the `destination Neuron` will be END

```go
/* This example omits error handling */
// add entry link
_, _ = bp.AddEntryLink("llm")

// add link
continueLink, _ := bp.AddLink("llm", "action")
_, _ = bp.AddLink("action", "llm")

// add end link
endLink, _ := bp.AddEndLink("llm")
```

#### 4. Set cast select at a branch

By default, all outbound links of a `Neuron` will propagate (belonging to the default casting group). To set up branch selections where you only want certain links to propagate, define casting groups (CastGroup) along with a casting selection function (CastGroupSelectFunc). Each cast group contains a set of links, and the return string of the cast group selection function determines which cast group to propagate to.

```go
// add link to cast group of a neuron
_ = bp.AddLinkToCastGroup("llm", "continue", continueLink)
_ = bp.AddLinkToCastGroup("llm", "end", endLink)
// bind cast group select function for neuron
_ = bp.BindCastGroupSelectFunc("llm", llmNext)
```

```go
func llmNext(b rModel.BrainRuntime) string {
    if !b.ExistMemory("messages") {
        return "end"
    }
    messages, _ := b.GetMemory("messages").([]openai.ChatCompletionMessage)
    lastMsg := messages[len(messages)-1]
    if len(lastMsg.ToolCalls) == 0 { // no need to call any tools
        return "end"
    }
    
    return "continue"
}
```

### Building a `Brain` from a Brainprint

Build with various withOpts parameters, although it can be done without configuring any, similar to the example below, using default construction parameters.

```go
brain := bp.Build()
```

### Running the `Brain`

As long as any `Link` or `Neuron` of `Brain` is activated, it is considered to be running.
The `Brain` can only be triggered to run through `Link`s. You can set initial brain memory `Memory` before the `Brain` runs to store some initial context, but this is an optional step. The following methods are used to trigger `Link`s:

- Use `brain.Entry()` to trigger all entry links.
- Use `brain.EntryWithMemory()` to set initial `Memory` and trigger all entry links.
- Use `brain.TrigLinks()` to trigger specific `Links`.
- You can also use `brain.SetMemory()` + `brain.TrigLinks()` to set initial `Memory` and trigger specific `Links`.

⚠️Note: Once a `Link` is triggered, the program is non-block; the operation of the `Brain` is asynchronous.

```go
// import "github.com/sashabaranov/go-openai" // just for message struct

// set memory and trigger all entry links
_ = brain.EntryWithMemory("messages", []openai.ChatCompletionMessage{{Role: openai.ChatMessageRoleUser, Content: "What is the weather in Boston today?"}})
```

### Retrieving Results from `Memory`

`Brain` operations are asynchronous and unlimited in terms of timing for fetching results. We typically call `Wait()` to wait for `Brain` to enter `Sleeping` state or for a certain `Memory` to reach the expected value before retrieving results. Results are obtained from `Memory`.

```go
// block process until the brain is sleeping
brain.Wait()

messages, _ := json.Marshal(brain.GetMemory("messages"))
fmt.Printf("messages: %s\n", messages)
```