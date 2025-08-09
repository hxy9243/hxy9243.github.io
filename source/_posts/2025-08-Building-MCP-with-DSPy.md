title: Building MCP Server with DSPy
date: 2025-08-09 13:13:04
comments: true
categories: 'Programming'
tags: ['Learning', 'AI']
---

Recently I've been looking into MCP and how it integrates with development framework.
I tried with some hands-on tinkering, and learned quite a bit. Here's my learning and
some reflections.

Code here: https://github.com/hxy9243/agents/tree/main/librarian


# What's MCP

## Bird's Eye View

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/docs/getting-started/intro)
is a standard way for AI agents to securely connect to your services.
It provides a uniform interface for tools, resources, and prompts, enabling seamless
integration between AI systems and external services.

Clients like VSCode, Gemini Code, or Claude Code can now integrate with different tools and services
and become incredibly powerful, from reading PDFs, search for websites,
read your Google Drive files, or book plane tickets for you. You could implement the
MCP service once and easily install in all kinds of the AI assistants of your
choice.

With this standardization, you can implement your MCP server and they can directly
integrate with your user's agents, assistants, or more complicated workflows.
They can call and leverage your tools with a standardized interface.

Think of an AppStore moment for app discovery and AI integration.

<!--more-->


# Building with FastMCP

## FastMCP

**FastMCP** is a Python library that makes it easy to build MCP servers.
It handles the protocol implementation, allowing you to focus on defining your
tools and business logic.

The first thing that comes to mind is the School Library management tool, the one
we did when we were in college learning about relational databases.

I quickly drafted a Library implementation in Python:

## Define Basic Library Class

Create a basic Python class for managing library:

```python
class Library:
    def add_book(self): ...

    def add_member(self): ...

    def borrow_book(self): ...
    ...
```

I used an in-memory database with Python dictionaries, since it's for demo
purpose only.

## Define MCP Tools

Tools in MCP are functions that agents can call to perform specific actions.
FastMCP uses decorators to expose functions as MCP tools, the same way to use FastAPI or Flask:

```python
@mcp.tool()
async def search(query: str) -> List[Book]:
    """Searches for books by title or author."""
    return library.search_books(query)
```

Each tool includes:

- Type annotations for query parameters.
- Definition and type annotations for return values.
- Descriptive docstrings for agent understanding. This will be part of list tool and can instruct
  your LLM behavior as well.


## Define Transport

MCP supports multiple transport mechanisms:

- **StreamableHTTP**: Uses HTTP with streaming support,
  suitable for API-based integrations. Supports authentication headers and custom middleware.
  provides real-time streaming with built-in reconnection handling.
  Underlying, it uses SSE streaming for fast and efficient communication.

  This is ideal when you're building and exposing a service.

- **Studio Integration**: Client reads MCP server stdio directly. This is most common
  for local setups, where MCP server and client are both deployed locally, and
  there's less communication overheads.

## Client Integration with DSPy

[DSPy](https://dspy.ai/learn/) is a powerful LLM agentic framework that helps you
build your agent by focusing on the signature instead of the prompt tweaking.
It can handle the prompt generation for you and can even optimize the prompts for you.
For this use case we'll use its powerful `Signature` feature to quickly create a chatbot.

Also, DSPy has already implemented MCP support: (https://dspy.ai/tutorials/mcp/),
which enables you to conveniently implement your agent with MCP integration.

In this case, I can quickly implement something to handle user requests:

The `LibrarianAgent` class integrates MCP with DSPy's reasoning framework:


```python
class LibrarianSignature(dspy.Signature):
    """You are a librarian. You are given a list of tools to handle user requests."""
    user_request: str = dspy.InputField()
    process_result: str = dspy.OutputField()
```

And the agent can integrate with MCP tools by converting them into DSPy tools:

```python
async with Client("http://localhost:5400/mcp") as client:
    tools = await client.session.list_tools()
    dspy_tools = []
    for tool in tools.tools:
        dspy_tools.append(dspy.Tool.from_mcp_tool(client.session, tool))

    lm = dspy.ReAct(
        signature=LibrarianSignature,
        tools=dspy_tools,
    )
```

> Notice a small detail that both `FastMCP.client.Client` and `mcp.ClientSession`
> provides `list_tool()` function call.
>
> But DSPy integrates with `mcp.ClientSession`,
> `mcp.Tool` and `mcp.CallToolResult`. There'll be incompatibility issues when you pass in
> in `FastMCP.client.Client` as the MCP session.

Example results in terminal:

```
Library > What books do you have?
We have a diverse collection of books available! Here's our current catalog:
1. The Hitchhiker's Guide to the Galaxy by Douglas Adams
2. The Lord of the Rings by J.R.R. Tolkien
3. The Da Vinci Code by Dan Brown
4. The Hunger Games by Suzanne Collins
5. The Martian by Andy Weir
6. Fahrenheit 451 by Ray Bradbury

...
```

Code here: https://github.com/hxy9243/agents/tree/main/librarian


# Reflections

I quickly realized the limitation of this implementation
when I'm half-way through this: this could come handy
for a librarian with full access to the system. It could accomplish:

- Searching books.
- Checking out books as a librarian for user.
- Return book for the user.

**But to provide a real automated library service accessible to everyone,
we definitely need more setup with user Authentication and Authorization.**

I jumped to the implementation quickly and didn't think it through. And a local
Library module with full access to the underlying data structure wouldn't be
quite a useful protected service.

Think of this agent is at the local library computer self-service desk, where you could login,
converse with the agent, and check out books. The agent needs to verify that
you're you, and your book is checked out / returned to your account.

Currently there are two paradigms of MCP implementation:

See Github MCP Service introduction: https://github.blog/ai-and-ml/generative-ai/a-practical-guide-on-how-to-use-the-github-mcp-server/

- **Local Deployment:** A local integration, running MCP server along with
  the client (with npx, python, docker, etc). It uses a Bearer token for authentication.

  For our use case, we can setup a proxy MCP server locally, which communicates with
  remote API service.

- **Server Deployment:**: Either an dedicated MCP server or MCP proxy that wraps an existing
  API endpoint. It could use OAuth or Bearer token.

  This is useful since users don't have to worry about setting up npx or docker locally.
  You can upgrade the service without nudging users to upgrade. It takes some extra
  steps to host and maintain the service, but like a web service, it's transparent to the user.

These trade-offs need to be considered during implementation.

# Next Steps

I made a naive implementation of a Library and demo-ed how it could be integrated
with frameworks like DSPy.
To make a useful MCP server for service, we need to provide a service and
consider security concerns like authentication.

The next steps could be:

- Multi-tenant support
- Role-based permissions (librarian vs member)
- API key management

# References

- MCP Protocol: https://modelcontextprotocol.io/docs/getting-started/intro
- FastMCP reference: https://gofastmcp.com/getting-started/welcome
- DeepLearning.ai tutorial https://learn.deeplearning.ai/courses/mcp-build-rich-context-ai-apps-with-anthropic/lesson/dbabg/creating-an-mcp-server
- Github MCP: https://github.blog/ai-and-ml/generative-ai/a-practical-guide-on-how-to-use-the-github-mcp-server/
- DSPy MCP Support: https://dspy.ai/tutorials/mcp/
- Understanding DSPy: https://dspy.ai/learn/