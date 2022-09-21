---
layout: default
title: Example Notes
---

# Prelude

**Question**: Can we improve on MCMC-based methods for sampling using
Generative Flow Networks?

**Answer**: Yes, when there is structure shared between the modes in a
distribution.

## Terminology

### Terms

- **DAG**: Directed Acyclic Graph
- **GFN**: Generative Flow Network
- **MCMC**: Markov Chain Monte Carlo
- **PPO**: Proximal Policy Optimization
- **RL**: Reinforcement Learning
  {% marginnote "mn-1" "This is an example Liquid `marginnote`." %}
- **Proxy**: A function that approximates an oracle, i.e. $R(x)$, trained using
  $(x,y)$ pairs, which can incorporate (Bayesian) uncertainty
- **Active learning**: A context in which "student" and "teacher" interact during training.
- **Flow matching**: The total flow going into a state must match the total
  flow leaving the state, except for the source, $s_0$, and sink(s), $s_f$ or
  $s_T$.
  {% marginfigure "mf-1" "assets/img/side.png" "This is side image." %}
- **Assay**: Measures the composition or quality of a substance.
  {% sidenote "sn-1" "This is an example Liquid `sidenote`." %}

### Notation

- $\mathcal{S}$: The set of states, i.e. the state space.
- $\mathcal{A}$: The set of actions, i.e. the action space.
- $\mathcal{A}(s)$: The set of allowed actions in state $s$.
- $\mathcal{A}^\ast(s)$: The set of all sequences of actions allowed after
  state $s$, i.e. $\vec{a}$.
- $C: \mathcal{A}^\ast\rightarrow \mathcal{S}$: Function that maps a sequence
  of actions to a state; when the sequence is incomplete, the reward is 0.
  - When the correspondence between actions and states is **bijective**, the
    states are uniquely described by some sequence $\vec{a}$ and the
    generative process can be represented as a tree.
  - When the correspondence is **non-injective**, i.e. multiple action
    sequences define the same $x$, the generative process resembles a DAG.
- $\tilde{V}: \mathcal{S}\rightarrow\mathbb{R}^+$: maps states to their
  expected values.
- $\tau$: A trajectory, i.e. a sequences of states and actions or edges.
- $\pi(a\mid s)$: The probability of an action $a\in\mathcal A$ conditional on
  a state $s\in\mathcal{S}$.
- $\pi(s)$: The probability of visiting state $s$ when starting at $s_0$ and following $\pi(\cdot\mid\cdot)$.
- $\mathcal{X}$: A set of discrete objects.
- $R(x)$: The reward associated with _terminal_ state $x\in\mathcal{X}$.
- $F(s)$: Total flow going through state $s$; terminal states have out-flow $R(s)$.
- $F(s, a)$: The total flow through edge $(s, a)$.
- $F_\theta$: A flow parameterized by $\theta$.
- $Z=F(s_0)=F(s_f)$: The partition function, which is equivalent to all the
  flow out of the source node or all the flow into the sink node.

# Generative Flow Networks

{% maincolumn "assets/img/full-width.png" "A main column image." %}

- Definitions:
  - $f$: energy function $\rightarrow$ generative distribution.
  - Transforms a positive reward function into a generative policy that
    samples in proportion returns.
  - $\pi(x) \approx \frac{R(x)}{Z}=\frac{R(x)}{\sum_{x^{\prime} \in \mathcal{X}} R\left(x^{\prime}\right)}$
    where $x\in\mathcal{X}$
  - The _flow consistency equations_ ensure that the inflow must equal outflow
    (note that for interior nodes $R(s)=0$ and for terminal nodes
    $A(s)=\emptyset$):
    $$
    \sum_{s, a: T(s, a)=s^{\prime}} F(s, a)
    =R\left(s^{\prime}\right)+\sum_{a^{\prime}
    \in \mathcal{A}\left(s^{\prime}\right)} F\left(s^{\prime}, a^{\prime}\right)
    $$
- Propositions:

  - **Proposition 1**: Illustrates the "overcounting" problem in the non-injective case.
    - Given:
      $$
      \begin{aligned}
        s_0
        &= C(\emptyset) \\
        \pi(a\mid s)
        &= \frac{\tilde{V}(s+a)}
          {\sum_{b\in\mathcal{A}(s)}\tilde{V}(s+b)} \\
        \pi(\vec{a}=(a_1,\ldots, a_N))
        &= \Pi_{i=1}^N \pi(a_i\mid C(a_1,\ldots,a_{i-1})) \\
      \end{aligned}
      $$
    - Then:
      - The probability of a given state is equivalent to the sum of all action
        sequences leading to that state,
        $\pi=\sum_{\vec{a}_i:C(\vec{a}_i)=s}\pi(\vec{a}_i)$.
      - If $C$ is bijective, then $\pi(s)=\frac{\tilde{V}(s)}{\tilde{V}(s_0)}$
        and as a special case for terminal states $x$,
        $\pi(x)=\frac{R(x)}{\sum_{x\in\mathcal{X}}R(x)}$.
      - If $C$ is non-injective and there are $n(x)$ distinct action sequences
        $\vec{a_i}$ such that $C(\vec{a_i})=x$, then
        $\pi(x)=\frac{n(x)R(x)}{\sum_{x'\in\mathcal{X}}n(x')R(x')}$.
    - Comments:
      - In combinatorial spaces, i.e. molecules, larger molecules would be
        exponentially more likely to be sampled because there are more paths
        leading to them, which means that $\tilde{V}$ is biased, since it
        assumes the MDP's structure is a tree.
  - **Proposition 2**: Shows that $\pi(a\mid s)=\frac{F(s,a)}{F(s)}$ creates
    a generative policy where $\pi(x)\propto R(x)$ regardless of whether $C$
    is bijective or non-injective.
    - Given:
      $$
      \begin{aligned}
        \pi(a\mid s)
        &=\frac{F(s,a)}{F(s)},\text{ where }F(s,a) > 0 \\
        F(s)
        &= R(s) + \sum_{a\in\mathcal{A}(s)}F(s,a) \\
        \sum_{s, a: T(s, a)=s^{\prime}} F(s, a)
        &=R\left(s^{\prime}\right)+\sum_{a^{\prime}
        \in \mathcal{A}\left(s^{\prime}\right)} F\left(s^{\prime}, a^{\prime}\right)
      \end{aligned}
      $$
    - Then:
      - $\pi(s)=\frac{F(s)}{F(s_0)}$ for non-terminal $s$.
      - $F(s_0)=\sum_{x\in\mathcal{X}}R(x)$
      - $\pi(x)=\frac{R(x)}{\sum_{x'\in\mathcal{X}} R(x')}$ for terminal $x\in\mathcal{X}$
    - Comments:
      - In all cases, $\sum_{x\in\mathcal{X}}\pi(x)=1$ because terminal states
        are mutually exclusive.
      - In the non-injective case, $\sum_{s\in\mathcal{S}}\pi(s)\ne 1$ in
        general because internal states are not mutually exclusive.
  - **Proposition 3**: Off-policy sampling works provided the sampling policy
    $P$ has adequate support.
    - Given:
      - The exploratory sampling policy $P$ parameterized by $\theta'$ has the
        same support as the optimal $\pi$ for a consistent flow
        $F^\ast\in\mathcal{F}^\ast$, i.e. in flow equals out flow.
      - $\exists \theta: F_\theta=F^\ast$, i.e. there is a sufficiently rich family of predictors.
      - $\theta^\ast\in \arg\min_\theta\mathbb{E_{\tau\sim P(\theta')}}\left[L_\theta(\tau)\right]$,
        where $L_\theta$ is the Flow-matching Loss.
    - Then:
      - A global optimum for the expected loss generates the correct flows:
        - $\forall\tau\sim P(\theta')$:
          - $F_{\theta^\ast}=F^\ast$
          - $L_{\theta^\ast}(\tau)=0$
      - If $\pi_{\theta^\ast}(a\mid s)=\frac{F_{\theta^\ast}(s, a)}{\sum_{a'\in\mathcal{A}(s)}F_{\theta^\ast}(s,a')}$
        then $\pi_{\theta^\ast}(x)=\frac{R(x)}{Z}$.
    - Comments:
      - In general, there are an infinite number of solutions. Imagine a case
        where only two trajectories are possible,
        $\tau_1=(s_0,a_1,s_A,a_2,s_T)$ and $\tau_2=(s_0,a_3,s_B,a_4,s_T)$. Then
        $F(s_A)+F(s_B)=R(s_T)$ and the solution is any linear combination of
        the two, i.e. $F(s_A)=\alpha$ and $F(s_B)=r-\alpha$ for $\alpha\in[0,
        r]$.

- Objective Functions:

  - **Naive flow-matching loss**:
    $$
    \tilde{\mathcal{L}}_\theta(\tau)
    =\sum_{s^{\prime} \in \tau \neq s_0}\left(\sum_{s, a: T(s, a)=s^{\prime}}
    F_\theta(s, a)
    -R\left(s^{\prime}\right)
    -\sum_{a^{\prime} \in \mathcal{A}\left(s^{\prime}\right)}
    F_\theta\left(s^{\prime}, a^{\prime}\right)\right)^2
    $$
    - Issues:
      - Flow variability:
        - **Problem**: flow will be much larger for nodes near the source; the
          higher the cardinality of $\mathcal{X}$, the smaller the terminal
          flow for each terminal state $x$.
        - **Solution**: estimate flow on the log scale.
  - **Flow-matching Loss**:
    $$
    \mathcal{L}_{\theta, \epsilon}(\tau)
    =\sum_{s^{\prime} \in \tau \neq s_0}\left(\log \left[\epsilon
    +\sum_{s, a: T(s, a)=s^{\prime}} \exp F_\theta^{\log }(s, a)\right]
    -\log \left[\begin{array}{c}
    \epsilon
    +R\left(s^{\prime}\right)+\sum_{a^{\prime} \in \mathcal{A}\left(s^{\prime}\right)}
    \exp F_\theta^{\log }\left(s^{\prime}, a^{\prime}\right)
    \end{array}\right]\right)^2
    $$
    - Issues:
      - Numerical instability:
        - **Problem**: taking the logarithm of very small flows.
        - **Solution**: the hyperpameter $\epsilon$ adjusts the pressure on
          matching small vs. large flows; the larger the value, the less
          sensitive to small flows; in practice, $\epsilon\approx \min_s R(s)$.

- Details:
  - When the mapping between action sequences and states is bijective,
    generating one $x$ is like an episode in a tree-structured deterministic
    MDP where the leaves are terminal states; in this case $\tilde{V}(s)$ is
    the sum of all descendant rewards.
  - When the mapping is non-injective, methods like MaxEntRL and other
    autoregressive methods "overcount."
- Examples:
  - Molecules:
    - $\mathcal{X}$ is a collection of molecules.
    - $R(x)$ measures a chemical property of the molecule, which is a proxy for
      actual values obtained from assays.
- Controls:
  - Temperature to control exploration around modes.
  - Powers of returns, i.e. $R(x)^\beta$, also control exploration.
- Advantages:
  - Off-policy training: can use samples generated by $\pi_T$, which is not the
    same as the trained distribution, provided it has adequate support for the
    true distribution.

# Empirical Results

- Hypergrid:
  - Converges to $\pi(x)\propto R(x)$.
  - Requires less samples than MCMC and PPO under various performance metrics.
  - Recovers all the modes and does so faster than MCMC and PPO.
  - Robust to separation between modes.
  - Top-k returns are greater in an active learning setting for nearly all rounds.
- Molecule generation:
  - Generates higher reward molecules than baselines.
  - Generates more diverse candidates than baselines.
  - Top-k returns are greater in an active learning setting for nearly all rounds.

# Comparisons

### GFNs vs. RL

- RL methods approximate the best possible policy in static and stochastic
  environments, i.e. they put the probability mass on the best action in each
  state.
- GFNs approximate distributions in deterministic environments.

### GFNs vs. MCMC

- GFNs trade sampling complexity (MCMC hallmark) for training complexity;
  training this model amortizes the cost of generating samples.
- Bootstrapping can cause optimization challenge and limit performance.
- MCMC suffers from mode-mixing problem, i.e. probability deserts between high
  value modes.
- MCMC methods are expensive when they generate samples uniformly because they
  generate low value samples.
- When the modes are random, GFNs should provide no added benefit.
- **Lemma 5**:

$$
\begin{aligned}
\forall \tau=\left(s_1, \ldots, s_n\right) \in \mathcal{T}^{\text {partial }}
& \hat{P}_F(\tau):=\prod_{t=1}^{n-1} \hat{P}_F\left(s_{t+1} \mid s_t\right) \\
\forall \tau=\left(s_1, \ldots, s_n\right) \in \mathcal{T}^{\text {partial }}
& \hat{P}_B(\tau):=\prod_{t=1}^{n-1} \hat{P}_B\left(s_t \mid s_{t+1}\right) \\
\forall s \in \mathcal{S} \backslash\left\{s_f\right\} \quad \sum_{\tau \in
\mathcal{T}_{s, f}}
& \hat{P}_F(\tau)=1 \\
\forall s^{\prime} \in \mathcal{S} \backslash\left\{s_0\right\} \quad
\sum_{\tau \in \mathcal{T}_{0, s^{\prime}}}
& \hat{P}_B(\tau)=1
\end{aligned}
$$
