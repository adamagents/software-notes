# Load Balancing

Load balancers smooth demand, isolate failures, and keep latency predictable by steering requests across compute pools.

## Summary
- **Fair Utilization**: Keep every healthy instance operating within safe CPU/memory envelopes.
- **Latency Management**: Prefer routes that minimize round-trip time and queueing delay.
- **Fault Isolation**: Detect and quarantine unhealthy targets before they impact upstream systems.
- **Elastic Growth**: Accept new instances seamlessly so autoscaling can add or remove capacity without downtime.

## Architecture Building Blocks
- **Global Traffic Management**: Anycast DNS, Geo-DNS, or BGP announcements route users to the nearest region before regional balancers take over.
- **Layer 4 Balancers**: Distribute TCP/UDP sessions (e.g., AWS NLB) when protocols are opaque or latency budgets are tight.
- **Layer 7 Proxies**: Envoy/NGINX terminate TLS, enforce routing rules, perform rewrites, and implement per-endpoint policies.
- **Client-Side Balancing**: Service meshes and gRPC clients keep endpoint lists locally to avoid centralized chokepoints.

## Scheduling Algorithms
| Algorithm | Use When | Caveats |
| --- | --- | --- |
| Round Robin | Homogeneous, stateless workloads | Ignores live load differences |
| Weighted Round Robin | Mixed instance sizes or on-prem + cloud burst scenarios | Requires accurate weights |
| Least Connections | Variable request cost, long-lived sessions | Needs timely connection tracking |
| Least Response Time | Tail latency matters most | Sensitive to noisy metrics |
| Consistent Hash | Sticky sessions or cache locality required | Rebalancing can cause churn |
| Power of Two Choices | Huge clusters where global state is expensive | Requires quick health snapshots |

## Operational Playbook
- **Health Checks**: Combine TCP, HTTP, and synthetic transactions; evict after N consecutive failures and probe quarantined hosts out of band.
- **Connection Draining**: During deployments, stop sending new traffic and allow keep-alive sessions to finish gracefully.
- **Circuit Breakers**: Trip per-origin breakers when error ratios spike; shed traffic instead of cascading failures downstream.
- **Autoscaling Hooks**: Pause scale-in when error budgets are depleted to avoid removing good capacity prematurely.
- **Traffic Shifting**: Use weighted pools or service mesh routing to canary new versions and monitor per-version metrics.

## Failure Modes & Mitigations
- **Thundering Herd on Recovery**: Enforce jittered retries and exponential backoff so recovered nodes are not flooded instantly.
- **Load Imbalance**: Prefer sliding-window EWMA metrics when feeding schedulers; alert on imbalance ratios (max node load/avg load).
- **Stateful Sessions**: Terminate affinity at the edge (JWTs, distributed caches) so balancers can route requests anywhere.
- **Split-Brain DNS**: Pair DNS with active health checks and short TTLs to avoid handing out dead endpoints.
- **Overloaded Drains**: Ensure draining instances respect max connection lifetimes; kill or migrate stuck keep-alives.

## Observability & Diagnostics
- Export per-target success rate, latency percentiles, active connections, and saturation metrics.
- Trace routing decisions in access logs (e.g., Envoy logs chosen cluster) to explain where a specific request landed.
- Alert on imbalance ratios, failure rates per AZ, and health-check flapping.
- Record every control-plane change (weight updates, upstream config) with audit events plus rollback recipes.

## Interview Drills
1. Design a multi-region load-balancing strategy that survives a full region loss without user-visible downtime.
2. Explain how to keep sticky sessions when migrating from appliance balancers to client-side hashing in a service mesh.
3. Walk through diagnosing intermittent 502s when only one AZ shows elevated response times.
