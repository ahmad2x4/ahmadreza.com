---
layout: post
title: Building a Model Context Protocol Server for SEQ Structured Logging
published: true
author: ahmadreza
comments: true
categories: [blog]
tags: [LLM, SEQ, Logging, Model Context Protocol, TypeScript]
shareimg: ../img/seq-mcp-hero.svg
excerpt_separator: <!--more-->
---

Large Language Models (LLMs) have become increasingly powerful, but their true potential emerges when they can interact with external tools and systems. As Mark Russinovich aptly described, giving LLMs the ability to use tools is like providing them with "arms and legs." In this post, I'll demonstrate how to create a Model Context Protocol (MCP) server that enables LLMs to interact with SEQ, a powerful structured logging and observability platform. <!--more-->

## Introduction

The inspiration for this project came from reading my former colleague Stafford Williams' blog post about [LLM Agent Assisted Coding](https://staffordwilliams.com/blog/2025/01/12/llm-agent-assisted-coding/). The concept of Model Context Protocol caught my attention as it provides a standardized way for LLMs to interact with external tools while maintaining a clear separation of concerns.

[SEQ](https://datalust.co/seq), developed by Datalust, is a structured logging platform that captures events as fully-structured JSON data. Its powerful query language makes it natural to search and analyze log data without awkward parsing or format handling. I realized that combining SEQ's capabilities with LLMs through an MCP server could create a powerful tool for log analysis and system monitoring.

## What we'll build

In this post, I'll walk you through creating an MCP server that:
1. Connects to a SEQ server and authenticates using an API key
2. Exposes SEQ's signal definitions for organizing and filtering logs
3. Provides access to log events with flexible querying capabilities
4. Enables monitoring of alert states

The end result will be a tool that allows LLMs to:
- Browse and understand available logging signals
- Query and analyze log events with natural language
- Monitor system health through alert states
- Provide intelligent insights about system behavior

## Understanding MCP and SEQ

Model Context Protocol (MCP) serves as a bridge between Large Language Models and external tools. Think of it as a standardized way for AI models to reach out and interact with real-world systems, much like how APIs allow different software systems to communicate. The protocol defines how tools can expose their capabilities to LLMs in a structured way that the models can understand and use effectively.

SEQ, on the other hand, represents a modern approach to application logging. Instead of dealing with plain text log files, SEQ treats each log entry as a structured event with rich metadata. When your application logs a message like "User completed checkout", you can provide additional contextual data such as the user ID, Shopping Basket Id, item count, order amount. SEQ also adds more metadata like timestamp, and any other relevant information - all in a structured format that's easy to query and analyze.

### How They Work Together

The magic happens when we connect these two systems through an MCP server. Interestingly, I used Claude.ai itself to help build this server. While I provided the endpoint specifications and guided the development process, Claude helped write much of the implementation code, demonstrating the practical value of LLM-assisted development.

Our server translates SEQ's powerful querying capabilities into a format that LLMs can understand and use. Let's look at the three main capabilities we're exposing:

1. **Signals**: In SEQ, signals are saved searches that help identify important patterns in your logs. Think of them as pre-defined filters for common scenarios like "Error Events" or "Failed Login Attempts". Here's how we expose them through our MCP server:

```typescript
server.tool(
  "get-signals",
  {
    ownerId: z.string().optional(),
    shared: z.boolean().optional(),
    partial: z.boolean().optional()
  },
  async ({ ownerId, shared, partial }) => {
    const signals = await makeSeqRequest<Signal[]>('/api/signals', {
      shared: shared?.toString() ?? "true"
    });
    // More code ...
    return {
      content: [{
        type: "text",
        text: JSON.stringify(signals, null, 2)
      }]
    };
  }
);
```

2. **Events**: The heart of SEQ is its event storage and querying system. Our MCP server makes this accessible to LLMs with flexible filtering options:

```typescript
server.tool(
  "get-events",
  {
    signal: z.string().optional()
      .describe('Comma-separated list of signal IDs'),
    filter: z.string().optional()
      .describe('Filter expression for events'),
    count: z.number().min(1).max(MAX_EVENTS).optional()
      .default(MAX_EVENTS),
    range: timeRangeSchema.optional()
      .describe('Time range (e.g., 1m, 15m, 1h, 1d, 7d)')
  },
  async (params) => {
    // Implementation details
  }
);
```

3. **Alert States**: SEQ can monitor your logs and raise alerts when certain conditions are met. Our MCP server allows LLMs to check the current state of these alerts:

```typescript
server.tool(
  "get-alertstate",
  {},
  async () => {
    const alertState = await makeSeqRequest<any>('/api/alertstate');
    return {
      content: [{
        type: "text",
        text: JSON.stringify(alertState, null, 2)
      }]
    };
  }
);
```

## Building the SEQ MCP Server

Let's walk through creating an MCP server that seamlessly connects with SEQ. Our server will need to handle authentication, manage API requests, and translate between SEQ's API responses and the MCP format that LLMs can understand.

### Setting Up the Foundation

First, we need to establish the core infrastructure of our server. We'll use TypeScript for type safety and the MCP SDK for protocol handling. Our server needs to manage environment variables for configuration and set up the base URL and API key for SEQ:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// Configuration and constants
const SEQ_BASE_URL = process.env.SEQ_BASE_URL || 'http://localhost:8080';
const SEQ_API_KEY = process.env.SEQ_API_KEY || '';
const MAX_EVENTS = 100;
```

### Create the API Client

Our API client provides a clean interface for interacting with SEQ's endpoints:

```typescript
async function makeSeqRequest<T>(endpoint: string, params: Record<string, string> = {}): Promise<T> {
  const url = new URL(`${SEQ_BASE_URL}${endpoint}`);
  url.searchParams.append('apiKey', SEQ_API_KEY);
  
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      url.searchParams.append(key, value);
    }
  });

  const response = await fetch(url.toString(), { 
    headers: {
      'Accept': 'application/json',
      'X-Seq-ApiKey': SEQ_API_KEY
    }
  });

  if (!response.ok) {
    throw new Error(`SEQ API error: ${response.statusText} (${response.status})`);
  }

  return response.json();
}
```

### Starting the Server

The final piece is setting up the server to run using the MCP's standard I/O transport:

```typescript
if (import.meta.url === `file://${process.argv[1]}`) {
  const transport = new StdioServerTransport();
  server.connect(transport).catch(error => {
    console.error('Failed to start server:', error);
    process.exit(1);
  });
}
```

### Building and Configuring the Server

To get the server running with Claude for Desktop, follow these steps:

1. Build the server:
```bash
npm build
```
This will create the JavaScript files in `build/seq-server.js`.

2. Create an API key in SEQ:
   - Navigate to Settings in your SEQ installation
   - Go to API Keys and create a new key with appropriate permissions
   ![SEQ API Key](../img/seq-api-key-page.png)

3. Configure the server in Claude for Desktop following the [installation guide](https://github.com/ahmad2x4/mcp-server-seq/blob/master/README.md#installation):

```json
{
  "mcpServers": {
    "seq": {
      "command": "/path/to/seq-server/build/seq-server.js",
      "env": {
        "SEQ_BASE_URL": "your-seq-url",
        "SEQ_API_KEY": "your-api-key"
      }
    }
  }
}
```

For detailed setup instructions, refer to the [Testing your server with Claude for Desktop](https://modelcontextprotocol.io/quickstart/server) guide.

## Practical Usage and Applications

Before we dive into using our SEQ MCP server, let's make sure Claude for Desktop is picking up the tools we've exposed. You can do this by looking for the hammer icon:

![Claude tools](../img/claude.ai-tools-01.png)

When you click on that button you'll see the list of installed tools. If you have successfully installed this MCP server you should see:

![Claude tools](../img/claude.ai-tools-02.png)

When we connect our SEQ MCP server to an LLM, we create a powerful tool for log analysis and system monitoring. Let's explore how this integration works in practice and see how it can help teams better understand their systems.

### Setting Up the LLM Assistant

First, let's provide the LLM with context about our logging system. Here's an effective prompt that helps the LLM understand its capabilities:

```text
You are a helpful system operations officer with access to SEQ, our log and monitoring system. You can:
- Query events using the get-events tool with filters and time ranges
- Check system signals using get-signals
- Monitor alert states using get-alertstate

Please familiarize yourself with our signals first to understand how our logs are organized.
```

This prompt establishes the LLM's role and outlines its available tools. The instruction to check signals first is crucial - it helps the LLM understand how our logging data is structured before attempting any analysis.

### Real-World Example: System Health Check

Let's see how this works in practice. When we ask the LLM to summarize today's system activity, it first checks the available signals and then analyzes the logs:

```text
I'll help you analyze the logs from today's activity.

First, let me check our available signals:

[LLM uses get-signals to retrieve signal list]

I see several important signals:
1. Logs (signal-m20231211) - General logs without spans
2. Spans (signal-m20231011) - Logs with start time and span ID for tracing
3. Errors (signal-m33301) - Critical, fatal, and error level logs
4. Warnings (signal-m33302) - Warning level logs
5. Exceptions (signal-m33303) - Logs containing exception information

Let me analyze today's events:

[LLM uses get-events with appropriate signals]

Here's what I found in today's activity:

1. Order Processing Performance:
   - Average response time: 20ms for order creation
   - Successful order updates with status transitions
   - No failed transactions detected

2. System Health:
   - All API endpoints responding normally
   - Database operations executing within expected timeframes
   - No active alerts or warnings

3. Notable Activities:
   - Peak order processing between 2-3 PM
   - Several successful customer order completions
   - Normal background task execution
```

The LLM first builds context by understanding available signals, then uses this knowledge to query relevant events. It can correlate information across different signals to provide a comprehensive view of system health.

### Investigating Issues

The real power of this integration shows when investigating system issues. Let's say we notice some customer complaints about order processing:

```text
Human: Can you check if there were any issues with order processing in the last hour?

LLM: I'll investigate the order processing system's logs from the last hour.

[LLM uses get-events with specific parameters]
{
    signal: "signal-m33301,signal-m33302",  // Checking both errors and warnings
    range: "1h",
    filter: "OrderProcessing" 
}

Based on my analysis:

1. Performance Impact:
   - Two brief spikes in response time at 14:15 and 14:45
   - Average order processing time increased from 20ms to 45ms

2. Identified Issues:
   - Database connection pool reached 80% capacity during peak times
   - Payment gateway showed intermittent latency
   - No complete failures, but some operations took longer than usual

3. Current Status:
   - System has recovered to normal processing times
   - All order completions successful despite delays
   - No active alerts related to order processing
```

The LLM combines information from multiple signals and time periods to provide context-aware analysis. It not only identifies issues but also correlates them with system metrics to provide a complete picture.

### The Advantage of Structured Logging with LLMs

What makes this integration particularly powerful is how it combines SEQ's structured logging with the LLM's natural language understanding. The LLM isn't just searching through text logs - it's analyzing structured data with rich context. This enables:

1. Intelligent Correlation: The LLM can connect related events across different parts of the system, even when they're not explicitly linked. For instance, it might notice that increased database latency correlates with specific customer actions, helping identify the root cause of performance issues.

2. Pattern Recognition: By analyzing structured data over time, the LLM can identify patterns that might not be immediately obvious to human operators. This could include detecting gradual degradation in response times or subtle increases in error rates before they become critical.

3. Natural Language Interaction: Team members can investigate issues using natural language questions, making the logs more accessible to everyone, not just those familiar with SEQ's query syntax. This democratizes access to system insights and speeds up troubleshooting processes.

## What's Next: Creating an NPM Package

While the current implementation works well as a standalone application, the next step is to make this tool more accessible to the developer community. I plan to package this as an npm module, making it easier for developers to integrate SEQ's capabilities with their LLM applications with minimal setup. This will involve proper packaging, documentation, and examples to help others get started quickly with their own SEQ MCP server implementations.

## Conclusion 

The integration of SEQ with Large Language Models through Model Context Protocol opens up new possibilities for log analysis and system monitoring. By bridging the gap between structured logging and natural language understanding, we've created a system that makes log analysis more intuitive and accessible.

Throughout this journey of building a SEQ MCP server, we've seen how the structured nature of SEQ's logging perfectly complements the analytical capabilities of LLMs. The system we've built doesn't just pass along log messages - it enables LLMs to understand the context, correlate events, and provide meaningful insights about system behavior.

The real power of this integration lies in how it transforms complex log analysis into natural conversations. Team members can now investigate system behavior by asking questions in plain language, while the LLM leverages SEQ's powerful querying capabilities to provide comprehensive answers.

As we continue to explore the intersection of structured logging and artificial intelligence, tools like this MCP server will become increasingly valuable for understanding and maintaining our systems. Whether you're investigating issues, monitoring system health, or looking for performance insights, the combination of SEQ's structured data and LLM's analytical capabilities provides a powerful platform for modern system observability.

You can find the complete source code for this project in my GitHub repository: [https://github.com/ahmad2x4/mcp-server-seq](https://github.com/ahmad2x4/mcp-server-seq)