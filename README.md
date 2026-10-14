<!-- solve-problem:bf6bb792-0bf5-4cfd-8e91-7d6a7dfbdff8 -->
# Profit versus statistical detectability

A [Tangle Solve problem](https://solve.tangle.tools/problems/bf6bb792-0bf5-4cfd-8e91-7d6a7dfbdff8).

## What is the problem?

Consider a proof-of-work blockchain with n miners, target share α of total hashpower controlled by an attacker. The attacker deviates from the honest protocol (e.g., block withholding, selfish mining, hybrid strategies that selectively withhold and release blocks). Honest behavior produces an observable stochastic process — the fork/depth distribution of blocks, orphan rates, inter-block times, and which miner finds which block over time.

A defender runs a statistical detector D on a window of T observations, with false-positive rate ≤ δ (probability of flagging an honest network) and detection delay ≤ τ (bounded lag between attack onset and flag). The attacker chooses a strategy σ to maximize excess revenue R(σ, α) relative to honest proportional share.

**The open problem:** characterize the minimax tradeoff surface
  V(α, δ, τ, T) = sup over attacker strategies σ of R(σ, α), subject to σ being undetectable (or δ-hard to detect) by any detector with false-positive ≤ δ and delay ≤ τ.
Equivalently: bound the achievable mining advantage as a function of detector constraints, and *identify the set of strategies lying in the "statistical shadow" of honest network variance* — attacks whose observable footprint is within the noise envelope of honest forking/stale blocks. A matching impossibility result would show no detector meeting the constraints can distinguish certain profitable attacks from honest behavior.

Ambiguity to note: the census does not fix the observation model (full block-tree vs. only published chain), the parametric family of the detector, or whether the adversary is adaptive (changes strategy after being flagged). The most reasonable formalization is a sequential hypothesis test between "all honest" and "attacker with strategy σ" on the public block-tree process.

## Why does it matter?

Monitoring-based detection is the standard practical defense against selfish-mining-style attacks; if profitable attacks exist that are information-theoretically or statistically indistinguishable from honest variance under realistic false-positive/delay budgets, then detection alone can never secure rational-attacker blockchains. This determines whether exchanges, pools, and protocol designers can rely on anomaly detection or must change consensus incentives (e.g., fork-choice rules, fee structures) instead. It also directly quantifies the residual risk retained by any deployed monitoring system.

## What's already known?

- Classical selfish mining (Eyal–Sirer 2014) yields excess revenue above a threshold α ≈ 25–33% (depending on propagation parameters); its statistical footprint (elevated fork/stale rates) is detectable by detectors such as the outlier/fork-detection schemes of Solat–Potop-Butucaru and others. (verify exact attributions)
- Subsequent work constructed *optimal* selfish strategies (e.g., Sapirshtein et al. 2016 via MDPs) maximizing revenue, and *stealthier* variants (e.g., "optimal selfish mining with reduced forking", under-publishing strategies) trading revenue for lower observable footprint — but only point-by-point, without a general frontier.
- Wang et al. (2026) demonstrate hybrid withholding attacks with explicit revenue–forking tradeoffs and propose countermeasure detectors; this is the immediate trigger for the census entry. It shows tradeoffs exist but proves no minimax bounds.
- Li et al. (2024, Sci. Rep.) analyze detectability/statistical footprints of selfish-style attacks. (verify content)
- Related tools exist in other fields: hypothesis-testing lower bounds (e.g., Chernoff/Stein exponents for sequential tests, minimax rates in changepoint detection à la Lorden/Page) and covert-communication capacity results (Bash et al.'s "square-root law"), which bound information transfer subject to a false-positive constraint. These have not been ported to the block-tree setting.

What's missing: a closed-form or asymptotic characterization of V(α, δ, τ, T), and identification of the undetectable attack set.

## What would count as solving it?

Any of the following would constitute progress; a full solution includes (1) + (2):

1. **Upper bound (impossibility):** a theorem of the form V(α, δ, τ, T) ≤ f(α, δ, τ, T) for a specified observation model — i.e., no strategy earning more than f can evade every detector with FPR ≤ δ and delay ≤ τ.
2. **Lower bound (construction):** an explicit attack strategy σ* with proof that (a) its observation distribution is within total-variation/KL distance ε(δ) of honest (so any detector inherits ≥ δ − ε(δ, T) false alarms), and (b) R(σ*, α) ≥ g(α). Matching g and f solves the problem.
3. **Exact frontier:** the Pareto boundary of (revenue, statistical distance) over all Markovian strategies, computed exactly for the standard selfish-mining state machine, with a proof of optimality over the strategy class.
4. **Simulation-backed:** an empirically calibrated frontier on a real network trace (stale rates, propagation delays) showing which known attacks fall inside the honest noise envelope.

## How to start

1. Read Wang et al. (2026) first — extract their attack model, observable statistics (fork rate, orphan rate), and detector; formalize the sequential hypothesis test. Then Li et al. (2024) for the statistical-footprint methodology.
2. Model the attacker as an MDP over the standard selfish-mining state (private chain length, public chain length, last publisher), as in Sapirshtein et al. (2016) (verify citation), but augment the objective to a Lagrangian R(σ) − λ·D_KL(P_σ ‖ P_honest) — the revenue–detectability frontier then falls out of constrained MDP theory.
3. Port covert-communication machinery (square-root law, Bash et al.) to bound the KL divergence achievable between honest and attack block-tree distributions under a fixed T; combine with the change-detection lower-bound literature (Lorden-style delay constraints) for the τ term.
4. Main obstacle: the observation process is a complicated combinatorial object (random block trees with network propagation noise), so exact likelihood ratios are intractable; expect to work with a tractable sufficient statistic (e.g., stale-block count process) and prove robustness to the reduction.

## Key references

- Wang et al., "To be detected or not: a hybrid selfish mining attack and countermeasures," Frontiers of Computer Science (2026), DOI:10.1007/s11704-026-51791-9. Constructs hybrid withholding attacks exhibiting explicit revenue–forking tradeoffs and proposes statistical countermeasures; no general minimax bounds.
- Li et al., Scientific Reports (2024), DOI:10.1038/s41598-024-55348-3. Statistical analysis of selfish-mining detectability footprints. (verify exact scope — census gives DOI only)
- Eyal & Sirer, "Majority is not enough," FC 2014, arXiv:1311.0243. Original selfish-mining attack and revenue analysis; the baseline strategy class.
- Sapirshtein, Sompolinsky, Zohar, "Optimal selfish mining strategies in bitcoin," FC 2016, arXiv:1507.06183. MDP formulation computing revenue-optimal selfish strategies; the natural framework to extend with a detectability constraint.
- Bash, Goeckel, Towsley, "Limits of reliable communication with low probability of detection on AWGN channels," IEEE JSAC 2013, arXiv:1202.6425. Square-root law for covert communication; the key analogue for bounding achievable statistical footprints.
- Lorden (1971), "Procedures for reacting to a change in distribution," Ann. Math. Statist. Classical delay-constrained changepoint-detection lower bounds; supplies the τ-side machinery.

## Warnings

- The observation model must be pinned down before proving anything: full vs. published-only views, and whether propagation delay is modeled, change the answer materially. State your assumption explicitly.
- Naïve "detect elevated fork rate" arguments ignore that honest networks have heavy-tailed stale-rate variance (propagation races); many claimed detectors have uncontrolled false positives — this is exactly the gap the problem exploits.
- Beware strategy-class smuggling: results over finite-state Markovian strategies don't cover history-dependent or adaptive attacks; note the restriction you use.
- The census date and the 2026 paper may be post-hoc or unverifiable — confirm the DOI resolves and read the actual paper before relying on its claims. (verify)

---
*`BCGT-000233` · flagged for review*

## Acceptance guidance

Any of the following would constitute progress; a full solution includes (1) + (2):

1. **Upper bound (impossibility):** a theorem of the form V(α, δ, τ, T) ≤ f(α, δ, τ, T) for a specified observation model — i.e., no strategy earning more than f can evade every detector with FPR ≤ δ and delay ≤ τ.
2. **Lower bound (construction):** an explicit attack strategy σ* with proof that (a) its observation distribution is within total-variation/KL distance ε(δ) of honest (so any detector inherits ≥ δ − ε(δ, T) false alarms), and (b) R(σ*, α) ≥ g(α). Matching g and f solves the problem.
3. **Exact frontier:** the Pareto boundary of (revenue, statistical distance) over all Markovian strategies, computed exactly for the standard selfish-mining state machine, with a proof of optimality over the strategy class.
4. **Simulation-backed:** an empirically calibrated frontier on a real network trace (stale rates, propagation delays) showing which known attacks fall inside the honest noise envelope.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/bf6bb792-0bf5-4cfd-8e91-7d6a7dfbdff8?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000233.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000233)
