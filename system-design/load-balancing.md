# Load Balancing

## Overview
Load balancers preserve latency budgets, protect downstream capacity, and create predictable failure domains. They absorb noisy traffic, keep compute fleets evenly utilized, and give platform teams a single policy surface for security and observability. The best architectures combine global routing, regional balancing, and client-side resiliency so every hop can survive partial outages.

## Core Challenges
- **Transport diversity**: HTTP, gRPC, WebSocket, and TCP each require different termination, header handling, and keepalive policies.
- **Stateful workloads**: Sticky sessions, streaming connections, and websocket fan-out need affinity controls without sacrificing failover guarantees.
- **Elasticity**: Autoscalers must add and remove instances without draining the pool or causing cold-start cascades.
- **Policy layering**: TLS, auth, rate limits, and WAF rules must execute consistently whether traffic arrives through DNS steering, anycast, or client libraries.

## Architecture Building Blocks
- **Global traffic management**: Anycast or GeoDNS measures health across regions and hands clients a close, healthy edge before regional balancers get involved.
- **Edge proxies**: Layer 7 components (Envoy, NGINX, HAProxy) terminate TLS, normalize headers, set timeouts, and enforce circuit breakers.
- **Layer 4 switches**: Network load balancers distribute TCP/UDP traffic for protocols that cannot tolerate HTTP-aware proxies.
- **Client-side libraries or service mesh**: SDKs or sidecars keep endpoint lists locally, implement retries with jitter, and expose per-request telemetry.
- **Control plane**: Stores upstream definitions, weights, security policies, and rollout metadata with audit logs and rollbacks.

## Design Checklist
- Document routing goals: latency, fairness, locality, or stickiness. Different goals require distinct algorithms (round robin, least outstanding requests, consistent hash, power-of-two choices).
- Keep health checks layered: lightweight TCP probes for basic reachability, HTTP checks for app health, and synthetic transactions for end-to-end guarantees.
- Use connection draining during deploys and AZ evacuations so in-flight requests finish gracefully before instances leave the pool.
- Separate read and write traffic or noisy neighbors with priority queues or separate listener ports.
- Version routing configurations and ship them via GitOps or config repos so you can diff and revert quickly.

## Failure Modes and Mitigations
- **Thundering herd on recovery**: When a pool returns, weighted warmup or token buckets keep it from immediately taking full traffic.
- **Stuck connections**: Track max lifetime and idle timeout per protocol; close zombie sockets to free resources.
- **Split brain DNS**: Keep TTLs short and couple DNS answers with active health checks so dead regions stop receiving new sessions.
- **Uneven distribution**: Feed schedulers with EWMA latency and success metrics instead of instantaneous counters to avoid oscillation.
- **Cascading retries**: Enforce global budgets for retries and timeouts; client libraries should honor `Retry-After` or gRPC status metadata.

## Observability and Operations
- Emit RED/USE metrics per listener, backend, and AZ. Include labels for TLS version, HTTP verb, and upstream service.
- Log chosen upstream, response codes, connection IDs, and trace IDs so distributed traces can stitch together hops.
- Continuously test failover paths with chaos tooling that drains nodes, injects latency, or corrupts TLS certificates.
- Automate certificate renewal, configuration diffing, and security patching to avoid manual drift.
- Record every weight or routing change with the operator identity and ticket reference for forensic clarity.

## Interview Prompts
1. Design a multi-region API front door that keeps 99th percentile latency under 150 ms even if one region disappears. Detail DNS, transport, and health-check choices.
2. Explain how you would migrate from appliance load balancers to service-mesh-based client-side balancing without downtime.
3. Walk through debugging sporadic 502 errors when only one AZ reports elevated queueing time—what metrics and logs do you inspect first?
