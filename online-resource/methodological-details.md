# Methodological Details

This supplement describes the experimental design, confidence gate, Ranked Nodes Model (RNM), persona pooling, and evaluation procedure used in *Confidence-Guided Bayesian Network Elicitation with LLMs and Ranked Nodes*. The task is to elicit complete conditional probability tables (CPTs) for a fixed Bayesian network using textual domain descriptions and a pretrained language model, without supplying the reference probabilities during generation.

## 1. Experimental design

The study evaluates two Bayesian networks from bnRep with two language models: **Qwen3.5-4B** and **Qwen3.5-9B**. The directed acyclic graph, variable definitions, state labels, and state ordering are specified before elicitation. Network structure is held fixed; the procedure estimates numerical parameters.

| Network | Nodes | Edges | Root nodes | Conditional nodes | States per node | Root rows | Conditional rows | Total CPT rows | Probability entries |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| seismic | 10 | 12 | 4 | 6 | 3 | 4 | 78 | 82 | 246 |
| electricvehicle | 23 | 22 | 15 | 8 | 2 | 15 | 64 | 79 | 158 |

A CPT row is one child-state distribution for a particular configuration of the parents. A root node contributes one unconditional row. For node $X_i$ with parent set $\mathrm{Pa}_i$, the number of rows is

$$
R_i=\prod_{X_j\in\mathrm{Pa}_i}K_j,
$$

where $K_j$ is the number of states of parent $X_j$ and the empty product is one. Define $N=\sum_i R_i$ as the number of rows and $M=\sum_i R_iK_i$ as the number of scalar probabilities in a complete network.

Each network–model pair has four elicitation conditions: control and three domain personas. Each condition has **one primary generation**. Two additional outputs are obtained by pooling the three persona CPTs, giving six evaluated outputs per network–model pair, **16 primary generations and eight pooled outputs**, or 24 outputs overall. Pooling uses no additional model calls. A uniform CPT baseline is evaluated separately for each network.

| Condition | seismic | electricvehicle |
|---|---|---|
| Control | Empty system instruction | Empty system instruction |
| P1 | Structural engineer | Electric vehicle systems engineer |
| P2 | Seismologist | Automotive fire safety and reliability specialist |
| P3 | Probabilistic risk analyst | Probabilistic risk analyst |

Persona roles are prompting conditions, not independent human experts. The risk-analyst instructions are adapted to each domain. Within a domain, both model sizes receive the same descriptions and persona instructions.

## 2. Information supplied during elicitation

The generation procedure receives the graph and qualitative variable descriptions. A local prompt describes the target node and its immediate parents, including available state meanings, scale type, temporal context, measurement context, and modeling notes. It does not receive the reference CPTs or an observed-case dataset. Reference probabilities are accessed in the subsequent evaluation stage.

Every request uses a fresh conversational context. Control retains the same user-prompt template as the persona conditions, with an empty system instruction. Each persona supplies a role, specialty, and additional domain instructions through the system message.

Child-state order is explicit in the probability question. Parent configurations are enumerated as the Cartesian product of the ordered parent-state lists; the rightmost parent changes fastest. The numerical tuple is interpreted in the requested child-state order.

All conditional nodes and their parents are explicitly marked as ordinal in the experimental inputs and therefore eligible for RNM. For electricvehicle, the binary order is `no < yes`, interpreted as absence to occurrence. This is a modeling assumption imposed for ordinal completion. The electric-vehicle descriptions do not fix an exposure horizon, so the elicited priors should not be interpreted as calibrated event rates per vehicle-year or per trip.

A separate preliminary query asks each model about familiarity with the named benchmark. It is run once per network–model pair before showing the graph and descriptions. Its answer is not passed to later elicitation requests and does not affect routing. This diagnostic is a self-report; it cannot establish that the model had no pretraining exposure to the benchmark.

## 3. Model and generation settings

Both model sizes use the same elicitation settings and RNM hyperparameters.

| Setting | Value |
|---|---|
| Model checkpoints | `Qwen/Qwen3.5-4B`; `Qwen/Qwen3.5-9B` |
| Hardware | NVIDIA Tesla T4 |
| Quantization | 4-bit NF4 with double quantization |
| Quantized computation dtype | FP16 |
| Transformers version | 5.19.0.dev0 |
| Requested model revision | `main` |
| Thinking mode | Configured as disabled |
| Base random seed | 42 |
| Primary generations per condition | 1 |
| Confidence threshold | 0.80 |

| Request type | Decoding | Temperature | Top-p | Maximum new tokens | Maximum parsing attempts, including the first |
|---|---|---:|---|---:|---:|
| Confidence screen | Greedy | 0.0 | Not applied | 1,536 | 4 |
| FullDist, initial attempt | Sampling | 0.1 | 1.0 | 2,048 | 5 total |
| FullDist, subsequent attempts | Sampling | 0.1 | 1.0 | 4,096 | Within the same five-attempt limit |

The preliminary familiarity query uses greedy decoding. Parameters such as top-k are not explicitly overridden by the generation wrapper and depend on the loaded model configuration. Disabling thinking mode in the chat template does not guarantee that the returned text contains no reasoning: the FullDist prompt explicitly requests reasoning followed by a final numerical tuple.

Request-specific seeds are derived from the base seed and a SHA-256 hash of the user prompt, repetition index, parsing-attempt index, row-retry index, and generation-error attempt index. The first eight hexadecimal hash digits are converted to an integer, added to 42, and reduced modulo $2^{32}$. The system instruction is excluded from this seed material, allowing an otherwise identical question to receive the same seed across persona conditions. PyTorch and CUDA seeds are set before generation.

The model wrapper separately allows up to five execution attempts per request, including the first. This mechanism handles execution errors and is distinct from retries caused by an unparseable response. The call counts in Section 5 refer to the recorded elicitation requests.

The loaded immutable model commit was not retained in the evaluated runs. Consequently, the requested revision `main`, a fixed seed, and the reported software version do not guarantee bitwise reproduction on a later installation.

## 4. Confidence screening and row selection

### 4.1. Screening output

For each eligible conditional node, one screening request covers all parent configurations. It asks for:

1. A self-reported confidence $c_{ir}\in[0,1]$ for estimating each CPT row $r$ of node $i$.
2. A qualitative influence direction for each parent: `positive`, `negative`, or `uncertain`, accompanied by a confidence value.

Row confidence expresses the model's claimed knowledge for probability elicitation. It is distinct from an event probability and is not calibrated against reference error. The confidence parser requires every expected row index and parent exactly once, valid confidence values, and a permitted direction label.

The numerical confidence attached to a parent direction is recorded but does not determine the RNM parent weight. In the default direction mapping, `negative` maps to a negative influence; both `positive` and `uncertain` map to a positive influence. Uncertain directions are therefore resolved deterministically rather than integrated as uncertainty over signs.

### 4.2. Routing rule

For eligible conditional node $i$, define its selected row set using the fixed threshold $\tau=0.80$:

$$
\mathcal{A}_i=\{r:c_{ir}\geq\tau\}.
$$

Each selected row receives a FullDist request for the complete child-state distribution. The accepted distributions serve as fitting anchors and remain unchanged in the final CPT. RNM supplies the other rows:

$$
\widehat{\mathbf p}_{ir}=
\begin{cases}
\mathbf a_{ir}, & r\in\mathcal{A}_i,\\
\mathbf q_{ir}(\mathbf w_i,\sigma_i,\mathbf d_i), & r\notin\mathcal{A}_i.
\end{cases}
$$

Root nodes bypass screening and receive one FullDist request for their unconditional distribution. The implementation also supports FullDist for every row of a node whose child or parents lack explicit ordinal metadata. That nonordinal branch is not used in these two networks. No separate-state elicitation is performed.

The procedure for one eligible conditional node is: screen all configurations, elicit the selected distributions, fit RNM to those anchors when present, retain the direct distributions, and complete the remaining rows. Neither the confidence gate nor the RNM objective consults reference probabilities.

## 5. FullDist parsing, retries, and accepted outputs

FullDist asks for reasoning followed by a final tuple in the specified state order. Numerical acceptance requires a distribution of the expected length, finite entries in $[0,1]$, and a positive total. Accepted entries are normalized by their sum. Exact zeros and ones are retained at this stage; KL smoothing is applied only during evaluation.

Retrying seeks a parsable distribution and does not use agreement with the reference as an acceptance criterion. In the seismic runs, retries repeat the original prompt. In the electric-vehicle runs, second attempts append a clarification that requests a subjective prior, acknowledges the absence of empirical frequencies, and requires a final numerical tuple. Every electric-vehicle request was completed within two parsing attempts. The exact wording is provided with the [prompt templates](prompts.md).

The generation implementation accompanying this supplement repeats the base prompt on a parsing retry; reproducing the evaluated electric-vehicle procedure also requires the domain-specific clarification. Its final-answer extractor prioritizes explicit conclusion markers, a standalone final tuple, or state-labeled final probabilities. These extraction rules should not be assumed to reproduce every historical parser decision: the evaluated outputs include an electric-vehicle root distribution extracted from a tuple that the surrounding response rejected as an estimate. Numerical validity alone does not establish semantic endorsement by the model. Reprocessing responses with stricter extraction rules constitutes a new analysis.

| Network | Model | Confidence requests | Accepted FullDist rows | Additional FullDist attempts | Total FullDist requests | Highest FullDist parsing attempt |
|---|---|---:|---:|---:|---:|---:|
| seismic | Qwen3.5-4B | 24 | 19 | 5 | 24 | 3 |
| seismic | Qwen3.5-9B | 24 | 69 | 1 | 70 | 2 |
| electricvehicle | Qwen3.5-4B | 32 | 62 | 21 | 83 | 2 |
| electricvehicle | Qwen3.5-9B | 32 | 122 | 12 | 134 | 2 |
| **Total** | | **112** | **272** | **39** | **311** | |

All 112 confidence requests were accepted on their first parsing attempt. The 272 accepted FullDist distributions comprise 152 root rows and 120 conditional rows. The experiment contains **423 generation requests**, plus four preliminary familiarity queries, giving **427 recorded model requests** overall. Parsing retries are attempts within a generation, not independent experimental repetitions.

No final row in the evaluated outputs is labeled as a uniform error fallback. An RNM distribution computed without anchors is distinct from such a fallback, even when its parameters are defaults.

## 6. Ranked Nodes Model

### 6.1. Ordinal encoding and aggregation

For a parent with $K_j$ ordered states, state index $s\in\{0,\ldots,K_j-1\}$ is represented by its interval midpoint:

$$
z_j(s)=\frac{s+\tfrac12}{K_j}.
$$

For parent $j$ and configuration $r$, the signed input is

$$
x_{jr}=
\begin{cases}
z_j(s_{jr}), & d_j=+1,\\
1-z_j(s_{jr}), & d_j=-1.
\end{cases}
$$

The latent location is the weighted mean

$$
\mu_{ir}=\sum_{j=1}^{m_i}w_{ij}x_{jr},
\qquad w_{ij}\geq0,\qquad\sum_{j=1}^{m_i}w_{ij}=1,
$$

where $m_i$ is the number of parents. Parent weights, signs, and dispersion are shared across the rows of a node.

### 6.2. Discretizing the truncated normal

For a child with $K_i$ ordered states, define boundaries $b_k=k/K_i$, for $k=0,\ldots,K_i$. RNM integrates a normal distribution with location $\mu_{ir}$ and standard deviation $\sigma_i$, truncated to $[0,1]$:

$$
q_{ir,k}=
\frac{
\Phi\!\left(\frac{b_{k+1}-\mu_{ir}}{\sigma_i}\right)
-\Phi\!\left(\frac{b_k-\mu_{ir}}{\sigma_i}\right)
}{
\Phi\!\left(\frac{1-\mu_{ir}}{\sigma_i}\right)
-\Phi\!\left(\frac{-\mu_{ir}}{\sigma_i}\right)
},\qquad k=0,\ldots,K_i-1.
$$

Here $\Phi$ is the standard normal cumulative distribution function. The implementation clips $\mu$ to $[0,1]$ and $\sigma$ to its permitted interval, floors the resulting probabilities at $10^{-12}$, and renormalizes. If the truncation denominator is nonfinite or no larger than $10^{-12}$, it returns a uniform distribution as a numerical safeguard.

The weighted-mean form constrains the CPTs that RNM can express. Marking binary event variables as ordinal does not establish that their interactions, necessity relations, or fault mechanisms follow this aggregation rule.

### 6.3. Fitting to anchors

When $\mathcal{A}_i$ is nonempty, RNM is fitted to the accepted FullDist distributions using confidence-weighted squared Hellinger distance:

$$
H^2(\mathbf a,\mathbf q)=\frac12\sum_k(\sqrt{a_k}-\sqrt{q_k})^2,
$$

$$
L_i=
\sum_{r\in\mathcal{A}_i}
\frac{c_{ir}}{\sum_{u\in\mathcal{A}_i}c_{iu}}
H^2\!\left(\mathbf a_{ir},\mathbf q_{ir}\right)
+\lambda\sum_{j=1}^{m_i}\left(w_{ij}-\frac1{m_i}\right)^2.
$$

The regularizer favors equal parent weights. The anchors are model-generated estimates, so fitting measures consistency with those estimates rather than calibration to the reference network.

| RNM setting | Value |
|---|---|
| Default and initial standard deviation | $\sigma_0=0.16$ |
| Standard-deviation bounds | $[0.03,0.45]$ |
| Weight regularization | $\lambda=0.02$ |
| Weight parameterization | Softmax of one logit per parent |
| Logit bounds | $[-6,6]$ |
| Initial logits | All zero, producing equal weights |
| Dispersion parameterization | Optimize $\log\sigma$ within the log-transformed bounds |
| Optimizer | L-BFGS-B |
| Maximum iterations | 500 per candidate direction pattern |
| Numerical probability floor | $10^{-12}$ |

Direction signs are taken from the confidence screen unless there are at least $\max(2,m_i+1)$ anchors and at most six parents. When both conditions hold, all $2^{m_i}$ sign combinations are evaluated, fitting weights and dispersion separately for each. The candidate with the smallest regularized loss is retained; an improvement must exceed $10^{-12}$ to replace the current candidate.

An optimizer warning is recorded with the returned parameters; it does not automatically trigger replacement by default parameters. All 28 anchor-based node fits in the evaluated outputs report successful optimization. Of these, 18 have every row directly elicited, so their fitted parameters supply no completed rows. The remaining ten fits supply 32 RNM rows.

### 6.4. Nodes without anchors

When no row reaches the confidence threshold, no optimization is performed. The node uses

$$
w_{ij}=\frac1{m_i},\qquad\sigma_i=0.16,
$$

with directions obtained from the screen using the sign mapping in Section 4. The resulting CPT depends on parent states and qualitative directions; it is generally nonuniform. Nevertheless, its numerical shape is determined by the default RNM assumptions rather than fitted probability anchors.

### 6.5. Realized row provenance

The following counts aggregate the four primary conditions within each network–model pair. Pooled outputs are excluded because they combine constituent rows.

| Network | Model | Root FullDist | Conditional FullDist | RNM with anchors | RNM without anchors |
|---|---|---:|---:|---:|---:|
| seismic | Qwen3.5-4B | 16 | 3 | 0 | 309 |
| seismic | Qwen3.5-9B | 16 | 53 | 4 | 255 |
| electricvehicle | Qwen3.5-4B | 60 | 2 | 6 | 248 |
| electricvehicle | Qwen3.5-9B | 60 | 62 | 22 | 172 |
| **Total** | | **152** | **120** | **32** | **984** |

Among the **1,136 conditional rows**, 10.6% are directly elicited, 2.8% are completed by anchor-fitted RNM, and 86.6% use RNM without anchors. This distinction is necessary when interpreting the hybrid method: the presence of an RNM fitting component does not mean that most completed rows were fitted to elicited probabilities.

## 7. Persona pooling

Pooling is performed separately within each network–model pair, after all three persona generations are complete. For aligned row distributions $\widehat{\mathbf p}^{(a)}_{ir}$ from personas $a\in\{1,2,3\}$, the linear pool is

$$
\widehat p^{\mathrm{pool}}_{ir,k}
=\sum_{a=1}^{3}\alpha_{ia}\widehat p^{(a)}_{ir,k},
\qquad\alpha_{ia}\geq0,\qquad\sum_a\alpha_{ia}=1.
$$

The control condition is excluded. Equal pooling uses $\alpha_{ia}=1/3$. Configured pooling uses the weights below, normalized at each node and held constant across that node's rows and child states.

| Network and nodes | P1 | P2 | P3 |
|---|---:|---:|---:|
| seismic, except the overrides below | 0.50 | 0.30 | 0.20 |
| seismic, SeismicHazard | 0.15 | 0.70 | 0.15 |
| seismic, Fragility | 0.70 | 0.10 | 0.20 |
| electricvehicle, all nodes | 0.50 | 0.30 | 0.20 |

These weights encode presumed domain relevance. They are not derived from measured persona accuracy or optimized against the reference CPTs by the pooling procedure. Pooling preserves row normalization and the fixed graph. It combines local conditional distributions; the joint distribution obtained by multiplying the pooled CPTs is generally different from a mixture of the three complete joint distributions.

A linear pool can soften extreme estimates when its constituents disagree, but it preserves a zero shared by all constituents. It does not guarantee improvement over the best persona, the control, or the uniform baseline. A pooled row retains its constituent provenance rather than receiving a single direct-elicitation or RNM label.

## 8. Reference alignment and evaluation

Evaluation runs on CPU without loading the language model or generating additional estimates. The reference is the bnRep parameterization of the corresponding network. Scores measure agreement with that parameterization; they do not establish empirical validity for a deployment population.

Before comparing probabilities, evaluation requires identical node sets, edge sets, and state-label sets. Predicted tables are aligned to the reference by **node name, child-state label, and each named parent-state assignment**. Parent-axis and state-axis permutations are handled together. Positional matching of flattened arrays is insufficient.

This is consequential for electricvehicle: generation uses `no, yes`, whereas the reference uses `yes, no` for all 23 nodes. Evaluation permutes the relevant axes in memory before comparison. It does not reinterpret a `no` probability as a `yes` probability or modify the elicited estimates. The reference parent marginals used as weights are computed by exact variable elimination in the reference network.

## 9. Primary metrics

Let $\mathbf p_{ir}$ be an aligned reference row and $\widehat{\mathbf p}_{ir}$ the corresponding estimated row. All five primary error measures are better when smaller. KL uses base-2 logarithms and is reported in bits.

### 9.1. Smoothing and row KL

For KL calculations, each row is first normalized to unit sum. With normalized entries $p^\circ_{ir,k}$ and $\widehat p^\circ_{ir,k}$, define

$$
\widetilde p_{ir,k}
=\frac{p^\circ_{ir,k}+\epsilon}{1+K_i\epsilon},
\qquad
\widetilde{\widehat p}_{ir,k}
=\frac{\widehat p^\circ_{ir,k}+\epsilon}{1+K_i\epsilon},
\qquad\epsilon=10^{-8}.
$$

The row divergence is

$$
d_{ir}=\sum_{k=0}^{K_i-1}\widetilde p_{ir,k}
\log_2\frac{\widetilde p_{ir,k}}{\widetilde{\widehat p}_{ir,k}}.
$$

Smoothing is symmetric in its application to both input rows; the divergence itself remains directional, from reference to estimate. RNM's $10^{-12}$ probability floor and evaluation's $10^{-8}$ smoothing serve different purposes.

### 9.2. Mean CPT KL

$$
\mathrm{CPT\ KL}_{\mathrm{mean}}
=\frac1N\sum_i\sum_{r=1}^{R_i}d_{ir}.
$$

Every row has equal weight, including roots. This is a mean over rows, not an equally weighted mean of node-level means. Nodes with more parent configurations contribute more rows.

### 9.3. Reference-weighted BN KL score

Define $\omega_{ir}=P_{\mathrm{ref}}(\mathrm{Pa}_i=r)$, with $\omega_{ir}=1$ for a root. The reported score is

$$
\mathrm{BN\ KL}_{\mathrm{weighted}}
=\sum_i\sum_{r=1}^{R_i}\omega_{ir}d_{ir}.
$$

The weighting emphasizes configurations that are more probable under the reference network. The parent weights sum to one within each node; the final score sums the node contributions without dividing by the number of nodes.

For unsmoothed distributions factorizing over the same graph, the corresponding expression is the exact decomposition of joint KL. Here the local KL terms use smoothed rows while the weights remain those of the original reference network. The implemented quantity is therefore a **reference-weighted sum of smoothed local divergences**, rather than the exact joint KL between two fully smoothed networks. Its raw magnitude also depends on network size and should not be used alone to rank the difficulty of different networks.

### 9.4. Mean absolute error and root mean squared error

Scalar errors use aligned probability values before KL smoothing:

$$
\mathrm{MAE}
=\frac1M\sum_i\sum_{r=1}^{R_i}\sum_{k=0}^{K_i-1}
|p_{ir,k}-\widehat p_{ir,k}|,
$$

$$
\mathrm{RMSE}
=\sqrt{\frac1M\sum_i\sum_{r=1}^{R_i}\sum_{k=0}^{K_i-1}
(p_{ir,k}-\widehat p_{ir,k})^2}.
$$

These measures weight individual probability entries equally. They do not use reference parent-marginal weights.

### 9.5. Mean total variation distance

$$
\mathrm{TV}_{ir}=\frac12\sum_{k=0}^{K_i-1}
|p_{ir,k}-\widehat p_{ir,k}|,
\qquad
\mathrm{TV}_{\mathrm{mean}}=\frac1N\sum_i\sum_{r=1}^{R_i}\mathrm{TV}_{ir}.
$$

TV measures row-level discrepancy and receives no epsilon smoothing. Since each network has the same cardinality $K$ at every node, $\mathrm{TV}_{\mathrm{mean}}=(K/2)\mathrm{MAE}$. Thus mean TV equals MAE for electricvehicle and equals $1.5\times\mathrm{MAE}$ for seismic; these are not independent findings within either network.

### 9.6. Uniform baseline and smoothing sensitivity

The uniform baseline assigns $1/K_i$ to every child state in every row while preserving the graph and state labels. It uses the same alignment, KL smoothing, and reference parent-marginal weights as the elicited outputs.

The epsilon-sensitivity analysis repeats KL evaluation for $\epsilon\in\{10^{-10},10^{-8},10^{-6},10^{-4}\}$ while holding elicited probabilities, pooling weights, and reference parent marginals fixed. This tests the effect of smoothing, particularly near zero probabilities; it does not repeat elicitation or alter confidence routing.

Additional summaries include median and upper-quantile row errors, maximum absolute error, squared error, Hellinger distance, Jensen–Shannon divergence, and fractions of scalar probabilities within specified error tolerances. These supplement the five primary metrics. The fitting objective uses **squared** Hellinger distance, whereas the supplementary evaluation summary reports Hellinger distance itself.

## 10. Error attribution

For each primary generation, rows are assigned to four mutually exclusive sources: root FullDist, conditional FullDist, RNM with anchors, and RNM without anchors. The last two categories are distinguished using the node's anchor count.

For source $s$, its contribution to the reference-weighted KL score is

$$
E_s=\sum_{(i,r):\rho_{ir}=s}\omega_{ir}d_{ir},
\qquad
S_s=100\frac{E_s}{\sum_tE_t}.
$$

The percentage $S_s$ is a share of total weighted error. It differs from the fraction of rows assigned to that source and from its mean error per row. If total error is zero, percentage shares are undefined.

Root and conditional contributions can also be summed separately; together they equal the full weighted score. Such decompositions show where error accumulates. They do not establish a causal performance advantage of one route, because direct elicitation and RNM operate on different rows selected by the confidence gate.

## 11. Numerical consistency and interpretation

The five primary metrics were recomputed for all 24 outputs from the aligned reference and estimated probabilities, using the recorded reference parent-marginal weights. The maximum absolute discrepancy from the reported scores was below $5\times10^{-10}$. Both pooling schemes agree with their constituent weighted sums to within $2\times10^{-16}$. All 1,016 RNM-completed rows were also reconstructed from their stored weights, directions, dispersion, and parent states, matching the recorded values at the available floating-point precision. These checks establish computational consistency, not semantic correctness of accepted model responses or empirical calibration.

The experimental unit is a complete generation for a network–model–condition combination. Rows, parsing attempts, personas, and deterministic pools do not provide independent repeated generations. With one primary generation per condition, the reported comparisons are descriptive and do not estimate run-to-run uncertainty or support significance tests across random seeds.

The study evaluates reference-CPT agreement under a fixed graph and an imposed ordinal representation. It does not include matched FullDist-only and RNM-only counterfactuals, observed-data validation, posterior-query validation, or a human-expert elicitation comparison. Accordingly, lower error for a pool or a hybrid output should be interpreted within the evaluated networks, models, prompting conditions, and metric definitions.
