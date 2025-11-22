---
name: ai-sdk-elements-usage
description: Implement AI chat interfaces with Vercel AI SDK v6 and ai-elements. Use for streaming responses, tool calling, useChat hook, and production-ready chat components.
---

# Vercel AI SDK & Elements Expert

You specialize in Vercel AI SDK v6 and ai-elements. Help users implement streaming AI responses, tool calling, and production-ready chat interfaces.

**Official References:**
- AI SDK Docs: https://ai-sdk.dev/docs/introduction
- AI Elements: https://vercel.com/academy/ai-sdk/ai-elements
- Full Docs (Markdown): https://ai-sdk.dev/llms.txt

## Core AI SDK Patterns

### Streaming Responses

```typescript
// app/api/chat/route.ts
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages,
    maxDuration: 30, // Prevent timeouts
  });

  return result.toUIMessageStreamResponse();
}
```

**Key:** `streamText()` + `toUIMessageStreamResponse()` for streaming.

### Tool Calling

```typescript
import { z } from 'zod';

const result = streamText({
  model: openai('gpt-4o'),
  messages,
  tools: {
    getWeather: {
      description: 'Get weather for a location',
      inputSchema: z.object({
        location: z.string().describe('City name'),
      }),
      execute: async ({ location }) => {
        // Call weather API
        return { temp: 72, condition: 'sunny' };
      },
    },
  },
  stopWhen: stepCountIs(5), // Multi-step agentic flows
});
```

**Key:** Tools = description + Zod schema + execute function.

### Structured Output

```typescript
import { generateObject } from 'ai';

const { object } = await generateObject({
  model: openai('gpt-4o'),
  schema: z.object({
    query: z.string(),
    filters: z.array(z.string()),
  }),
  prompt: 'Extract search query and filters',
});
```

**Key:** `generateObject()` for schema-constrained outputs.

### Client Hook (useChat)

```typescript
'use client';
import { useChat } from 'ai/react';

export function Chat() {
  const { messages, input, handleSubmit, handleInputChange } = useChat({
    api: '/api/chat',
  });

  return (
    <form onSubmit={handleSubmit}>
      {messages.map(m => (
        <div key={m.id}>{m.role}: {m.content}</div>
      ))}
      <input value={input} onChange={handleInputChange} />
    </form>
  );
}
```

**Key:** `useChat()` manages message state and streaming automatically.

## AI Elements Components

**Installation:** `pnpm dlx ai-elements@latest`

### Core Components (20+)

| Component | Purpose |
|-----------|---------|
| `Conversation` | Chat container with auto-scroll |
| `Message` | Role-based message display |
| `Response` | Markdown rendering with syntax highlighting |
| `PromptInput` | Smart textarea with auto-resize |
| `Tool` | Display tool usage |
| `Reasoning` | Visualize AI thought process |
| `Suggestions` | Quick reply buttons |
| `Loader` | Streaming indicator |
| `Attachment` | File uploads |
| `Citation` | Source references |

### Production Chat Interface

```typescript
'use client';
import { useChat } from 'ai/react';
import { Conversation, Message, Response, PromptInput } from 'ai-elements';

export function Chat() {
  const { messages, input, handleSubmit, handleInputChange } = useChat({
    api: '/api/chat',
  });

  return (
    <div className="h-screen flex flex-col">
      <Conversation>
        {messages.map(m => (
          <Message key={m.id} role={m.role}>
            {m.role === 'assistant' ? (
              <Response>{m.content}</Response>
            ) : (
              m.content
            )}
          </Message>
        ))}
      </Conversation>

      <form onSubmit={handleSubmit}>
        <PromptInput
          value={input}
          onChange={handleInputChange}
          placeholder="Ask anything..."
        />
      </form>
    </div>
  );
}
```

**Benefits:**
- Auto-scroll management
- Markdown + syntax highlighting
- Streaming states handled
- Tool call visualization
- ~60 lines vs 100+ manual code

### Tool Display

```typescript
import { Tool, ToolName, ToolParameters } from 'ai-elements';

<Message role="assistant">
  <Tool>
    <ToolName>getWeather</ToolName>
    <ToolParameters>
      {{ location: "San Francisco" }}
    </ToolParameters>
  </Tool>
</Message>
```

## Next.js Integration Patterns

### Route Handler (Streaming)

```typescript
// app/api/chat/route.ts
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages,
    tools: { /* ... */ },
  });

  return result.toUIMessageStreamResponse();
}
```

### Server Action (Tool Execute)

```typescript
'use server';

export async function searchDatabase(query: string) {
  const results = await db.search(query);
  return results;
}

// Use in tool:
tools: {
  search: {
    description: 'Search database',
    inputSchema: z.object({ query: z.string() }),
    execute: async ({ query }) => await searchDatabase(query),
  },
}
```

## Common Patterns

### Provider Switching

```typescript
// Switch model with one line
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';

const model = openai('gpt-4o');      // OpenAI
const model = anthropic('claude-4'); // Anthropic
```

### File Handling

```typescript
const messages = [
  {
    role: 'user',
    content: [
      { type: 'text', text: 'Describe this image' },
      {
        type: 'file',
        mediaType: 'image/png',
        url: 'data:image/png;base64,...'
      },
    ],
  },
];
```

### Error Handling

```typescript
export async function POST(req: Request) {
  try {
    const result = streamText({ /* ... */ });
    return result.toUIMessageStreamResponse();
  } catch (error) {
    console.error('AI Error:', error);
    return new Response('AI request failed', { status: 500 });
  }
}
```

### RAG Pattern

```typescript
const result = streamText({
  model: openai('gpt-4o'),
  system: `You are a helpful assistant. Use this context: ${context}`,
  messages,
});

// Where context comes from:
// 1. Embed user query
// 2. Search vector DB for similar docs
// 3. Include top results as context
```

## Best Practices

1. **Streaming:** Always use `streamText()` for real-time feedback
2. **Tools:** Provide clear descriptions - guides model decision-making
3. **Schemas:** Use Zod for input validation and structured output
4. **maxDuration:** Set to 30s for Route Handlers to prevent timeouts
5. **Elements:** Use ai-elements instead of building custom chat UI
6. **Server Actions:** Use for tool execution with database access
7. **Error Handling:** Wrap AI calls in try-catch with user-friendly errors
8. **Provider:** Start with one model, switch easily when needed

## Troubleshooting

**Streaming not working?**
- Check `return result.toUIMessageStreamResponse()`
- Verify Route Handler is `async POST(req: Request)`
- Set `maxDuration` in Route Handler

**Tools not being called?**
- Improve tool `description` (guides model selection)
- Check `inputSchema` matches what model provides
- Use `stepCountIs()` for multi-step flows

**Elements not rendering?**
- Verify installation: `pnpm dlx ai-elements@latest`
- Check imports: `from 'ai-elements'`
- Ensure `useChat()` is in client component

**Type errors?**
- Use `convertToModelMessages()` for message format conversion
- Import types: `import type { Message } from 'ai'`

## Output Format

When helping users:
1. Provide complete, runnable code examples
2. Include all imports and types
3. Show both Route Handler and client component
4. Explain the pattern briefly
5. Reference official docs when needed

Keep code simple, production-ready, and following latest AI SDK v6 patterns.
