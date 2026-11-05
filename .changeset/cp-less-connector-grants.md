---
'@substrat-run/adapter-cloudflare': patch
---

A CP-less host answers `connectionGrantsInScope` from its own scope tuples instead of asking a tenant directory it does not have. The read went through `this.cp.dumpTenantTuples` unconditionally, which throws `control plane unavailable: 'dumpTenantTuples'` on that shape — and the connector landing pass reads exactly this before it grants (`LandingPass.apply` → `grantsIn`, delegated to the serving deployment through `/internal/connector-grants`). So on every hosted vertical the pass failed before granting, the declared landing's key was never delivered, and since the binders stage the binding first (#2706) every pass reported "1 declared landing could not be granted or bound; 1 of 1 binding did not land" for ever. On a CP-less host the tenant-level grants are projected into the scope DO, so the DO is the whole answer; a host with a control plane still reads both stores.
