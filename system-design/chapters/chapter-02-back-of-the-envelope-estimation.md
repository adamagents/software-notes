# Chapter 2: Back-of-the-Envelope Estimation

## Summary

This chapter turns rough estimation into a design tool. The point is not exact arithmetic; it is to build intuition for whether a design is plausible before discussing detailed architecture.

## Key Ideas

- Keep powers of two and storage units in your head so you can estimate memory and disk needs quickly.
- Memorize relative latency costs: memory is cheap, disk seeks are expensive, and cross-region communication is slower still.
- Translate availability targets into downtime budgets using the "number of nines" model.
- Convert product assumptions into operational numbers such as QPS, peak QPS, storage growth, and bandwidth.
- Show your assumptions explicitly so the interviewer can correct them before the math drifts.
- Round aggressively. Clean approximations are better than precise-looking numbers with hidden mistakes.

## Interview Takeaway

Use estimations to constrain design choices. If the write rate is low, a simple database may be fine. If storage growth is petabyte-scale, the design must address sharding, archival, and cost from the start.
