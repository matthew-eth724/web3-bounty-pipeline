---
name: web3-bounty-pipeline
description: Run an evidence-gated, AI-driven workflow for finding smart contract vulnerabilities in authorized bug bounties and audit contests, designed for a user who lacks deep web3 security expertise. Use whenever the user wants to hunt bugs in a bounty or contest scope, triage a Solidity/Move/Rust codebase, build or run an agent pipeline for vulnerability research, validate or judge a suspected finding, write a Foundry fork PoC, or decide which bounty programs are worth their time. Trigger even if they only say "audit this contract", "is this a valid finding", "pick a bounty", or "set up my hunting agents".
---

# Web3 Bounty Pipeline

A workflow for AI-driven vulnerability research where the human cannot yet judge whether a finding is plausible. The expert's filter (reading the code line by line and knowing what looks wrong) is replaced by **executable evidence**: nothing is a finding until a test reproduces it. Models generate candidates cheaply and are confidently wrong often, so every stage exists to cut noise, not add volume.

Model: *generate → execute → gate*, with learning built in so the human improves over time.

## Ground rules

- Work only inside programs the user is authorized to test (public bounty or contest scope). Read the scope page first and record in-scope assets, out-of-scope items, severity rules, known issues, and payout conditions.
- Run PoCs on a **local fork** (Foundry/Anvil). Never send exploit transactions to live networks.
- Invalid or spammy submissions can get researchers penalized or banned. Precision beats volume.
- Be honest about confidence. If a bug class can't be proven by a test (economic or design-level issues), say so and mark it "unproven" rather than inflating it.

## Pipeline

Keep state in a working directory per target (see `references/artifacts.md` for file layouts). Complete each stage before the next.

### 1. Select the program
Skip or flag a program if any of these hold:
- Bounty page not updated in a year or more, and the project looks wound down (check activity, repo, socials).
- Low TVL (check DefiLlama). A critical on a tiny protocol pays little.
- Heavy reliance on offchain backends or trusted roles. Findings get invalidated as "assumes trusted component" because the offchain code can't be inspected.
Prefer fully onchain scope where an attacker-executable bug can be verified end to end. Also weigh novelty: chain-specific logic, custom VMs, consensus and transaction validation get far less AI-pipeline competition than popular DeFi forks.

### 2. Build context
- Clone in-scope contracts locally (block explorer fetch for deployed code, repo for source).
- Resolve dependencies and integrations into a `deps/` folder so agents see the protocols the target touches.
- Have an agent write `lifecycle.md`: the main user flows in order (deposit → withdraw → borrow → liquidate, etc.), what state each touches, and `assumptions.md`: every trusted role, oracle, and external dependency.
- **Teach while building.** Have the agent explain each flow in plain language and cite line numbers so the user can check claims against code. This is how the user builds real judgment.

### 3. Map attack surface
List all external and public user-callable functions, ranked by attack surface size (complex, multi-step, value-moving functions first, trivial deposit-style ones last). Output `surface.md`. Work down the ranking.

### 4. Hunt
Run focused hunters per function or flow, not one giant "find bugs" prompt:
- External call enumeration, then per-call checks (reentrancy, return data handling, unchecked returns, return data bombs).
- Accounting and rounding (share/asset math, first-depositor, donation, precision loss).
- Access control and initialization.
- Oracle and price manipulation, flash-loan reachable state.
- Invariants: have agents propose invariants ("total shares x price <= assets", "sum of balances == total supply"), then reason about every function that mutates the relevant state to try to break them.
- Chain-specific behavior for non-EVM or custom chains: look for logic that wouldn't exist on other chains.
Collect raw candidates in `candidates.md` with: claim, affected function, attack steps, claimed impact. Do not filter yet.

### 5. Hard gate: PoC on a fork
For every candidate, write a Foundry test (or the target ecosystem's equivalent) that reproduces the exploit and asserts the violation (attacker profit, broken invariant, frozen funds, halted state). Use a fork of the real chain state when the bug depends on deployed state.
- Test passes and demonstrates impact: candidate survives.
- Test fails, can't be written, or only passes under unrealistic setup: drop it, or park it under "unproven" with the reason.
Treat a failing PoC as information. Iterate at most a few times, then move on. Never weaken the assertion to make a test pass.

### 6. Judge
Use a separate agent, ideally a different model than the hunter, with a strict rubric (see `references/artifacts.md`). Check each surviving finding against:
- In scope? Matches an allowed impact category?
- Severity per the program's own rules, not generic CVSS.
- Depends on a trusted role or offchain assumption that makes it invalid?
- On the known-issues list or already fixed? Likely duplicate? Search prior reports (Solodit, Code4rena, Sherlock, the program's disclosed reports).
- Does the PoC actually show the claimed impact, or just a revert or a no-op?
Judge output: `valid`, `invalid`, or `needs-human`, with reasons.

### 7. Human review
Only survivors reach the user. Present each as: one-paragraph plain-language explanation, the PoC command to run, impact, severity per program rules, duplicate risk, and what the user should verify themselves. Then help draft the report.

## Closing the knowledge gap

- Build a retrieval base of past findings (Solodit, public contest reports) and have hunters cite similar historical bug classes.
- Start with bug classes where PoCs are mechanical (rounding, access control, reentrancy) before economic or governance attacks.
- After each target, write `lessons.md`: what the agent got wrong, what the PoC revealed, and which concepts the user should study. Surface these when starting the next target.

## Known limits (state these to the user when relevant)
- High duplicate rates: popular programs are scanned by many AI pipelines. Differentiated bugs tend to come from chain-specific or consensus-level logic.
- Design and economic flaws are often unprovable by a simple test and will be underreported without human expertise.
- A same-model judge shares the hunter's blind spots.
- Self-reported or model-asserted impact is not evidence. Only the PoC is.
- Bounty income is unstable (duplicates, ghosting by off-platform programs). Don't promise returns.

## Output when the user asks for a status or result
Lead with counts (candidates, PoC-passing, judged valid), then the surviving findings, then the unproven list, then recommended next steps. Keep it short; link to files for detail.
