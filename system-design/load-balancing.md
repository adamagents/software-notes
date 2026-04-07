# Load Balancing

Load balancers absorb fluctuating demand, shield unhealthy nodes, and keep latency predictable by steering requests intelligently across compute pools.

## System Goals
- **Fair Utilization**: Keep every healthy instance within a safe CPU/memory envelope.
- **Latency Control**: Prefer routes that minimize round-trip time and queueing delays.
- **Fault Isolation**: Detect and quarantine bad nodes before they poison upstream capacity.
- **Elastic Growth**: Accept new instances seamlessly so autoscaling can add/remove capacity.

## Core Architectures
1. **Global Traffic Management**: Anycast DNS, Geo-DNS, or BGP announcements route users to the nearest region before regional balancers take over.
2. **Layer 7 Proxies**: Envoy/NGINX terminate TLS, apply routing rules, rewrites, and per-endpoint policies.
3. **Layer 4 Load Balancers**: Simple TCP/UDP load spreading (e.g., AWS NLB) when application protocols are opaque.
4. **Client-Side Balancing**: Service meshes and gRPC clients keep endpoint lists and pick targets locally to avoid centralized chokepoints.

## Algorithms & When to Use Them
| Algorithm | Pick When | Caveats |
| --- | --- | --- |
| Round Robin | Homogeneous nodes, stateless work | Ignores live load differences |
| Weighted Round Robin | Mixed instance sizes (e.g., on-prem + cloud burst) | Requires accurate weight tuning |
| Least Connections | Variable request cost, long-lived sessions | Needs timely connection counts |
| Least Response Time | Tail latency matters | Sensitive to outliers/noisy metrics |
| Consistent Hash | Sticky sessions or cache locality required | Rebalancing may cause cache churn |
| Power of Two Choices | Huge clusters where global state is expensive | Needs quick health snapshots |

## Operational Playbook
- **Health Checks**: Combine TCP, HTTP, and synthetic transaction checks; evict after N consecutive failures and probe quarantined hosts out-of-band.
- **Connection Draining**: During deployments, stop sending new traffic, allowing keep-alive sessions to finish gracefully.
- **Circuit Breakers**: Trip per-origin breaker when error ratios exceed thresholds; shed traffic instead of cascading failure downstream.
- **Autoscaling Hooks**: Pause scale-in when error budgets are depleted to avoid removing good capacity prematurely.

## Failure Modes & Mitigations
- **Thundering Herd on Recovery**: Use jittered retry budgets and exponential backoff so recovered nodes do not get flooded immediately.
- **Load Imbalance from Slow Metrics**: Prefer sliding-window EWMA measurements rather than cumulative averages when feeding schedulers.
- **Stateful Sessions**: Terminate affinity at the edge (JWTs, distributed caches) so balancers are free to route anywhere.
- **Split-Brain DNS**: Monitor authoritative DNS health; pair DNS with health-checked load balancers to avoid handing out dead endpoints.

## Observability
- Export per-target success rate, latency percentiles, active connections, and saturation metrics.
- Trace routing decisions (e.g., Envoy access logs with chosen cluster) to explain why specific requests hit specific backends.
- Alert on imbalance ratios (max node load / avg load) to catch skewed pools early.

## Diagnostic Checklist
- Can you drain or quarantine a single instance/AZ/region without impacting SLOs? If not, add automation for progressive ramp-downs.
- Does every control plane change (weights, upstream configs) emit an audit event plus a rollback recipe?
- How quickly do health signals converge compared to the average failover window? Slow convergence causes flapping.
- Are clients retrying with jitter and exponential backoff, or accidentally stampeding the same recovering hosts?
- Is there an enforced config review (lint/tests) that blocks invalid routing rules before they reach production?

## Quick Reference
| Scenario | Preferred Strategy | Notes |
| --- | --- | --- |
| Sudden regional spike | Shift via global load balancer + autoscaling warm pool | Keep spare capacity or spot/scale set ready |
| Making sticky sessions stateless | Issue signed tokens / store session in distributed cache | Allows L7 balancers to treat requests uniformly |
| Migrating traffic between versions | Use weighted target groups or service mesh traffic shifting | Instrument per-version canary metrics |
| Handling long-lived connections | Consider L4 balancers with connection preemption + draining | Track max-connections per host |
| Protecting databases from retry storms | Circuit breakers + retry budgets propagated to clients | Pair with load-shed responses (429/503) |

## Interview Drills
1. Design a multi-region load-balancing strategy that survives a full region loss without global outage.
2. Explain how to keep sticky sessions when moving from appliance load balancers to service-mesh-based client-side hashing.
3. Walk through diagnosing intermittent 502s when only one AZ shows elevated response times.
