# Prompt Templates and Archived Messages

This document accompanies **“Confidence-Guided Bayesian Network Elicitation with LLMs and Ranked Nodes.”** It records the prompts used for `seismic` and `electricvehicle` with Qwen3.5-4B and Qwen3.5-9B, including control, three domain personas, confidence screening, FullDist elicitation, and the retry variants present in the generation archives.

The authoritative evidence is the text stored in the four original generation ZIPs: `system_prompt` and `prompt` in the JSONL call logs, together with `personas_original.json` and `rede_original.json`. The supplied generation notebooks were used to recover and cross-check the common text builders. Generation and evaluation outputs were not modified while preparing this document.

**Version scope:** the provided notebooks and historical call logs differ in their electric-vehicle retry behavior. Section 7 preserves the messages actually sent in the archived runs and identifies that difference. The prompt bodies below retain their original English wording, punctuation, state names, and requested answer formats.

## 1. Reading the templates

Text blocks labeled **template** use angle-bracket placeholders such as `<NODE_NAME>` to describe values filled by the generation code. These placeholders are documentation notation and were not sent literally. Blocks labeled **archived message** contain the complete recorded message without placeholder substitution or ellipses.

| Placeholder | Expansion |
|---|---|
| `<RELEVANT_NODE_DESCRIPTIONS>` | Description lines for the target node, followed by its immediate parents in the recorded parent order. |
| `<NODE_NAME>` | Exact target-node identifier from the input specification. |
| `<CHILD_STATES_IN_ORDER>` | Child-state names joined with comma and space in the generated CPT order. |
| `<PARENT_CONDITIONS>` | Parent assignments joined with comma and space, for example `ConstructionQuality is Low`; roots use `there are no parent conditions`. |
| `<PARENT_NAMES>` | Parent identifiers joined with comma and space. |
| `<INDEXED_PARENT_CONFIGURATIONS>` | One zero-based indexed configuration per line, with assignments written as `Parent=State`. |
| `<LAST_ROW_INDEX>` | Number of parent configurations minus one. |
| `<ROLE>`, `<SPECIALTY>`, `<PERSONA_INSTRUCTIONS>` | Values from `papel`, `especialidade`, and `instrucoes` in `personas_original.json`. |
| `<NETWORK_NAME>` | `seismic` or `electricvehicle`, used only in the separate familiarity query. |

`PARENT_NAME` and `positive|negative|uncertain` inside the confidence JSON schema are literal instructions present in the original prompt. They are not angle-bracket placeholders introduced by this document.

## 2. Message composition and routing

The archived call interface stores a system-message string and a user-message string for each request. Persona conditions combine their system instruction with the same user-prompt construction used by control. The control CPT calls have an empty `system_prompt`; no neutral-expert role instruction replaces it.

The supplied inference code constructs a fresh message list for each call. The archived records identify the chat-template strategy as `processor_text_only_nonthinking`. This document preserves the logical message texts; it does not claim to preserve an immutable tokenizer revision or the full serialized token sequence.

| Call type | System instruction | User message | Function in the procedure |
|---|---|---|---|
| Network familiarity | Dedicated study instruction | Network-name familiarity question | Separate preliminary diagnostic. |
| Confidence screen | Empty for control; persona instruction otherwise | Confidence template | Estimates familiarity and qualitative parent directions for all rows of one conditional node. |
| Root FullDist | Empty for control; persona instruction otherwise | FullDist template without parent conditions | Direct root distribution. |
| Conditional FullDist | Empty for control; persona instruction otherwise | FullDist template with one parent configuration | Direct row and fitting anchor when selected by the gate. |
| RNM completion | No additional probability-elicitation prompt | No FullDist message for that completed row | Computes the distribution from the recorded anchors/defaults and influence directions. |

The routing threshold is 0.80. Confidence is self-reported knowledge for elicitation, not an event probability or a calibrated chance of a correct estimate. Equal and configured persona pooling are numerical operations on existing CPTs and introduce no additional prompts.

## 3. Context construction and state ordering

Local context includes the target node and its immediate parents. The graph determines the parent relationships; each local prompt does not enumerate all network edges.

The description builder uses the following available input fields, in this order: network domain, network description, node definition, entity, scale type, state meanings, temporal context, measurement context, and notes. Empty fields are omitted. The resulting fragments are joined with spaces. Each context line has this shape:

```text
- <NODE_NAME>: <COMPOSED_DESCRIPTION> Possible states: [<STATES_IN_ORDER>]
```

State names and their order come from the generation input JSON. They must be retained when interpreting the numerical tuple. Parent configurations are enumerated from the ordered parent-state lists; the rightmost parent changes fastest. The call logs preserve the actual parent order and assignments.

For `electricvehicle`, the generation inputs explicitly order states as `no, yes` for the ordinal approximation and state that the event horizon is not fixed. These are input modeling assumptions. They do not establish that binary failure mechanisms are adequately represented by a weighted mean.

### Generated state orders

| Network | Node | Child-state order |
|---|---|---|
| `seismic` | `Distance` | `Short, Medium, Long` |
| `seismic` | `Magnitude` | `Low, Medium, High` |
| `seismic` | `ShakingIntensity` | `Low, Medium, High` |
| `seismic` | `SeismicHazard` | `Low, Medium, High` |
| `seismic` | `ConstructionQuality` | `Low, Medium, High` |
| `seismic` | `StrengthDegradation` | `Low, Medium, High` |
| `seismic` | `LiveLoad` | `Low, Medium, High` |
| `seismic` | `Fragility` | `Low, Medium, High` |
| `seismic` | `Vulnerability` | `Low, Medium, High` |
| `seismic` | `SeismicRisk` | `Low, Medium, High` |
| `electricvehicle` | `ACF` | `no, yes` |
| `electricvehicle` | `AM` | `no, yes` |
| `electricvehicle` | `AWE` | `no, yes` |
| `electricvehicle` | `BEP` | `no, yes` |
| `electricvehicle` | `BF` | `no, yes` |
| `electricvehicle` | `BO` | `no, yes` |
| `electricvehicle` | `CBI` | `no, yes` |
| `electricvehicle` | `CEF` | `no, yes` |
| `electricvehicle` | `CI` | `no, yes` |
| `electricvehicle` | `DTH` | `no, yes` |
| `electricvehicle` | `EC` | `no, yes` |
| `electricvehicle` | `ECF` | `no, yes` |
| `electricvehicle` | `FFE` | `no, yes` |
| `electricvehicle` | `HF` | `no, yes` |
| `electricvehicle` | `IS` | `no, yes` |
| `electricvehicle` | `ISC` | `no, yes` |
| `electricvehicle` | `MMA` | `no, yes` |
| `electricvehicle` | `OFE` | `no, yes` |
| `electricvehicle` | `REI` | `no, yes` |
| `electricvehicle` | `SBB` | `no, yes` |
| `electricvehicle` | `SCB` | `no, yes` |
| `electricvehicle` | `TLD` | `no, yes` |
| `electricvehicle` | `VFD` | `no, yes` |

## 4. Persona system instructions

The system-message builder uses the following wrapper. The specialty line is included when the field is nonempty; all six domain-persona records below contain a specialty.

```text
You are acting in the role of: <ROLE>.
Specialty: <SPECIALTY>.
Additional persona instructions supplied by the researcher: <PERSONA_INSTRUCTIONS>
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

The documentary persona weights are not inserted into these system messages. The same persona strings are present for 4B and 9B within each domain. The risk-analyst role has domain-specific wording and is therefore reproduced separately for the two networks.

### 4.1. Control

For control CPT calls, `system_prompt` is an empty string. The user template and node context remain in place.

### 4.2. seismic personas

#### P1: Structural engineer

Persona ID: `engenheiro_estrutural`. Source: `personas_original.json` in the `seismic` archives. **Archived system message:**

```text
You are acting in the role of: Structural engineer.
Specialty: Structural fragility, construction quality and building vulnerability.
Additional persona instructions supplied by the researcher: Estimate the requested conditional probability from structural-engineering domain knowledge. State relevant assumptions and uncertainty. Do not claim to have looked up data or exact frequencies.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P2: Seismologist

Persona ID: `sismologista`. Source: `personas_original.json` in the `seismic` archives. **Archived system message:**

```text
You are acting in the role of: Seismologist.
Specialty: Earthquake magnitude, shaking intensity, distance and seismic hazard.
Additional persona instructions supplied by the researcher: Estimate conditional probabilities using domain knowledge of seismic phenomena. Explain physical relationships and acknowledge uncertainty without inventing measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P3: Probabilistic risk analyst

Persona ID: `analista_risco`. Source: `personas_original.json` in the `seismic` archives. **Archived system message:**

```text
You are acting in the role of: Probabilistic risk analyst.
Specialty: Risk assessment and uncertainty quantification.
Additional persona instructions supplied by the researcher: Provide your best informed conditional probability estimate and state major sources of uncertainty. Use no external datasets and do not present guessed probabilities as empirical measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

### 4.3. electricvehicle personas

#### P1: Electric vehicle systems engineer

Persona ID: `engenheiro_veiculos_eletricos`. Source: `personas_original.json` in the `electricvehicle` archives. **Archived system message:**

```text
You are acting in the role of: Electric vehicle systems engineer.
Specialty: Electric vehicle architecture, traction batteries, charging equipment, electrical circuits, short circuits, overheating, and interactions among technical faults that may initiate a fire.
Additional persona instructions supplied by the researcher: Estimate the requested conditional distribution using general engineering knowledge about electric vehicles. Consider the given node and parent states, distinguishing the presence from the absence of each failure or hazard. Treat the specified causal graph as given; do not introduce unlisted causal dependencies. Follow the requested output format exactly. State uncertainty honestly when asked, without claiming to know the benchmark CPTs, measured frequencies, or external data.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P2: Automotive fire safety and reliability specialist

Persona ID: `especialista_seguranca_automotiva`. Source: `personas_original.json` in the `electricvehicle` archives. **Archived system message:**

```text
You are acting in the role of: Automotive fire safety and reliability specialist.
Specialty: Electric vehicle fire prevention, reliability engineering, ignition mechanisms, collision-related ignition, hazard detection, fire-fighting equipment, and failure propagation.
Additional persona instructions supplied by the researcher: Estimate the requested conditional distribution using general knowledge of automotive safety, reliability, and fire-risk mechanisms. Evaluate how each stated parent condition affects the event of interest, including protective or mitigating mechanisms, without assuming that every cause has equal importance. Follow the requested output format exactly. Acknowledge uncertainty when asked and never claim access to benchmark probabilities, incident datasets, or measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P3: Probabilistic risk analyst

Persona ID: `analista_risco`. Source: `personas_original.json` in the `electricvehicle` archives. **Archived system message:**

```text
You are acting in the role of: Probabilistic risk analyst.
Specialty: Bayesian risk assessment, conditional probability elicitation, uncertainty quantification, dependencies among rare events, and reliability of risk estimates.
Additional persona instructions supplied by the researcher: Provide an informed estimate of the conditional probability distribution for the specified node and parent configuration. Distinguish logical constraints from uncertain causal tendencies, consider rare-event uncertainty, and avoid unsupported numerical precision. Follow the requested output format exactly. Report uncertainty honestly when asked; do not fabricate observations, cite inaccessible empirical studies, or claim familiarity with the reference CPTs.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

## 5. Confidence-screening user template

One confidence request covers all parent configurations of a conditional node. The JSON request provides an output schema rather than a few-shot demonstration. It requests both row-level confidence and qualitative parent influence, with no CPT probabilities.

**Template:**

```text
These variables belong to a Bayesian Network:

<RELEVANT_NODE_DESCRIPTIONS>

Before estimating any numerical CPT probabilities, assess whether your INTERNAL KNOWLEDGE is sufficient to make an informed conditional probability estimate for each parent configuration of node <NODE_NAME>.

Important definition of confidence:
- confidence measures how much relevant knowledge you believe you have for estimating the FULL probability distribution;
- it does NOT mean how easy it is to invent a plausible numeric answer;
- use values from 0 to 1;
- do not provide any CPT probabilities in this response.

Also report the qualitative influence of each parent on higher states of <NODE_NAME>: "positive", "negative", or "uncertain". This direction is only a qualitative aid for Ranked Nodes.

Parents: <PARENT_NAMES>
Configurations:
<INDEXED_PARENT_CONFIGURATIONS>

Return ONLY one valid JSON object in exactly this structure:
{
  "parent_influence": [
    {"parent": "PARENT_NAME", "direction": "positive|negative|uncertain", "confidence": 0.0}
  ],
  "rows": [
    {"row": 0, "confidence": 0.0}
  ]
}

Include every parent exactly once and every row index from 0 to <LAST_ROW_INDEX> exactly once. Do not include explanations outside the JSON.
```

The ordinal direction refers to higher states in the supplied order. The recorded screen can influence later direct/ranked-node routing and RNM fitting; it does not validate the selected probabilities. All 112 archived confidence requests were made at temperature 0.0 on their first parsing attempt.

## 6. FullDist user template

Roots and selected conditional rows use the same FullDist wording. The parent-condition clause distinguishes the two cases. The prompt asks for reasoning followed by a final tuple, in the explicit child-state order.

**Template:**

```text
These nodes are related to the question inside a Bayesian Network:

<RELEVANT_NODE_DESCRIPTIONS>

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (<CHILD_STATES_IN_ORDER>)

Given that <PARENT_CONDITIONS>, what is the probability distribution of <NODE_NAME>?
```

For a root, the final question is generated as:

```text
Given that there are no parent conditions, what is the probability distribution of <NODE_NAME>?
```

For a conditional row, the final question uses the recorded assignments:

```text
Given that <PARENT_1> is <STATE_1>, <PARENT_2> is <STATE_2>, what is the probability distribution of <NODE_NAME>?
```

The number of parent assignments varies by node. Numerical tuple entries must correspond to `Order of states`; variable labels and reference-state ordering must not be silently interchanged. The request for a final marker does not by itself ensure that the extractor accepts a tuple actually endorsed by the response.

## 7. Retry behavior present in the archived calls

### 7.1. Seismic runs

The seismic parsing retries repeat the original user prompt and keep the same system instruction for the condition. Five additional FullDist attempts occur in the 4B archive and one in the 9B archive. One 4B request reaches parsing attempt 3; its text is still the original FullDist prompt.

### 7.2. Electric-vehicle runs

The first FullDist parsing attempt uses the original user prompt. Recorded second attempts append **two newline characters** followed by this exact clarification:

```text
Clarification for probability elicitation: the task is to produce a SUBJECTIVE prior probability assessment, not to retrieve empirical frequencies or the benchmark CPT. No dataset is expected or available. Use general domain reasoning, explicitly acknowledge uncertainty, and end with EXACTLY the requested numeric tuple in state order. Unknown/undefined/NaN or an all-zero tuple are not probabilities. Do not claim these numbers are measured facts.
```

The system instruction remains unchanged. The additional text appears in 21 calls for 4B and 12 calls for 9B, under `prompt_revision = subjective_prior_reprompt_v2`. The complete retry input is the original FullDist message plus the suffix above; it is not a conversation containing the previous response.

No third-or-later electric-vehicle parsing attempt appears in these archived calls. Consequently, no further electric-vehicle retry wording is exported here as an observed prompt.

### 7.3. Relationship to the supplied notebooks

In the two consulted notebook copies, `query_llm_full_distribution` repeats the original prompt on parsing retries and raises the output-token limit after the first attempt. It does not contain the archived electric-vehicle clarification. The base FullDist, confidence, persona, and familiarity text builders match the archived messages; the electric-vehicle retry addition is established directly from the call logs.

The supplied notebook constants allow five FullDist parsing attempts and four confidence parsing attempts. The observed attempt counts are reported in Section 10. Preserve this distinction when packaging the historical implementation or running a revised one.

## 8. Preliminary network-familiarity query

Each network–model archive contains one separate network-name query in `familiaridade_rede_chamadas.jsonl`. It is distinct from the confidence screen for conditional rows and has its own system instruction. The supplied procedure gives this query a fresh context and does not inject its response into the CPT prompts.

**Archived system message:**

```text
You are participating in a scientific study of Bayesian network parameter elicitation. Report uncertainty accurately. Do not pretend to remember a benchmark, graph or CPT when you only recognize its general topic. Do not invent numerical probabilities. You have no external tools in this query.
```

**User template:**

```text
Before being shown any graph, node descriptions, states, data, or probability tables: were you ALREADY familiar with the SPECIFIC named Bayesian network benchmark "<NETWORK_NAME>"? Distinguish clearly: (a) knowledge of its general domain, (b) recognition of this exact benchmark/network by name, (c) prior knowledge of its original nodes and edges, and (d) prior knowledge of its original numerical CPTs. Answer yes, no, or uncertain for each aspect and briefly explain what you know, or say you do not know. Do not guess CPT numbers or claim that the generic domain is the named benchmark.
```

The network name is the only network-specific information in this diagnostic request. The four records are additional to the 423 generation calls counted below. A model's denial of familiarity is a self-report and does not rule out pretraining exposure.

## 9. Complete examples from the archived messages

The examples below preserve the full user message. Their system instruction is identified separately and, for persona conditions, reproduced in Section 4. They illustrate inputs; no reference probabilities or substituted numerical answers have been added.

### 9.1. Seismic confidence screen

Archive: `resultados_qwen35_4b_confidence_ranked_nodes_seismic.zip`; member: `logs/controle_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 2; call ID: 3; condition: `controle`; parsing attempt: 1.

System instruction: empty string (control).

**Archived user message:**

```text
These variables belong to a Bayesian Network:

- ShakingIntensity: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: ShakingIntensity represents the level of ground shaking experienced at the structure or site. Entity: Structure or site affected by the earthquake event. Scale type: ordinal State meanings: Low: The site experiences low ground shaking intensity.; Medium: The site experiences intermediate ground shaking intensity.; High: The site experiences high ground shaking intensity. Temporal context: Condition during the earthquake event or seismic scenario. Measurement context: Qualitative classification of ground motion intensity at the site. Notes: Shaking intensity is an ordered category describing the severity of ground motion. Possible states: [Low, Medium, High]
- Distance: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: Distance represents the distance from the structure or site to the earthquake source or epicenter. Entity: Structure or site under seismic assessment. Scale type: ordinal State meanings: Short: The structure or site is close to the earthquake source.; Medium: The structure or site is at an intermediate distance from the earthquake source.; Long: The structure or site is far from the earthquake source. Temporal context: Condition considered for a seismic event or seismic assessment scenario. Measurement context: Qualitative discretization of source-to-site distance. Notes: Distance is represented as an ordered categorical variable from closer to farther. Possible states: [Short, Medium, Long]
- Magnitude: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: Magnitude represents the severity or released energy of an earthquake event. Entity: Earthquake event considered in the seismic assessment. Scale type: ordinal State meanings: Low: The earthquake has low magnitude or low released seismic energy.; Medium: The earthquake has intermediate magnitude or intermediate released seismic energy.; High: The earthquake has high magnitude or high released seismic energy. Temporal context: Observed or assumed for the earthquake event being assessed. Measurement context: Qualitative discretization of earthquake magnitude. Notes: Magnitude is represented as an ordered categorical variable from lower to higher severity. Possible states: [Low, Medium, High]

Before estimating any numerical CPT probabilities, assess whether your INTERNAL KNOWLEDGE is sufficient to make an informed conditional probability estimate for each parent configuration of node ShakingIntensity.

Important definition of confidence:
- confidence measures how much relevant knowledge you believe you have for estimating the FULL probability distribution;
- it does NOT mean how easy it is to invent a plausible numeric answer;
- use values from 0 to 1;
- do not provide any CPT probabilities in this response.

Also report the qualitative influence of each parent on higher states of ShakingIntensity: "positive", "negative", or "uncertain". This direction is only a qualitative aid for Ranked Nodes.

Parents: Distance, Magnitude
Configurations:
0: Distance=Short, Magnitude=Low
1: Distance=Short, Magnitude=Medium
2: Distance=Short, Magnitude=High
3: Distance=Medium, Magnitude=Low
4: Distance=Medium, Magnitude=Medium
5: Distance=Medium, Magnitude=High
6: Distance=Long, Magnitude=Low
7: Distance=Long, Magnitude=Medium
8: Distance=Long, Magnitude=High

Return ONLY one valid JSON object in exactly this structure:
{
  "parent_influence": [
    {"parent": "PARENT_NAME", "direction": "positive|negative|uncertain", "confidence": 0.0}
  ],
  "rows": [
    {"row": 0, "confidence": 0.0}
  ]
}

Include every parent exactly once and every row index from 0 to 8 exactly once. Do not include explanations outside the JSON.
```

### 9.2. Seismic root FullDist

Archive: `resultados_qwen35_4b_confidence_ranked_nodes_seismic.zip`; member: `logs/controle_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 4; call ID: 5; condition: `controle`; parsing attempt: 1.

System instruction: empty string (control).

**Archived user message:**

```text
These nodes are related to the question inside a Bayesian Network:

- Magnitude: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: Magnitude represents the severity or released energy of an earthquake event. Entity: Earthquake event considered in the seismic assessment. Scale type: ordinal State meanings: Low: The earthquake has low magnitude or low released seismic energy.; Medium: The earthquake has intermediate magnitude or intermediate released seismic energy.; High: The earthquake has high magnitude or high released seismic energy. Temporal context: Observed or assumed for the earthquake event being assessed. Measurement context: Qualitative discretization of earthquake magnitude. Notes: Magnitude is represented as an ordered categorical variable from lower to higher severity. Possible states: [Low, Medium, High]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (Low, Medium, High)

Given that there are no parent conditions, what is the probability distribution of Magnitude?
```

### 9.3. Seismic conditional FullDist

Archive: `resultados_qwen35_9b_confidence_ranked_nodes_seismic.zip`; member: `logs/engenheiro_estrutural_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 18; call ID: 42; condition: `engenheiro_estrutural`; parsing attempt: 1.

System instruction: the `engenheiro_estrutural` message in Section 4 for `seismic`.

**Archived user message:**

```text
These nodes are related to the question inside a Bayesian Network:

- StrengthDegradation: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: StrengthDegradation represents the reduction of structural strength due to deterioration, aging, damage, or material degradation. Entity: Structure or building under seismic assessment. Scale type: ordinal State meanings: Low: The structure has low degradation of structural strength.; Medium: The structure has intermediate degradation of structural strength.; High: The structure has high degradation of structural strength. Temporal context: Condition at the time of the structural or seismic assessment. Measurement context: Qualitative classification of the current level of structural strength degradation. Notes: Strength degradation is represented as an ordered category from lower to higher degradation. Possible states: [Low, Medium, High]
- ConstructionQuality: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: ConstructionQuality represents the quality of design, materials, construction practices, and structural detailing. Entity: Structure or building under seismic assessment. Scale type: ordinal State meanings: Low: The structure has low construction quality, poor materials, inadequate detailing, or deficient construction practices.; Medium: The structure has intermediate construction quality, materials, detailing, or construction practices.; High: The structure has high construction quality, good materials, adequate detailing, or reliable construction practices. Temporal context: Condition associated with the construction and current assessment of the structure. Measurement context: Qualitative classification of structural construction quality. Notes: Construction quality is represented as an ordered category from poorer to better quality. Possible states: [Low, Medium, High]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (Low, Medium, High)

Given that ConstructionQuality is Low, what is the probability distribution of StrengthDegradation?
```

### 9.4. Electric-vehicle confidence screen

Archive: `resultados_qwen35_4b_confidence_ranked_nodes_eletric.zip`; member: `logs/controle_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 16; call ID: 17; condition: `controle`; parsing attempt: 1.

System instruction: empty string (control).

**Archived user message:**

```text
These variables belong to a Bayesian Network:

- CBI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: The car body is ignited. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CBI is absent / does not occur.; yes: The event CBI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- IS: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Ignition source. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event IS is absent / does not occur.; yes: The event IS is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- CI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Collision ignition. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CI is absent / does not occur.; yes: The event CI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]

Before estimating any numerical CPT probabilities, assess whether your INTERNAL KNOWLEDGE is sufficient to make an informed conditional probability estimate for each parent configuration of node CBI.

Important definition of confidence:
- confidence measures how much relevant knowledge you believe you have for estimating the FULL probability distribution;
- it does NOT mean how easy it is to invent a plausible numeric answer;
- use values from 0 to 1;
- do not provide any CPT probabilities in this response.

Also report the qualitative influence of each parent on higher states of CBI: "positive", "negative", or "uncertain". This direction is only a qualitative aid for Ranked Nodes.

Parents: IS, CI
Configurations:
0: IS=no, CI=no
1: IS=no, CI=yes
2: IS=yes, CI=no
3: IS=yes, CI=yes

Return ONLY one valid JSON object in exactly this structure:
{
  "parent_influence": [
    {"parent": "PARENT_NAME", "direction": "positive|negative|uncertain", "confidence": 0.0}
  ],
  "rows": [
    {"row": 0, "confidence": 0.0}
  ]
}

Include every parent exactly once and every row index from 0 to 3 exactly once. Do not include explanations outside the JSON.
```

### 9.5. Electric-vehicle root FullDist

Archive: `resultados_qwen35_4b_confidence_ranked_nodes_eletric.zip`; member: `logs/controle_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 1; call ID: 2; condition: `controle`; parsing attempt: 1.

System instruction: empty string (control).

**Archived user message:**

```text
These nodes are related to the question inside a Bayesian Network:

- BF: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Battery failure. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event BF is absent / does not occur.; yes: The event BF is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (no, yes)

Given that there are no parent conditions, what is the probability distribution of BF?
```

### 9.6. Electric-vehicle conditional FullDist

Archive: `resultados_qwen35_9b_electricvehicle_confidence_ranked_nodes.zip`; member: `logs/engenheiro_veiculos_eletricos_ConfidenceRankedNodes_r1_chamadas.jsonl`; JSONL line: 25; call ID: 53; condition: `engenheiro_veiculos_eletricos`; parsing attempt: 1.

System instruction: the `engenheiro_veiculos_eletricos` message in Section 4 for `electricvehicle`.

**Archived user message:**

```text
These nodes are related to the question inside a Bayesian Network:

- CBI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: The car body is ignited. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CBI is absent / does not occur.; yes: The event CBI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- IS: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Ignition source. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event IS is absent / does not occur.; yes: The event IS is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- CI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Collision ignition. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CI is absent / does not occur.; yes: The event CI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (no, yes)

Given that IS is no, CI is no, what is the probability distribution of CBI?
```

## 10. Prompt-reconstruction checks

The documentation was checked against every archived generation request. The common notebook formatters reconstruct the 112 confidence messages and the base FullDist messages exactly. For the 33 electric-vehicle retries, appending the recorded clarification reconstructs the complete user message. Every persona/control system string and all four familiarity system/user pairs were also matched.

| Network | Model | Confidence calls | FullDist calls, including retries | Additional FullDist attempts | Maximum FullDist parsing attempt |
|---|---|---:|---:|---:|---:|
| `seismic` | `Qwen3.5-4B` | 24 | 24 | 5 | 3 |
| `seismic` | `Qwen3.5-9B` | 24 | 70 | 1 | 2 |
| `electricvehicle` | `Qwen3.5-4B` | 32 | 83 | 21 | 2 |
| `electricvehicle` | `Qwen3.5-9B` | 32 | 134 | 12 | 2 |

These counts total **112 confidence calls and 311 FullDist calls**, including **39 additional FullDist attempts**, plus **four separate familiarity calls**. The 39 additional attempts correspond to parsing retries; this document does not assign them an error category without consulting the parser records.

Both notebook copies contain the same relevant text-building function bodies. These are text-construction checks using preserved inputs and messages, not new model executions, a reproduction of the evaluation metrics, or verification of semantic correctness of generated probabilities.

## 11. Source inventory and integrity

The source ZIP names below identify the files consulted for this document. Member paths refer to entries inside each ZIP. Preserve these artifacts or record any packaging transformations explicitly.

| Source ZIP | Model / network | Recorded run start (UTC) | SHA-256 |
|---|---|---|---|
| `resultados_qwen35_4b_confidence_ranked_nodes_seismic.zip` | `Qwen3.5-4B` / `seismic` | `2026-10-05T11:43:10.136646+00:00` | `9f4bd951e0a6429dafbd28523b2d6b536c284adf9d1a2cc028cfda30b7c78826` |
| `resultados_qwen35_9b_confidence_ranked_nodes_seismic.zip` | `Qwen3.5-9B` / `seismic` | `2026-10-03T23:02:48.147007+00:00` | `083b9b85a005243423d8eba50df2c9e6d2bc9952702fdc18c51459fcee2eb49d` |
| `resultados_qwen35_4b_confidence_ranked_nodes_eletric.zip` | `Qwen3.5-4B` / `electricvehicle` | `2026-10-08T21:43:14.542057+00:00` | `eb908639bf0ae95aefe00dfab0f4650b018c2f9cbbd6c474b95374c794e4450b` |
| `resultados_qwen35_9b_electricvehicle_confidence_ranked_nodes.zip` | `Qwen3.5-9B` / `electricvehicle` | `2026-10-10T12:32:12.253735+00:00` | `5fd74699c5f79851d4a0ad418e7b83854ecc873412dc0cb42b7779ef1c93e9ea` |

The prompt sources inside each generation archive are:

- `logs/*_chamadas.jsonl`: exact system/user strings, condition, phase, parsing attempt, and recorded call metadata.
- `personas_original.json`: role, specialty, and additional instructions for each domain persona.
- `rede_original.json`: network and node metadata, state meanings, and supplied state orders.
- `familiaridade_rede_chamadas.jsonl`: the separate name-only familiarity request.
- `manifesto_geracao.json`: model, network identifier, run configuration, and execution inventory.

The consulted notebook copies are listed separately because their names and upload dates alone do not establish that they contain every historical runtime change:

| Notebook | SHA-256 |
|---|---|
| `Qwen35_4B_01_Gerar_CPTs_Confidence_RankedNodes_Colab_SemDrive(4).ipynb` | `c38b0356dd24883f69b471149f55ddc47211be00f61e70f25e0d38d364699177` |
| `Qwen35_9B_01_Gerar_CPTs_Confidence_RankedNodes_Colab_SemDrive(6).ipynb` | `c5857e18ed18692d40b9e0f899bd941ebcafa1c1bd94b74bf62377b31f1718a1` |

Relevant function names are `load_bn_spec`, `node_context_text`, `prompt_sistema_persona`, `format_prompt_confianca_no`, `format_epk_prompt_full_distribution`, `_prompt_familiaridade_rede`, and `query_llm_full_distribution`. The text-builder checks use the recorded parent ordering; they do not revalidate the network graph.

For a particular result row, locate its network/model archive, condition, target node, parent assignment, and parsing attempt in the call logs. Inspect the accompanying parser records and accepted CPT when evaluating whether the response was processed correctly. Full generated responses and parsing decisions belong in the response-audit materials rather than being inferred from the prompt template.
