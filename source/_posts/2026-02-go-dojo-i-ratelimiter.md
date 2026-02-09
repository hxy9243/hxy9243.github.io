title: Golang Dojo Project I - Rate Limiter
date: 2026-02-08 17:45:08
tags: ["opensource", "programming", "computer system"]
---

# Learning Golang with Real Projects

There's a particular itch I get when learning new tech concepts. Reading docs and watching tutorials helps, but the knowledge stays hazy until I open the hood and build something myself. So I started a project: implement small but interesting systems in Go, working with the garage door open.

This isn't pure "vibe coding." I'm using AI to scaffold and accelerate, but the goal is understanding, not just shipping. The first "Hello World" system: a **rate limiter**. It has enough complexity to be interesting, bounded enough to complete in a short span.

I started off the implementation by auto-complete with an AI copilot first, nothing more. And throughout this project, I've learned more than I expected. Now I truly believe that creating something greatly helps understanding. And AI helps much of the implementation burdens.

In the future, I'll experiment with more sophisticated "vibe-coding" techniques and see how far I'll go.

<!--more-->

## Architecture and Algorithm

## Sliding Window

Three common approaches:

- **Fixed Window**
  - Pro: Simple, memory efficient
  - Con: Burst at window boundaries
- **Sliding Window**
  -  Pro: Smooth rate limiting, no bursts
  -  Con: More complex, requires sorted data
- **Token Bucket**
  - Pro: Allows bursts, easy to understand
  - Con: Can be complex to implement at scale

I chose **sliding window log** because it provides the smoothest limiting and avoids the bursting problem at window boundaries.

The trade-off: it's slightly more complicated, as you need to store and query timestamps efficiently.


## The Data Structure: Skip Lists in Redis

Here's where it gets interesting. To implement sliding window, you need to:

1. **Insert** a new timestamp (when request arrives)
2. **Delete** timestamps outside the current window
3. **Count** remaining timestamps (to check against limit)

All of these need to be fast—this runs on every API request.

Redis implements sorted sets using a **skip list**. They're beautiful: a probabilistic data structure that gives O(log N) search/insert/delete with less complexity than balanced trees.

A skip list is essentially a linked list with "express lanes." Each element has a random "height" (typically coin-flip based). Level 0 connects all elements. Higher levels skip ahead:

```
Level 2:  head ──────────────────────► 50
              │                         │
Level 1:  head ─────────► 20 ─────────► 50 ──────► 80
              │            │            │           │
Level 0:  head ──► 10 ──► 20 ──► 30 ──► 50 ──► 70 ──► 80 ──► 100
```

**Why this matters for us:**

- Insert: `ZADD`, `O(log N)`, Log new request timestamp
- Range delete: `ZREMRANGEBYSCORE`, `O(log N + M)`, Remove old timestamps outside window
- Count: `ZCARD`, `O(1)*`, Check current request count

Compare to a simple list: insertion might be O(1) but finding and removing old entries is O(N).


## Architecture Diagram

The meat of the problem focuses on the Rate Limiter Middleware. But to demo the work, I've created example
services to showcase how the middleware could be used as a part of the service implementation.

![Architecture Diagram](./2026/02/08/go-dojo-i-ratelimiter/architecture.svg)

# Implementation

## The Lua Script

The critical path needs to be atomic — no race conditions when multiple requests hit simultaneously. Redis Lua scripts execute atomically.

For the atomic operation, you'll need to:

- Determine if the count is already over the limit.
	- Remove the records outside the window (`ZREMRANGEBYSCORE`).
	- Count the current score using `ZCARD`.
- If less than limit:
	- Add a new record (`ZADD`), optionally set an expiry.
- Else:
	- Immediately return over-the-limit error.

The algorithm should be straight-forward enough to do in a Lua script:

```lua
local key = KEYS[1]           -- rate limit key (e.g., "ratelimit:user123")
local value = KEYS[2]         -- unique request identifier (timestamp + random)
local limit = tonumber(ARGV[1])    -- max requests allowed
local window = tonumber(ARGV[2])   -- window size in seconds
local now = tonumber(ARGV[3])      -- current timestamp in ms

-- Remove all timestamps outside the sliding window
redis.call('ZREMRANGEBYSCORE', key, 0, now - (1000 * window))

-- Count how many requests remain in current window
local current = redis.call('ZCARD', key)

if current < limit then
    -- Under limit: add this request and set expiry
    redis.call('ZADD', key, now, value)
    redis.call('EXPIRE', key, 2 * window)
    return 1  -- Allow request
else
    return 0  -- Rate limit exceeded
end
```



## Go Middleware Skeleton

To create the Go Middleware, you can use the Go HTTP library. This part should be straightforward enough, it:

- Accepts request from the previous service, checks for the expected header (`X-Client-Id`).
- Queries Redis for rate limits.
- If success: queries the backend service and return.
- If Error: returns 429.

```go
type RateLimiterOptions struct {
	Limit  int64
	Window int64

	MasterName    string
	SentinelAddrs []string
	Password      string
}

type RateLimiterClient struct {
	redisConn   *redis.Client
	redisScript *redis.Script
	options     RateLimiterOptions
}

// Middleware returns an HTTP middleware for HTTP servers to check rate limits
func (rl *RateLimiterClient) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// get request client identifier from request header
		clientId := r.Header.Get("X-Client-Id")
		if clientId == "" {
			log.Print("X-Client-Id header is required")

			w.WriteHeader(http.StatusBadRequest)
			json.NewEncoder(w).Encode(map[string]interface{}{"error": "X-Client-Id header is required", "status": http.StatusBadRequest})
			return
		}

		allow, err := rl.Allow(r.Context(), clientId)
		if err != nil {
			log.Printf("Error checking rate limiter: %s", err)

			w.WriteHeader(http.StatusInternalServerError)
			json.NewEncoder(w).Encode(map[string]interface{}{"error": "Internal error", "status": http.StatusInternalServerError})
			return
		}

		if !allow {
			log.Printf("Too many requests from: %s", clientId)

			w.WriteHeader(http.StatusTooManyRequests)
			json.NewEncoder(w).Encode(map[string]interface{}{"error": "Too many requests", "status": http.StatusTooManyRequests})
			return
		}

		next.ServeHTTP(w, r)
	})
}
```


## Redis High Availability: What I Learned

I used to treat Redis as a "magic fast database." Building this forced me to understand the operational side.

In order to use Redis in production, you'll need to take into account scalability and reliability. There are a few methods of
deploying them.

### Sentinel vs Clustering Mode

- **Sentinel**: High availability with automatic failover. Single primary (write bottleneck), simpler.
- **Clustering**: Horizontal scaling, data sharding. Complex client setup, resharding challenges.

For a rate limiter, Sentinel is usually sufficient—you're doing simple key operations that don't need sharding. But knowing the difference matters when you hit scale.

### Production Considerations

- **Memory pressure**: Sliding windows with many clients = memory growth. Set appropriate `MAXMEMORY` policies
- **Availability and Scalability**: You need to consider for your use case, whether setting up Redis Sentinel is sufficient. Redis Cluster provides better scalability and availability, at the cost of consistency. See more:
	- Redis Sentinel Mode: https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/
	- Redis Clustering Mode: https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/

## Kubernetes Deployment

One of the joys of vibe coding with AI: generating Kubernetes manifests is now trivial. But understanding what they do is still your responsibility.

Key insight: **the rate limiter is stateless**. All state lives in Redis. This means you can scale the Go service horizontally without coordination headaches. The complexity is pushed to Redis, which is designed for exactly this.

# Vibe Coding

Here's how I actually worked on this:

- **Design Discussion**: Spent some good quality time discussing with ChatGPT on the overall architecture, algorithm, and technical choices.
- **Scaffold**: Have the AI create a boilerplate.
- **Implementation**: I fill in the actual implementation with the help of Copilot autocomplete. For core algorithms and Lua Script, do the research, create the implementation and validate with AI conversations.
- **Infrastructure & Deployment**: Create the KIND setup, dockerfile, with the assistance of AI, and bootstrap the project.
- **Testing**: Actual manual testing. AI can help generating scripts to test the system locally.

# What I Learned

AI greatly helped and remove much of the pain for bootstrapping the project:

- It accelerates scaffolding by at least 10x, like Dockerfile, K8s Helm, environment setup, etc.
- It can debate with you and help you design the overall system architecture.
- Writing tests forces you to understand behavior the AI implied but didn't guarantee.

Sure, AI can greatly improve education of Software Engineering concepts. And it can greatly smoothen
the entire implementation process.

But, feel free to point out that I'm wrong on this, AI still doesn't fully replace:

- A good grasp of the data structure, algorithm, system knowledge.
- For designing the architecture of a complicated system.
- Decide on the trade-offs in a system design, and implement the system from end-to-end.

That's why I still believe good system engineering cannot be easily replaced by AI yet.
I'll keep playing around with vibe-coding and find out AI's limits.

# Final Thoughts

Disclaimer: the project is for demo and educational purpose, and by no means production-ready.

Everything is open source: [https://github.com/hxy9243/go-dojo](https://github.com/hxy9243/go-dojo)

The repo includes:

- Complete rate limiter implementation.
- A demo server deployable on K8s environment.
- KIND setup for local development.
- Helm charts for K8s deployment.

Building a rate limiter scratched the itch, but there are more small system concepts that could be built. Each one teaches something different about the boundaries between algorithms, data structures, and real-world constraints.

Feedback and suggestions are welcome! Hit me up—always curious how others approach system building with AI assistance.


# References

- Rate Limiter Design: [https://medium.com/phyllo-engineering-blog/rate-limiter-using-sorted-set-in-cache-redis-bee5d939448d](https://medium.com/phyllo-engineering-blog/rate-limiter-using-sorted-set-in-cache-redis-bee5d939448d)
- Rate Limiting with Redis: [https://redis.io/tutorials/howtos/ratelimiting/](https://redis.io/tutorials/howtos/ratelimiting/)
- Redis Sentinel Mode: [https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
- Redis Clustering Mode: [https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)
