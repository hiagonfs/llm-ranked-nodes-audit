# LLM Ranked Nodes Audit

Research artifacts for the study:

**“Confidence-Guided Bayesian Network Elicitation with LLMs and Ranked Nodes.”**

The study audits a confidence-guided procedure for constructing Bayesian network conditional probability tables (CPTs) with large language models and a midpoint approximation of the Ranked Nodes Method (RNM). It examines agreement with published reference networks, the effects of persona pooling, and the provenance of errors introduced during elicitation, response processing, and CPT completion.

## Reproducibility materials

The supplementary documentation is organized into four Markdown documents in `online-resource/`:

| Document | Scope |
| --- | --- |
| [prompts.md](online-resource/prompts.md) | Confidence-screening and FullDist templates, domain personas, retry instructions, and complete prompt examples. |
| [methodological-details.md](online-resource/methodological-details.md) | Experimental settings, confidence routing, midpoint RNM construction and fitting, pooling, alignment, and metric definitions. |
| [secondary-tables.md](online-resource/secondary-tables.md) | Complete scores, supplementary metrics, root/conditional decompositions, routing counts, node contributions, and smoothing sensitivity. |
| [response-audit.md](online-resource/response-audit.md) | Inspection protocol and 27 case records, with original excerpts, accepted/reference vectors, generation parameters, classifications, and error contributions. |

Generation and evaluation ZIPs, network specifications, and persona inputs are grouped in `data/`. Scripts and notebooks are grouped in `scripts/`.

## Repository layout

The repository uses three directories:

| Directory | Contents |
| --- | --- |
| `data/` | Generation and evaluation ZIPs for each network–model pair, together with network and persona input specifications. |
| `scripts/` | Generation and evaluation notebooks and supporting Python scripts. |
| `online-resource/` | The four linked Markdown documents containing prompts, methodological details, supplementary tables, and the response audit. |

`README.md` and `LICENSE` are kept at the repository root.

Each network–model pair has two ZIP packages: **generation results** and **evaluation results**. For two networks and two model sizes, this gives four pairs and eight ZIPs. Their names identify the network, model, and package type. Network and persona inputs shared by both models can be stored once per network.

The ZIPs preserve their internal organization, including complete CPTs, raw generation records, references, and evaluation tables. The three repository directories group these materials without requiring separate directories for every type of output.

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

## Computational workflow

### 1. Network preparation

Use the network and persona inputs in `data/`. The network specification defines the graph, variable descriptions, and ordered state labels. Reference CPTs remain part of the evaluation materials.

### 2. Confidence-guided CPT generation

For each network–model pair, use the corresponding generation notebook in `scripts/` and the matching inputs from `data/`. Run control and P1–P3:

- Roots receive a direct FullDist request.
- Each eligible conditional node receives one confidence-screening request covering all its parent configurations.
- Configurations at or above 0.80 receive a direct FullDist request.
- Successfully parsed direct rows become final CPT rows and fitting anchors.
- Remaining conditional rows use the midpoint RNM approximation, fitted when anchors exist and completed with defaults otherwise.

The generation package preserves the raw responses, parsing decisions, accepted vectors, retry history, influence directions, and fitting parameters. Keep the resulting ZIP in `data/`, identified by its network and model.

### 3. Persona pooling

Construct equal and configured weighted pools from P1–P3. Apply the seismic node overrides when reconstructing the configured pool. Pooling requires no additional LLM calls.

### 4. Alignment and evaluation

Use the evaluation notebook in `scripts/` with the matching generation ZIP. Align generated and reference probabilities by node name, child-state name, and every parent name/state pair. Compute the same metrics for all 24 outputs and the corresponding uniform baselines. Keep the evaluation ZIP in `data/` alongside its generation package.

### 5. Diagnostic analysis

The [supplementary tables](online-resource/secondary-tables.md) report route counts, root/conditional decompositions, smoothing sensitivity, boundary-probability counts, and node contributions. The [response audit](online-resource/response-audit.md) connects selected numerical outputs to the original response wording.

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

## Contents of the data packages

| Material | Package |
| --- | --- |
| Complete generated CPTs for control and P1–P3 | Generation ZIP |
| Raw model responses, accepted/rejected parsing attempts, and retry records | Generation ZIP |
| Row confidence, routing provenance, anchors, and RNM fitting parameters | Generation ZIP |
| Reference CPTs and state-alignment information | Evaluation ZIP |
| Individual-output and persona-pool metrics | Evaluation ZIP |
| Errors by probability entry, CPT row, node, and construction source | Evaluation ZIP |

The Markdown documents present the detailed method and analyses, while the ZIP packages retain the corresponding computational records and machine-readable tables.

Raw responses should remain available in their original textual form. Normalized CPTs alone cannot establish whether an accepted tuple agrees with the surrounding response.

The [response audit](online-resource/response-audit.md) examines 27 accepted electric-vehicle FullDist responses containing zero or one. It includes the Qwen3.5-4B/P3 ignition-source root case, whose accepted tuple contributes 74.35% of that condition's reference-weighted error, together with the original rejection excerpt, state order, accepted/reference vectors, and percentage denominator.

## Recomputing archived results

The primary reproduction route evaluates the preserved outputs without requesting new LLM responses:

1. Select the matching generation and evaluation ZIPs in `data/` for the network and model of interest.
2. Recover the generated CPTs, matching references, configuration records, and row provenance.
3. Reconstruct both persona pools and the uniform baselines.
4. Use the evaluation notebook and supporting scripts in `scripts/`, following the recorded alignment and smoothing rules. Provide the selected generation ZIP when the notebook requests its input.
5. Compare archived and recomputed results and record numerical differences.
6. Compare the resulting summaries with the [supplementary tables](online-resource/secondary-tables.md) and use the [methodological details](online-resource/methodological-details.md) to check metric and routing definitions.

The notebooks document their dependencies and execution cells. Evaluation uses the preserved probabilities and reference network; it does not require new language-model responses.

Generating new LLM outputs is a separate execution route. It requires the model dependencies and suitable inference hardware, and produces a new experimental realization. The archived seeds do not guarantee identical regeneration across model revisions, software environments, or hardware.

## Interpretation and versioning

- One generation per condition supports descriptive comparisons of the archived outputs; it does not estimate repeated-run stability.
- CPT rows share prompts, node parameters, and network context. They are not independent experimental replications.
- The confidence score expresses self-reported familiarity for elicitation, not an empirically calibrated chance of a correct probability estimate.
- The completion implementation uses state midpoints and a weighted-mean truncated-normal construction. Results concern this approximation rather than every RNM formulation.
- The archived design does not contain matched direct-only, RNM-only, and hybrid outputs for every row.
- Route error shares must be interpreted together with route counts and reference weights.

Preserve a fixed package version corresponding to the manuscript. Extraction fixes, new model generations, changed thresholds, and revised completion procedures should be identified as subsequent versions, with an explicit record of which outputs and analyses changed.

## Licensing

The intended licensing arrangement is:

| Material | License |
|---|---|
| Original project code, scripts, and notebook code | [MIT](https://opensource.org/license/mit) |
| Original documentation, figures, and authored supplementary material | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| Imported networks, reference data, and other third-party material | Their applicable original licenses and attribution requirements |

Include the applicable license texts and identify their scope when assembling the package. Retain source attribution for bnRep and the publications underlying the reference networks.
