# Design decisions and trade-offs

This document describes the privately maintained implementation.

| Choice | Reason | Cost / boundary |
|---|---|---|
| PostgreSQL Queue | Durable claims and Run metadata share transactional authority | DB availability/row contention affect scheduling |
| FOR UPDATE SKIP LOCKED | Workers avoid waiting on another claimant's row | Empty polls are valid; not strict fairness |
| Epoch Fencing | Old processes can resume after takeover | Every controlled write must enforce the grant |
| Revision CAS | Owner identity alone cannot order concurrent checkpoints | Reload/reconcile expected Revision on conflict |
| Git Worktree | Isolate writes and preserve an inspectable source baseline | Local Git metadata/lifecycle must be trusted |
| Patch Intent/Result | Record intent across filesystem/SQL commit gap | Reconcile actual hashes; stop on unknown bytes |
| Short guarded transactions | Serialize Patch write with claims and renewal | A long file operation can delay Renew and expire |
| Model/tests outside row locks | Avoid holding locks over network/build latency | Revalidate authority, binding and hash afterward |
| Separate budgets | Crashes, retries and new proposals consume different resources | More counters, but takeover cannot reset them |
| Retained package names | Preserve tested component boundaries | Less conventional naming, without risky mass refactor |

Diagnosis persists model/tool intent in history before execution, then outputs
in a later checkpoint. A pending call can replay after crash: durable ordering
does not guarantee arbitrary external-side-effect exactly-once. Controlled
Patch effects permit more specific recovery via Before/After SHA.

This public edition publishes documentation only. Implementation source,
execution materials and a software license are not included.
