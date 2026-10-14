<!-- solve-problem:138e8af0-f557-4dbc-8018-2c664fe057f9 -->
# Network-aware optimal block withholding

A [Tangle Solve problem](https://solve.tangle.tools/problems/138e8af0-f557-4dbc-8018-2c664fe057f9).

## What is the problem?

In proof-of-work blockchains, a "selfish" miner with hash fraction α can withhold mined blocks and release them selectively to orphan honest miners' blocks, earning more than its fair share. Prior work (notably Sapirshtein et al. 2015) computes *optimal* withholding policies by modeling mining as a Markov Decision Process over the attacker's private lead. But that MDP assumes a stylized network: races are won with fixed probabilities, propagation is instantaneous or governed by a single parameter γ (fraction of honest nodes the attacker reaches first), and block size / congestion play no role.

The problem: **formalize and solve the optimal withholding problem when release events actually depend on network state** — topology (who connects to whom), block size (larger blocks propagate slower), congestion (bandwidth limits, queueing at relays), and selective release (the attacker can target releases at subsets of peers). Concretely: define a stochastic model M = (topology G, block size distribution, propagation/queueing dynamics, release action space) and compute (or certify) the policy π* maximizing the attacker's discounted relative reward, or bound the profit threshold α*(M) below which no withholding strategy is profitable.

Ambiguities to note: the "right" propagation model is not specified (delayed-diffusion approximations vs. adversarial queueing vs. epidemic/rumor models on G are all candidates), and whether the attacker observes network state (full vs. partial observability) changes the problem from an MDP to a POMDP.

## Why does it matter?

Profitability thresholds like α* ≈ 0.25 (with γ-parameters) are the standard justification for claiming Bitcoin's security margin under selfish mining — but real thresholds shift with topology, block size, and congestion. A network-aware solution would let protocol designers defend parameter choices (block size limits, relay policies, peer topology) against rational attackers quantitatively, and would tell miners when withholding is actually profitable on deployed networks. It also connects to adversarial queueing and POMDP theory, where certified-optimal results are scarce.

## What's already known?

- **Sapirshtein, Sompolinsky, Zohar (2015):** MDP over private-lead states; optimal policies computed exactly via value iteration; shows optimal selfish mining becomes profitable below the commonly cited α = 1/4 (down to ~0.231 with optimal γ handling). Network reduced to a single race-win parameter γ.
- **Gervais et al. (2016):** Quantifies how propagation delay, block size, and topology affect stale-block rates and hence security, via simulation on measured topologies; establishes that network parameters matter empirically but gives no optimal-policy computation.
- **Earlier:** Eyal–Sirer (2014) introduced selfish mining with threshold 1/4 under γ-parameterized races (verify exact attribution of thresholds). Follow-ups (e.g., stubborn mining, nothing-at-stake variants) extend strategy spaces but keep abstract network models.
- **Gap:** No work computes certified optimal policies where the transition kernel itself depends on congestion/topology, and no POMDP treatment with network state as latent exists (verify — this is the open target).

## What would count as solving it?

1. **Model + optimal policy:** A well-specified stochastic model of propagation on a graph with block-size and congestion effects, and either (a) a proof that a computed withholding policy is optimal within that model (exact or ε-optimal), or (b) a certified lower bound on the profitability threshold α*(M).
2. **Robustness result:** Bounds on how far α* shifts as topology/congestion vary (e.g., "α* ∈ [a, b] for any graph with diameter ≤ d and bandwidth ≥ B").
3. **Validation:** Empirical demonstration on a measured network topology (à la Gervais et al.) that the computed policy outperforms γ-based optimal policies, with regret bounds or confidence intervals.
A referee wants: a formal model, optimality certificate or provable threshold, not just simulation heuristics.

## How to start

1. Read Sapirshtein et al. (2015) §3–4 to master the lead-state MDP and the γ-parameterization; this is your baseline abstraction.
2. Read Gervais et al. (2016) §on propagation/stale rates to pick a tractable propagation model (e.g., exponential/Gaussian delay on each edge, or a fluid queueing model) that is empirically grounded but MDP-compatible.
3. Formulate the augmented state: (private lead, network congestion level, observed release acknowledgments). Recognize this is a POMDP; consider restricting to a finite congestion-state abstraction to retain computability.
4. Try first: complete information, two-cluster topology (attacker-connected vs. rest) — generalizes γ to a controlled release decision; solve exactly, then add congestion.
Main obstacle: the action space of selective release over a graph explodes, and congestion makes transitions history-dependent (non-Markovian) unless carefully state-augmented.

## Key references

- Sapirshtein, H., Sompolinsky, Y., Zohar, A. (2015). "Optimal Selfish Mining Strategies in Bitcoin." arXiv:1507.06183. Computes optimal selfish-mining policies via MDP over the attacker's private chain lead, parameterizing network races by γ. https://arxiv.org/abs/1507.06183
- Gervais, A., Ritzdorf, H., Karame, G.O., Capkun, S. (2016). "On the Security and Performance of Proof of Work Blockchains." ACM CCS 2016, DOI:10.1145/2976749.2978341. Quantifies empirically how propagation delay, block size, and topology drive stale rates and security of PoW chains. https://doi.org/10.1145/2976749.2978341
- Eyal, I., Sirer, E.G. (2014). "Majority is not Enough: Bitcoin Mining is Vulnerable." FC 2014. Introduces the selfish-mining attack and the α = 1/4 profitability threshold under γ-race assumptions. (verify details)
- (Optional) Carlsten et al. or POMDP literature on certified policies — for the partially observable extension. (verify)

## Warnings

- Don't reinvent the γ-MDP; it's fully solved. The novelty must be in the network-dependence.
- Congestion makes propagation times depend on the entire release history — naive state augmentation breaks Markovianity; you may need fluid/large-buffer limits or state abstraction with provable error bounds.
- "Optimal" is only meaningful relative to a stated model; a referee will attack the model's realism vs. tractability trade-off. State assumptions explicitly.
- Full POMDP solvers won't scale to realistic topologies; expect to need structure (symmetry, monotonicity in lead) to get certificates.
- The census record is sparse; specific claims about what exists post-2020 beyond the two key works should be independently verified.

---
*`BCGT-000225` · flagged for review*

## Acceptance guidance

1. **Model + optimal policy:** A well-specified stochastic model of propagation on a graph with block-size and congestion effects, and either (a) a proof that a computed withholding policy is optimal within that model (exact or ε-optimal), or (b) a certified lower bound on the profitability threshold α*(M).
2. **Robustness result:** Bounds on how far α* shifts as topology/congestion vary (e.g., "α* ∈ [a, b] for any graph with diameter ≤ d and bandwidth ≥ B").
3. **Validation:** Empirical demonstration on a measured network topology (à la Gervais et al.) that the computed policy outperforms γ-based optimal policies, with regret bounds or confidence intervals.
A referee wants: a formal model, optimality certificate or provable threshold, not just simulation heuristics.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/138e8af0-f557-4dbc-8018-2c664fe057f9?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000225.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000225)
