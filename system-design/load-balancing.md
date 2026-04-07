# Load Balancing

Effective load balancing keeps distributed systems responsive and resilient by spreading requests across compute resources, minimizing hotspots, and isolating failures before they cascade.

## Core Goals
- **Evenly utilize capacity** so no single node throttles throughput.
- **Reduce latency** by routing users to the nearest or least busy backend.
- **Provide resilience** through rapid detection and removal of unhealthy targets.
- **Enable elasticity** by making it safe to add or remove instances at runtime.

## Common Algorithms
| Algorithm | How It Works | Strengths | Watch Outs |
| --- | --- | --- | --- |
| Round Robin | Sequence through hosts in order | Simple, stateless | Ignores load/latency differences |
| Weighted Round Robin | Assigns higher frequency to powerful nodes | Honors heterogenous capacity | Needs periodic weight tuning |
| Least Connections | Chooses target with fewest active requests | Reacts to uneven work | Requires up-to-date connection counts |
| Least Response Time | Tracks observed latency per target | Optimizes tail latency | Sensitive to noisy measurements |
| Hash-Based | Hash on user/session/id to pick host | Sticky sessions without cookies | Rebalancing causes cache churn |
| Random with Two Choices | Sample two hosts, pick better metric | Near-optimal balance with low state | Requires consistent health data |

## Architectural Patterns
- **Global vs. Regional Balancers**: DNS-based or Anycast layers steer traffic into regions, then regional L4/L7 balancers distribute within a data center.
- **Layer 4 (Transport) Balancing**: Operates on TCP/UDP tuples using NAT; low overhead but limited application insight.
- **Layer 7 (Application) Balancing**: Terminates TLS/HTTP, can route by URL path, headers, content type, or user metadata.
- **Client-Side Balancing**: Smart clients fetch service registries (e.g., via Consul/Eureka) and pick targets locally, reducing centralized chokepoints.
- **Edge + Internal Tiers**: CDN/WAF at the edge, gateway/load balancer pair at ingress, then service mesh or per-service balancers deeper in the stack.

## Health Checking & Failover
- Out-of-band health checks (HTTP/TCP) plus in-band request success metrics prevent relying on a single signal.
- Mark hosts as *draining* before maintenance to allow graceful connection wind-down.
- Implement circuit breakers per target to stop flapping hosts from rejoining too quickly.
- Keep failover domains well-scoped: prefer regional rerouting before cross-continent failovers to avoid massive cold starts.

## Stateful Traffic Strategies
- Prefer stateless services so any node can serve any request; where sticky routing is required (shopping carts, WebSockets), back it with shared state (Redis, session DB) or replicate the data via consistent hashing so the balancer can still move traffic when needed.
- For gRPC and long-lived streams, use **subchannel pools** with periodic rebalancing to avoid permanent skew.

## Observability Checklist
- Export per-target request rate, latency histogram, and HTTP status distribution.
- Alert on imbalance indicators: max/min load ratio, percentage of traffic hitting degraded nodes.
- Record routing decisions (e.g., via access logs with backend IDs) to debug intermittent customer issues quickly.

## Interview / Design Questions
1. How would you scale a global API gateway from 10k to 1M RPS?
2. What happens if your health checks are too aggressive or too lax?
3. How do you handle blue/green deploys without dropping long-lived connections?
4. How would you route premium customers to beefier hosts without duplicating fleets?
