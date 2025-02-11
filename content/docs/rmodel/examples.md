+++
title = "Chat Agent Example"
weight = 5
description = """
This page tells you how to get started with the Compose theme.
"""
+++

## Chat Agent Example

### Brain (Process Control Center)
```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "os"
    
    "github.com/sashabaranov/go-openai"
    "github.com/sashabaranov/go-openai/jsonschema"
    "github.com/rModel/rModel"
)

func main() {
    // Initialize brain configuration
    bp := rModel.NewBrainPrint()
    
    // Register processing units
    bp.AddNeuron("llm", chatLLM)       // LLM conversation module
    bp.AddNeuron("action", callTools)  // Tool execution module

    // Configure processing flow
    _, _ = bp.AddEntryLink("llm")             // Entry point
    continueLink, _ := bp.AddLink("llm", "action")  // LLM → Tools
    _, _ = bp.AddLink("action", "llm")        // Tools → LLM
    endLink, _ := bp.AddEndLink("llm")        // Exit point

    // Configure decision routing
    _ = bp.AddLinkToCastGroup("llm", "continue", continueLink)
    _ = bp.AddLinkToCastGroup("llm", "end", endLink)
    _ = bp.BindCastGroupSelectFunc("llm", llmNext)

    // Start processing
    brain := bp.Build()
    _ = brain.EntryWithMemory(
        "messages", []openai.ChatCompletionMessage{
            {Role: openai.ChatMessageRoleUser, Content: "What's the weather in Boston today?"},
        })
    brain.Wait()

    // Output final state
    messages, _ := json.Marshal(brain.GetMemory("messages"))
    fmt.Printf("Message history: %s\n", messages)
}
```

### LLM Conversation Module
```go
var tools = []openai.Tool{
    {
        Type: openai.ToolTypeFunction,
        Function: &openai.FunctionDefinition{
            Name:        "get_current_weather",
            Description: "Retrieve current weather conditions",
            Parameters: jsonschema.Definition{
                Type: jsonschema.Object,
                Properties: map[string]jsonschema.Definition{
                    "location": {
                        Type:        jsonschema.String,
                        Description: "City and state, e.g. San Francisco, CA",
                    },
                    "unit": {Type: jsonschema.String, Enum: []string{"celsius", "fahrenheit"}},
                },
                Required: []string{"location"},
            },
        },
    },
}

func chatLLM(b rModel.BrainRuntime) error {
    messages := b.GetMemory("messages").([]openai.ChatCompletionMessage)
    
    client := openai.NewClient(os.Getenv("OPENAI_API_KEY"))
    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model:    openai.GPT3Dot5Turbo0125,
            Messages: messages,
            Tools:    tools,
        },
    )
    
    // Handle response
    if err == nil && len(resp.Choices) > 0 {
        msg := resp.Choices[0].Message
        messages = append(messages, msg)
        b.SetMemory("messages", messages)
    }
    return err
}
```

### Tool Execution Module
```go
func callTools(b rModel.BrainRuntime) error {
    messages := b.GetMemory("messages").([]openai.ChatCompletionMessage)
    lastMsg := messages[len(messages)-1]

    for _, call := range lastMsg.ToolCalls {
        switch call.Function.Name {
        case "get_current_weather":
            // Execute weather API call (mock implementation)
            fmt.Printf("Executing %s with params: %s\n", 
                call.Function.Name, call.Function.Arguments)
            
            // Simulate API response
            messages = append(messages, openai.ChatCompletionMessage{
                Role:       openai.ChatMessageRoleTool,
                Content:    "Sunny, 22°C",
                ToolCallID: call.ID,
                Name:       call.Function.Name,
            })
        }
    }
    b.SetMemory("messages", messages)
    return nil
}
```
### Routing Decision Logic
```go
func llmNext(b rModel.BrainRuntime) string {
    messages := b.GetMemory("messages").([]openai.ChatCompletionMessage)
    lastMsg := messages[len(messages)-1]
    
    if len(lastMsg.ToolCalls) > 0 {
        return "continue"  // Requires tool execution
    }
    return "end"           // Conversation complete
}
```

### Workflow Explanation
- Initialization: 
  - Configure processing nodes and connections
- Message Handling:
  - User input → LLM processing
  - LLM response analysis for tool requirements
- Tool Execution:
  - Parse function call parameters
  - Simulate/Make API calls
  - Store results back to memory
- Response Generation:
  - Feed tool results back to LLM
  - Generate final user response
- Termination
  - Exit when conversation completes