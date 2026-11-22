# Artifact layouts and rubrics

## Working directory per target
```
<target>/
├── scope.md          # in/out of scope, severity rules, known issues, payout terms
├── src/              # cloned in-scope contracts
├── deps/             # dependencies and integrated protocols
├── lifecycle.md      # main flows, state touched, line references
├── assumptions.md    # trusted roles, oracles, external deps
├── surface.md        # public/external functions ranked by attack surface
├── candidates.md     # raw hunter output (unfiltered)
├── poc/              # one test per candidate (Foundry)
├── judged.md         # valid / invalid / needs-human, with reasons
└── lessons.md        # what to study next
```

## Candidate format
```
ID: C-007
Function: Vault.liquidate
Claim: <one sentence>
Attack steps: 1) ... 2) ...
Claimed impact: <funds lost / frozen / protocol insolvent / chain halt>
Preconditions: <state, roles, timing>
PoC: poc/C007.t.sol (status: passing | failing | not-written)
```

## Judge rubric (give this to the judge agent)
Answer each with evidence from the PoC and scope.md, not from the hunter's description.
1. Is the affected asset in scope?
2. Does the PoC execute with an unprivileged attacker, realistic preconditions, and no cheated state (no vm.store hacks unless modeling a real condition)?
3. Does the assertion prove the claimed impact?
4. Is the impact category listed as rewardable in the program?
5. Does it rely on a trusted role acting maliciously, or on an offchain component? If so, mark invalid unless the program explicitly includes it.
6. Is it a known issue, previously disclosed, or an obvious duplicate?
7. Severity by the program's matrix. If unclear, say so.
Verdict: valid | invalid | needs-human. One-paragraph reason.

## Hunter prompt skeleton
"Target function: <name>. Context: lifecycle.md, assumptions.md, deps/. Look only for <bug class>. For each candidate give attack steps, preconditions, and impact. Say 'none found' if nothing credible; do not pad. Do not assume trusted roles are malicious."
