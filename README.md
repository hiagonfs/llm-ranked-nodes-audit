# LLM Ranked Nodes Audit

Research artifacts for the study:

**“Confidence-Guided Bayesian Network Elicitation with LLMs and Ranked Nodes.”**

The study audits a confidence-guided procedure for constructing Bayesian network conditional probability tables (CPTs) with large language models and a midpoint approximation of the Ranked Nodes Method (RNM). It examines agreement with published reference networks, the effects of persona pooling, and the provenance of errors introduced during elicitation, response processing, and CPT completion.

> **Package status:** the repository is being assembled. The directory and file names below define the target layout. Their inclusion in this README does not indicate that the corresponding artifacts have already been uploaded or verified.

## Reproducibility materials

The `online-resource/` directory is intended to contain the documentation accompanying the computational artifacts:

| Document | Scope |
|---|---|
| `prompts.md` | Complete confidence-screening, FullDist, persona, and retry instructions for both domains. |
| `methodological-details.md` | Routing rules, midpoint RNM construction and fitting, pooling, alignment, metric definitions, and smoothing. |
| `secondary-tables.md` | Complete scores, root/conditional decompositions, routing counts, and sensitivity results. |
| `response-audit.md` | Inspection protocol, response identifiers, accepted tuples, relevant excerpts, and error contributions for the audited cases. |
| `supplementary-material.pdf` | Consolidated supplementary document connecting the detailed evidence to the research questions. |

Machine-readable tables belong in `results/`; full generation records belong in `generations/` and the preserved source archives in `archives/`.

## Study design

The archived study comprises:

- 2 reference Bayesian networks: `seismic` and `electricvehicle`;
- 2 models: Qwen3.5-4B and Qwen3.5-9B;
- 4 prompting conditions per network–model pair: control and three domain-specific personas;
- 1 recorded generation (`r1`) per condition;
- 16 primary network generations;
- 8 derived persona pools: equal and configured weighted pooling for each network–model pair;
- a uniform-probability baseline for each reference network.

The eight pools are deterministic combinations of existing persona outputs. They do not add independent LLM executions. Comparisons across the archived outputs are descriptive.

The analysis addresses three questions:

1. **Reference agreement:** how do generated CPTs compare with the published references and uniform baselines across networks, models, and conditions?
2. **Persona pooling:** how do equal and configured pools change root and conditional errors relative to individual outputs?
3. **Error provenance:** what do routing decisions, completion metadata, and raw responses reveal about concentrated errors?

## Reference networks

Both networks originate from [bnRep](https://doi.org/10.1016/j.neucom.2025.129502).

| Network | Domain | Nodes | Edges | States per variable | Root / conditional CPT rows |
|---|---|---:|---:|---:|---:|
| `seismic` | Seismic risk of a reinforced-concrete building | 10 | 12 | 3 | 4 / 78 |
| `electricvehicle` | Electric-vehicle fire risk | 23 | 22 | 2 | 15 / 64 |

Generation uses the graph, variable descriptions, and state labels. The reference CPTs enter the evaluation stage. Reference exports must retain their source/version information and explicit state and parent-configuration ordering.

Agreement with these CPTs measures correspondence to the source models. It does not establish calibration against observed real-world events.

## Evaluated models

| Model | Checkpoint identifier |
|---|---|
| Qwen3.5-4B | [`Qwen/Qwen3.5-4B`](https://huggingface.co/Qwen/Qwen3.5-4B) |
| Qwen3.5-9B | [`Qwen/Qwen3.5-9B`](https://huggingface.co/Qwen/Qwen3.5-9B) |

The identifiers locate the model repositories. The loaded checkpoint commit hashes were not preserved in the archived study; requesting revision `main` does not identify an immutable historical version.

## Prompting conditions and persona pooling

| Condition | Seismic domain | Electric-vehicle domain |
|---|---|---|
| Control | No persona instruction | No persona instruction |
| P1 | Structural engineer | Electric-vehicle systems engineer |
| P2 | Seismologist | Automotive fire-safety and reliability specialist |
| P3 | Probabilistic risk analyst | Probabilistic risk analyst |

Each condition uses the same hybrid construction procedure. Persona labels denote prompting roles; their outputs are not judgments from independent human experts.

Pools combine P1–P3 and exclude the control:

| Pool or override | P1 weight | P2 weight | P3 weight |
|---|---:|---:|---:|
| Equal pool | 1/3 | 1/3 | 1/3 |
| Configured weighted pool: default | 0.50 | 0.30 | 0.20 |
| Configured weighted pool: seismic `SeismicHazard` | 0.15 | 0.70 | 0.15 |
| Configured weighted pool: seismic `Fragility` | 0.70 | 0.10 | 0.20 |

The configured weights express assumed domain relevance. They were not calibrated against reference CPT accuracy. Pool records should identify their constituent CPTs and the weights applied to each node.

## Execution settings

The manuscript reports the following settings for the archived generations:

| Setting | Recorded configuration |
|---|---|
| Hardware | NVIDIA Tesla T4 |
| Quantization | 4-bit NF4 with double quantization |
| Compute precision | FP16 |
| Transformers | `5.19.0.dev0` |
| Base seed | 42 |
| Repetitions | One generation per network–model–condition combination |
| Thinking mode | Configured as disabled |
| Confidence screening | Greedy decoding; temperature 0; maximum 1,536 output tokens |
| FullDist | Temperature 0.1; top-p 1.0; maximum 2,048 output tokens, increased to 4,096 on retry |
| Confidence threshold | 0.80 |
| RNM default spread | 0.16 |
| RNM spread bounds | [0.03, 0.45] |
| Weight regularization | 0.02 toward equal parent weights |
| Optimization | L-BFGS-B; up to 500 iterations |

Some inherited decoding defaults and exact loaded model commits were not recorded. Generation and evaluation dependencies should be documented separately from recoverable execution records. A newly installed environment should be identified as a new environment rather than presented as the exact historical setup.

## Repository layout

The following layout guides package assembly:

| Path | Intended contents |
|---|---|
| `archives/` | Original generation and evaluation ZIPs for the four network–model pairs, with an inventory and checksums. |
| `networks/` | Graphs, variable descriptions, ordered states, and generation input specifications. |
| `reference/` | bnRep reference CPT exports, source/version information, alignment metadata, and reference parent-configuration weights. |
| `notebooks/generation/` | Generation notebooks corresponding to the archived experiments. |
| `notebooks/evaluation/` | Evaluation and numerical-verification notebooks. |
| `prompts/` | Exported prompt templates, domain personas, and retry instructions. |
| `configs/` | Model, decoding, RNM, routing, pooling, and seed configurations. |
| `generations/` | CPTs, raw responses, accepted/rejected attempts, routing decisions, anchors, and fitting metadata. |
| `scripts/` | Code for extraction, pooling, alignment, evaluation, diagnostics, verification, and figure generation. |
| `results/` | Consolidated scores, derived pool CPTs, row-level evaluations, audit summaries, and sensitivity tables. |
| `figures/` | Figures used in the manuscript and supplement. |
| `online-resource/` | Detailed documentation, audit evidence, supplementary tables, and supplementary PDF. |

The archive inventory should map each source file to its network, model, function, and experimental version. Later notebook revisions should be distinguished from the versions that produced the reported outputs.

## Computational workflow

### 1. Network preparation

Prepare generation specifications from the network structures, variable descriptions, and state labels. Keep the corresponding reference CPT exports identifiable for subsequent evaluation.

### 2. Confidence-guided CPT generation

For each network–model pair, run control and P1–P3 using the corresponding generation notebook:

- Roots receive a direct FullDist request.
- Conditional configurations receive a self-reported confidence score.
- Configurations at or above 0.80 receive a direct FullDist request.
- Successfully parsed direct rows become final CPT rows and fitting anchors.
- Remaining conditional rows use the midpoint RNM approximation, fitted when anchors exist and completed with defaults otherwise.

Preserve the raw response, parsing decision, accepted vector, and retry history. Influence directions and fitting parameters must remain associated with the node that used them.

### 3. Persona pooling

Construct equal and configured weighted pools from P1–P3. Apply the seismic node overrides when reconstructing the configured pool. Pooling requires no additional LLM calls.

### 4. Alignment and evaluation

Align generated and reference probabilities by node name, child-state name, and every parent name/state pair. Compute the same metrics for all 24 outputs and the corresponding uniform baselines.

### 5. Diagnostic analysis

Produce route counts, root/conditional decompositions, smoothing sensitivity, boundary-probability counts, and response-audit records. Generate manuscript figures from the consolidated full-precision results.

## Evaluation

The evaluation uses:

- Mean absolute error (MAE).
- Root mean squared error (RMSE).
- Total variation distance (TV).
- Mean CPT Kullback–Leibler divergence, equally weighting CPT rows.
- Reference-weighted BN KL score, weighting each conditional row by its parent-configuration probability under the reference network.

Lower values indicate closer agreement with the reference. KL scores use base-2 logarithms and are reported in bits.

The default KL smoothing constant is `1e-8`. Sensitivity analysis uses `1e-10`, `1e-8`, `1e-6`, and `1e-4`, holding the reference parent-configuration weights fixed. With this smoothing procedure, the weighted score is not generally the exact joint KL between two fully smoothed networks.

For the uniform baseline, each child state receives probability `1/K`, where `K` is the node's number of states. The same alignment and evaluation settings apply to generated outputs and baselines.

## Diagnostic outputs

The following tables provide the target machine-readable interface for the package:

| File in `results/` | Contents |
|---|---|
| `scores_all_outputs.csv` | All five metrics for the 24 outputs and network-specific uniform baselines. |
| `aligned_probabilities.csv` | Reference and estimated probabilities with network, model, condition, run, node, child state, and parent configuration. |
| `root_conditional_errors.csv` | Root and conditional error components for control, each persona, and both pools. |
| `row_provenance.csv` | Construction history of the 16 primary generations: root FullDist, conditional FullDist, anchor-fitted RNM, or unanchored RNM. |
| `routing_summary.csv` | Confidence-screening coverage, route counts, and anchor counts by network, model, condition, and node. |
| `smoothing_sensitivity.csv` | Recomputed KL scores and uniform-baseline comparisons at each epsilon. |
| `boundary_probabilities.csv` | Counts of zero/one probabilities before smoothing, including individual outputs and pools. |
| `audit_cases.csv` | Audited response identifiers, classification, accepted tuples, and links to the relevant source records. |
| `verification_report.csv` | Archived-versus-recomputed comparisons, numerical differences, and the tolerances used. |
| `paper_artifact_map.csv` | Mapping of manuscript/supplement figures and tables to source data, analysis code, and generated outputs. |

Raw responses should remain available in their original textual form. Normalized CPTs alone cannot establish whether an accepted tuple agrees with the surrounding response.

The detailed response audit described in the manuscript targets 27 accepted electric-vehicle FullDist responses containing zero or one. Its documentation should include the root case reported to account for 74.3% of one configuration's reference-weighted error: node and states, response excerpt, accepted/reference vectors, contribution in bits, and percentage denominator.

## Recomputing archived results

The primary reproduction route evaluates the preserved outputs without requesting new LLM responses:

1. Obtain the generation and evaluation archives associated with the manuscript version.
2. Recover the generated CPTs, matching references, configuration records, and row provenance.
3. Reconstruct both persona pools and the uniform baselines.
4. Run the evaluation and verification procedures using the recorded alignment and smoothing rules.
5. Compare archived and recomputed results and record numerical differences.
6. Regenerate the consolidated tables, diagnostic outputs, and figures.

The package's notebooks and scripts should document their inputs, environment, execution order, and output paths before the package is released as reproducible. This README does not prescribe commands for scripts that have not yet been supplied.

Generating new LLM outputs is a separate execution route. It requires the model dependencies and suitable inference hardware, and produces a new experimental realization. The archived seeds do not guarantee identical regeneration across model revisions, software environments, or hardware.

## Interpretation and versioning

- One generation per condition supports descriptive comparisons of the archived outputs; it does not estimate repeated-run stability.
- CPT rows share prompts, node parameters, and network context. They are not independent experimental replications.
- The confidence score expresses self-reported familiarity for elicitation, not an empirically calibrated chance of a correct probability estimate.
- The completion implementation uses state midpoints and a weighted-mean truncated-normal construction. Results concern this approximation rather than every RNM formulation.
- The archived design does not contain matched direct-only, RNM-only, and hybrid outputs for every row.
- Route error shares must be interpreted together with route counts and reference weights.

Preserve a fixed package version corresponding to the manuscript. Extraction fixes, new model generations, changed thresholds, and revised completion procedures should be identified as subsequent versions, with an explicit record of which outputs and analyses changed.

## Package assembly checklist

- [ ] Upload the original generation and evaluation archives and record their checksums.
- [ ] Add the notebooks associated with the reported runs and distinguish later revisions.
- [ ] Export network specifications, references, prompts, configurations, and raw response records.
- [ ] Add evaluation, pooling, diagnostic, and verification code with execution instructions.
- [ ] Export the complete result tables and reproduce the manuscript figures.
- [ ] Add the supplementary documentation and the detailed response-audit cases.
- [ ] Document recoverable dependencies and limitations of historical regeneration.
- [ ] Add the license texts and third-party attribution notices.
- [ ] Verify the paper-to-artifact mapping and freeze the manuscript-associated release.

## Licensing

The intended licensing arrangement is:

| Material | License |
|---|---|
| Original project code, scripts, and notebook code | [MIT](https://opensource.org/license/mit) |
| Original documentation, figures, and authored supplementary material | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| Imported networks, reference data, and other third-party material | Their applicable original licenses and attribution requirements |

Include the applicable license texts and identify their scope when assembling the package. Retain source attribution for bnRep and the publications underlying the reference networks.
