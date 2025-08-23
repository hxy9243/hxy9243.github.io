title: Building a School Library Assistant with FastMCP and DSPy (II)
date: 2025-08-22 21:56:34
comments: true
categories: 'Programming'
tags: ['Learning', 'AI']
---


The last time I dived into building a toy MCP service for a school library
assistant, which helps you search for books, borrow copies, and return them.
I quickly realized this wasn't practical. Without authentication and authorization, the service was only useful for an administrator with full access.

What if we want to expose a real service? Security needs to be a first-class
citizen.

MCP service itself is not that mysterious. It's essentially built upon
HTTP protocol with an extra layer of protocol and formatting. We can
leverage all the concept for building security for HTTP services, like TLS, OAuth, etc.
With the [FastMCP](https://gofastmcp.com/) framework we can extend the Authn/Authz, just
like the Python backend development process.

This is a toy implementation, but I think can demo the basic ideas for MCP service and
LLM integration.

Code here: https://github.com/hxy9243/agents/tree/main/librarian

See last blog: https://blog.kevinhu.me/2025/08/09/Building-MCP-with-DSPy/


<!--more-->

# MCP Service Architecture

As mentioned here on [Github MCP service introduction](https://github.blog/ai-and-ml/generative-ai/a-practical-guide-on-how-to-use-the-github-mcp-server/),
there are more than one way to provide MCP service to client/users.

- **Local Docker/Python Server**: A local integration, running MCP server along with
  the client (with npx, python, docker, etc). And the local service.

  The MCP server runs locally on your machine, and then communicates with the service provider with
  security token.
- **MCP Service Endpoint**: An streamable HTTP endpoint, sign in with OAuth, with security
  taken care for you.

  The MCP server runs on the service provider host, and the MCP client connects directly to
  the endpoint, with no more indirections in between.

The benefit of a service endpoint is that you don't have to worry about local setup, deployment, and upgrade.
Over time, I believe this is the right direction for services that want to provide a good experience.
You only need to provide a single HTTP endpoint, and the users need to worry about nothing else.

# FastMCP Middleware

[FastMCP](https://github.com/jlowin/fastmcp) is quite a nifty framework for quickly
setting up MCP servers/clients in Python.
It recently introduced Middleware for MCP server. Users can now easily use that to
intercept calls, implement logging, monitoring, and Auths.

And we can use that to implement Authn/Authz, the same way to implement for any HTTP service.


# Implement (A Really Naive) Auth


This is a very high-level overview of what that could look like.

```python
class UserAuthMiddleware(Middleware):
    async def on_call_tool(self, context, call_next):
        headers = context.fastmcp_context.request_context.request.headers

        # parse tokens out of headers
        token = get_auth_from_headers(headers)
        if not token:
            raise HTTPException(status_code=401, detail="missing token")

        ...

        # save in context for next step of authz
        context.fastmcp_context.set_state("member_id", member_id)

        return await call_next(context)
```

FastMCP also provides OAuth with third-party integration, like Google Auth, Azure, etc.
See examples: https://gofastmcp.com/integrations/google.

The `on_call_tool` can work as a hook when a tool is called. There are other hooks for
the entire lifecycle of a client request. See: https://gofastmcp.com/servers/middleware#hook-hierarchy-and-execution-order

# Assistant Experience

Finally, we made an example service endpoint that we could provide to all users
of the library, each with their own login and authorization. They can
access their own information with enhanced security.

Like described in the last blog, we can now integrate the MCP tools into
a DSPy agent and create an chat application:

```python
    # self.mcp_session is session from MCP client session member
    async with Client(...) as client:
        mcp_session = client.session

        tools = await mcp_session.list_tools()
        for tool in tools.tools:
            dspy_tools.append(dspy.Tool.from_mcp_tool(mcp_session, tool))

        self.lm = dspy.ReAct(
            signature=LibrarianSignature,
            tools=dspy_tools,
        )
```

(Notice, DSPy requires `mcp.session` and `mcp.types.ListToolResult` instead of types defined in FastMCP.
However, FastMCP provides easy way of conversion.)

We can now run it with the example auth token for the assistant:

```
# Token for Alice
export TEST_TOKEN=token_001

uv run python assistant.py
Welcome, user Alice
Library > What books have I borrowed?
You have borrowed the following book:
- "1984" by George Orwell
Library > What's the due dates?
The due date for "1984" by George Orwell is 2025-09-05.
```

But you should be refused access to other users' information:

```
Library > What books did bob borrow?
I'm sorry, but I am not authorized to access information about other members, including what books Bob has borrowed.
```

Code here: https://github.com/hxy9243/agents/tree/main/librarian


# FastMCP OAuth

OAuth is a common pattern for HTTP services, and FastMCP has incorporated it in the framework
as well. You can delegate Authentication to third-party services.

See more: https://gofastmcp.com/clients/auth/oauth


# Final Thoughts

Like the HTTP web protocol, MCP is a powerful abstraction to connect services, capabilities,
and functionalities to end users. It reuses a lot of concepts and even implementations from the HTTP protocol.
There's nothing mysterious about it. With enough digging, we can
implement cool and interesting ideas around it.

There's so much more to MCP security setup if we want to use this in production, think
about TLS, OAuth, namespace, user capability management, Authorization setup, etc,
not to mention load balancing, availability, etc.

I'm excited that AI can so quickly create so many new opportunities to explore
our curiosity. The possibility is infinite.

# References

- FastMCP: https://gofastmcp.com/getting-started/welcome
- FastMCP Bearer: https://gofastmcp.com/clients/auth/bearer
- FastMCP Middleware: https://gofastmcp.com/servers/middleware
- FastMCP Google OAuth: https://gofastmcp.com/integrations/google
- MCP Protocol: https://modelcontextprotocol.io/docs/getting-started/intro
- DSPy MCP Support: https://dspy.ai/tutorials/mcp/
- Understanding DSPy: https://dspy.ai/learn/
- Github MCP: https://github.blog/ai-and-ml/generative-ai/a-practical-guide-on-how-to-use-the-github-mcp-server/