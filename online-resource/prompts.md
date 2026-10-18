# Prompt Templates

The elicitation protocol uses Qwen3.5-4B and Qwen3.5-9B to construct conditional probability tables for the seismic and electric-vehicle networks. Each network–model pair is evaluated under a control condition and three domain-specific personas.

The following sections present the system instructions, confidence-screening and FullDist templates, retry instructions, and complete examples of the messages used in the study. Each example is identified by its model, network, prompting condition, and generation parameters. Prompt wording and state ordering are reproduced as used during elicitation.

## 1. Reading the templates

Text blocks labeled **template** use angle-bracket placeholders such as `<NODE_NAME>` to describe values filled by the generation code. These placeholders are documentation notation and were not sent literally. Blocks labeled **example message** contain the complete message used in the corresponding condition.

| Placeholder | Expansion |
|---|---|
| `<RELEVANT_NODE_DESCRIPTIONS>` | Description lines for the target node, followed by its immediate parents in the recorded parent order. |
| `<NODE_NAME>` | Exact target-node identifier from the input specification. |
| `<CHILD_STATES_IN_ORDER>` | Child-state names joined with comma and space in the generated CPT order. |
| `<PARENT_CONDITIONS>` | Parent assignments joined with comma and space, for example `ConstructionQuality is Low`; roots use `there are no parent conditions`. |
| `<PARENT_NAMES>` | Parent identifiers joined with comma and space. |
| `<INDEXED_PARENT_CONFIGURATIONS>` | One zero-based indexed configuration per line, with assignments written as `Parent=State`. |
| `<LAST_ROW_INDEX>` | Number of parent configurations minus one. |
| `<ROLE>`, `<SPECIALTY>`, `<PERSONA_INSTRUCTIONS>` | The role, specialty, and additional instructions defining the persona. |
| `<NETWORK_NAME>` | `seismic` or `electricvehicle`, used only in the separate familiarity query. |

`PARENT_NAME` and `positive|negative|uncertain` inside the confidence JSON schema are literal instructions present in the original prompt. They are not angle-bracket placeholders introduced by this document.

## 2. Message composition and routing

Each request combines a system instruction with a user prompt. Persona conditions use the role-specific system instructions in Section 4. Control uses an empty system instruction and the same user-prompt template.

Requests use a fresh context, with thinking mode configured as disabled. The user prompt contains the local node descriptions and the question to be answered. Generation settings are summarized in Section 11.

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

State names and their order are defined in the network specification. They must be retained when interpreting the numerical tuple. Parent configurations are enumerated from the ordered parent-state lists; the rightmost parent changes fastest. The parent order is retained when constructing each conditional query.

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

Persona weights are applied during pooling. Within each domain, Qwen3.5-4B and Qwen3.5-9B receive the same persona instructions. The risk-analyst instructions differ between domains and are presented separately.

### 4.1. Control

Control uses the user template and node context with an empty system instruction.

### 4.2. Seismic personas

#### P1: Structural engineer

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

```text
You are acting in the role of: Structural engineer.
Specialty: Structural fragility, construction quality and building vulnerability.
Additional persona instructions supplied by the researcher: Estimate the requested conditional probability from structural-engineering domain knowledge. State relevant assumptions and uncertainty. Do not claim to have looked up data or exact frequencies.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P2: Seismologist

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

```text
You are acting in the role of: Seismologist.
Specialty: Earthquake magnitude, shaking intensity, distance and seismic hazard.
Additional persona instructions supplied by the researcher: Estimate conditional probabilities using domain knowledge of seismic phenomena. Explain physical relationships and acknowledge uncertainty without inventing measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P3: Probabilistic risk analyst

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

```text
You are acting in the role of: Probabilistic risk analyst.
Specialty: Risk assessment and uncertainty quantification.
Additional persona instructions supplied by the researcher: Provide your best informed conditional probability estimate and state major sources of uncertainty. Use no external datasets and do not present guessed probabilities as empirical measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

### 4.3. Electric-vehicle personas

#### P1: Electric vehicle systems engineer

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

```text
You are acting in the role of: Electric vehicle systems engineer.
Specialty: Electric vehicle architecture, traction batteries, charging equipment, electrical circuits, short circuits, overheating, and interactions among technical faults that may initiate a fire.
Additional persona instructions supplied by the researcher: Estimate the requested conditional distribution using general engineering knowledge about electric vehicles. Consider the given node and parent states, distinguishing the presence from the absence of each failure or hazard. Treat the specified causal graph as given; do not introduce unlisted causal dependencies. Follow the requested output format exactly. State uncertainty honestly when asked, without claiming to know the benchmark CPTs, measured frequencies, or external data.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P2: Automotive fire safety and reliability specialist

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

```text
You are acting in the role of: Automotive fire safety and reliability specialist.
Specialty: Electric vehicle fire prevention, reliability engineering, ignition mechanisms, collision-related ignition, hazard detection, fire-fighting equipment, and failure propagation.
Additional persona instructions supplied by the researcher: Estimate the requested conditional distribution using general knowledge of automotive safety, reliability, and fire-risk mechanisms. Evaluate how each stated parent condition affects the event of interest, including protective or mitigating mechanisms, without assuming that every cause has equal importance. Follow the requested output format exactly. Acknowledge uncertainty when asked and never claim access to benchmark probabilities, incident datasets, or measurements.
Follow the exact output format requested by the user prompt. Report uncertainty honestly.
```

#### P3: Probabilistic risk analyst

**Models:** Qwen3.5-4B and Qwen3.5-9B. **System instruction:**

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

The ordinal direction refers to higher states in the supplied order. The recorded screen can influence later direct/ranked-node routing and RNM fitting; it does not validate the selected probabilities. All 112 confidence requests used greedy decoding at temperature 0.0 and were accepted on the first parsing attempt.

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

## 7. Retry procedure

### 7.1. Seismic runs

Seismic retries repeat the original user prompt and retain the system instruction for the condition. Qwen3.5-4B required five additional FullDist attempts and Qwen3.5-9B required one. One Qwen3.5-4B request reached a third parsing attempt using the same FullDist wording.

### 7.2. Electric-vehicle runs

The first FullDist parsing attempt uses the original user prompt. Recorded second attempts append **two newline characters** followed by this exact clarification:

```text
Clarification for probability elicitation: the task is to produce a SUBJECTIVE prior probability assessment, not to retrieve empirical frequencies or the benchmark CPT. No dataset is expected or available. Use general domain reasoning, explicitly acknowledge uncertainty, and end with EXACTLY the requested numeric tuple in state order. Unknown/undefined/NaN or an all-zero tuple are not probabilities. Do not claim these numbers are measured facts.
```

The second attempt retains the system instruction and appends the clarification to the original FullDist prompt. This procedure was used in 21 Qwen3.5-4B calls and 12 Qwen3.5-9B calls. Each retry is issued in a fresh context.

All electric-vehicle FullDist requests were completed within two parsing attempts.

### 7.3. Retry limits and implementation

The available generation implementation allows up to five FullDist parsing attempts and four confidence parsing attempts, increasing the FullDist output limit after the first attempt. It resubmits the base prompt on retry. Reproducing the electric-vehicle runs additionally requires the clarification in Section 7.2, which was present in those executions.

## 8. Preliminary network-familiarity query

A preliminary query asks each model about its familiarity with the named network. This diagnostic is performed once per network–model pair, in a fresh context, with the system instruction below. Its response is not included in subsequent CPT-elicitation prompts.

**System instruction:**

```text
You are participating in a scientific study of Bayesian network parameter elicitation. Report uncertainty accurately. Do not pretend to remember a benchmark, graph or CPT when you only recognize its general topic. Do not invent numerical probabilities. You have no external tools in this query.
```

**User template:**

```text
Before being shown any graph, node descriptions, states, data, or probability tables: were you ALREADY familiar with the SPECIFIC named Bayesian network benchmark "<NETWORK_NAME>"? Distinguish clearly: (a) knowledge of its general domain, (b) recognition of this exact benchmark/network by name, (c) prior knowledge of its original nodes and edges, and (d) prior knowledge of its original numerical CPTs. Answer yes, no, or uncertain for each aspect and briefly explain what you know, or say you do not know. Do not guess CPT numbers or claim that the generic domain is the named benchmark.
```

The network name is the only network-specific information in this diagnostic request. These four diagnostic queries are additional to the 423 generation calls summarized in Section 10. A model's denial of familiarity is a self-report and does not rule out pretraining exposure.

## 9. Complete prompt examples

Each example presents the complete user prompt for a specific model and condition. Persona system instructions are given in Section 4. Generation parameters accompany each example.

### 9.1. Seismic confidence screen

| Setting | Value |
|---|---|
| Model | Qwen3.5-4B |
| Network | seismic |
| Condition | Control |
| Target node | ShakingIntensity |
| Parent configuration | All 9 configurations of Distance, Magnitude |
| Decoding | Greedy; temperature 0.0; maximum 1,536 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 3222242815 |

System instruction: empty (control).

**Example message:**

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

| Setting | Value |
|---|---|
| Model | Qwen3.5-4B |
| Network | seismic |
| Condition | Control |
| Target node | Magnitude |
| Parent configuration | Unconditional root distribution |
| Decoding | Sampling; temperature 0.1; top-p 1.0; maximum 2,048 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 4098443485 |

System instruction: empty (control).

**Example message:**

```text
These nodes are related to the question inside a Bayesian Network:

- Magnitude: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: Magnitude represents the severity or released energy of an earthquake event. Entity: Earthquake event considered in the seismic assessment. Scale type: ordinal State meanings: Low: The earthquake has low magnitude or low released seismic energy.; Medium: The earthquake has intermediate magnitude or intermediate released seismic energy.; High: The earthquake has high magnitude or high released seismic energy. Temporal context: Observed or assumed for the earthquake event being assessed. Measurement context: Qualitative discretization of earthquake magnitude. Notes: Magnitude is represented as an ordered categorical variable from lower to higher severity. Possible states: [Low, Medium, High]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (Low, Medium, High)

Given that there are no parent conditions, what is the probability distribution of Magnitude?
```

### 9.3. Seismic conditional FullDist

| Setting | Value |
|---|---|
| Model | Qwen3.5-9B |
| Network | seismic |
| Condition | P1 — Structural engineer |
| Target node | StrengthDegradation |
| Parent configuration | ConstructionQuality=Low |
| Decoding | Sampling; temperature 0.1; top-p 1.0; maximum 2,048 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 1597556010 |

System instruction: P1, Structural engineer (Section 4.2).

**Example message:**

```text
These nodes are related to the question inside a Bayesian Network:

- StrengthDegradation: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: StrengthDegradation represents the reduction of structural strength due to deterioration, aging, damage, or material degradation. Entity: Structure or building under seismic assessment. Scale type: ordinal State meanings: Low: The structure has low degradation of structural strength.; Medium: The structure has intermediate degradation of structural strength.; High: The structure has high degradation of structural strength. Temporal context: Condition at the time of the structural or seismic assessment. Measurement context: Qualitative classification of the current level of structural strength degradation. Notes: Strength degradation is represented as an ordered category from lower to higher degradation. Possible states: [Low, Medium, High]
- ConstructionQuality: Network domain: Seismic risk assessment. Network description: This Bayesian Network represents conceptual relationships among earthquake characteristics, site conditions, structural condition, fragility, vulnerability, and seismic risk. Definition: ConstructionQuality represents the quality of design, materials, construction practices, and structural detailing. Entity: Structure or building under seismic assessment. Scale type: ordinal State meanings: Low: The structure has low construction quality, poor materials, inadequate detailing, or deficient construction practices.; Medium: The structure has intermediate construction quality, materials, detailing, or construction practices.; High: The structure has high construction quality, good materials, adequate detailing, or reliable construction practices. Temporal context: Condition associated with the construction and current assessment of the structure. Measurement context: Qualitative classification of structural construction quality. Notes: Construction quality is represented as an ordered category from poorer to better quality. Possible states: [Low, Medium, High]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (Low, Medium, High)

Given that ConstructionQuality is Low, what is the probability distribution of StrengthDegradation?
```

### 9.4. Electric-vehicle confidence screen

| Setting | Value |
|---|---|
| Model | Qwen3.5-4B |
| Network | electricvehicle |
| Condition | Control |
| Target node | CBI |
| Parent configuration | All 4 configurations of IS, CI |
| Decoding | Greedy; temperature 0.0; maximum 1,536 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 2989137400 |

System instruction: empty (control).

**Example message:**

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

| Setting | Value |
|---|---|
| Model | Qwen3.5-4B |
| Network | electricvehicle |
| Condition | Control |
| Target node | BF |
| Parent configuration | Unconditional root distribution |
| Decoding | Sampling; temperature 0.1; top-p 1.0; maximum 2,048 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 643433159 |

System instruction: empty (control).

**Example message:**

```text
These nodes are related to the question inside a Bayesian Network:

- BF: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Battery failure. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event BF is absent / does not occur.; yes: The event BF is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (no, yes)

Given that there are no parent conditions, what is the probability distribution of BF?
```

### 9.6. Electric-vehicle conditional FullDist

| Setting | Value |
|---|---|
| Model | Qwen3.5-9B |
| Network | electricvehicle |
| Condition | P1 — Electric vehicle systems engineer |
| Target node | CBI |
| Parent configuration | IS=no, CI=no |
| Decoding | Sampling; temperature 0.1; top-p 1.0; maximum 2,048 output tokens |
| Generation / parsing attempt | r1 / 1 |
| Call seed | 3119613890 |

System instruction: P1, Electric vehicle systems engineer (Section 4.3).

**Example message:**

```text
These nodes are related to the question inside a Bayesian Network:

- CBI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: The car body is ignited. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CBI is absent / does not occur.; yes: The event CBI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- IS: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Ignition source. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event IS is absent / does not occur.; yes: The event IS is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]
- CI: Network domain: Electric vehicle fire risk assessment. Network description: Fire-risk Bayesian network for electric vehicles, with component failures, environmental/human factors and intermediate ignition events. Based on the electricvehicle network documented in bnRep and Chen, Li and Yang (2022). Definition: Collision ignition. Entity: Electric vehicle fire risk event Scale type: ordinal State meanings: no: The event CI is absent / does not occur.; yes: The event CI is present / occurs. Temporal context: Occurrence of the event within the modeled electric-vehicle fire scenario; no fixed time horizon specified. Measurement context: Binary event (no/yes). Ordered no < yes for Ranked Nodes as absence to occurrence; this is a modeling assumption, not the bnRep variable-class declaration. Notes: Evaluate occurrence probability for this binary risk/failure event; the network contains no original CPT probabilities. Possible states: [no, yes]

Given this information, answer the following question by providing the probability distribution of the node. Your answer should include your reasoning and at the end a sentence that says 'The probability distribution of the node is:' followed by the probabilities given in a tuple with each probability representing a state in the given order.

Order of states: (no, yes)

Given that IS is no, CI is no, what is the probability distribution of CBI?
```

## 10. Verification and call counts

The templates were checked against 112 confidence requests and 311 FullDist requests. The electric-vehicle clarification accounts for 33 of the FullDist requests. All system instructions and the four familiarity queries were also matched to the messages used during execution.

| Network | Model | Confidence calls | FullDist calls, including retries | Additional FullDist attempts | Maximum FullDist parsing attempt |
|---|---|---:|---:|---:|---:|
| `seismic` | `Qwen3.5-4B` | 24 | 24 | 5 | 3 |
| `seismic` | `Qwen3.5-9B` | 24 | 70 | 1 | 2 |
| `electricvehicle` | `Qwen3.5-4B` | 32 | 83 | 21 | 2 |
| `electricvehicle` | `Qwen3.5-9B` | 32 | 134 | 12 | 2 |

The runs comprise **112 confidence calls and 311 FullDist calls**, including **39 additional FullDist attempts**, plus **four familiarity queries**. Retry counts describe attempts to obtain a parsable response.

Verification establishes equivalence of the prompt text for the recorded conditions. Evaluation of the generated probabilities and the accepted responses is performed separately.

## 11. Experimental configuration

Both model sizes use the same base templates, confidence threshold, and RNM hyperparameters. Domain-specific context and persona instructions determine the textual differences between networks.

| Setting | Configuration |
|---|---|
| Models | Qwen3.5-4B and Qwen3.5-9B |
| Checkpoints | `Qwen/Qwen3.5-4B`; `Qwen/Qwen3.5-9B` |
| Hardware | NVIDIA Tesla T4 |
| Quantization | 4-bit NF4, double quantization, FP16 compute |
| Transformers | 5.19.0.dev0 |
| Thinking mode | Configured as disabled |
| Base seed | 42; request-specific seeds are reported with the examples |
| Repetitions | One network generation per model, network, and prompting condition |
| Confidence screening | Greedy decoding; temperature 0.0; maximum 1,536 output tokens |
| FullDist, first attempt | Sampling; temperature 0.1; top-p 1.0; maximum 2,048 output tokens |
| FullDist, retry | Sampling; temperature 0.1; top-p 1.0; maximum 4,096 output tokens |
| Confidence gate | Direct elicitation when row confidence is at least 0.80 |
| RNM spread | Default 0.16; fitting bounds [0.03, 0.45] |
| RNM weight regularization | 0.02 toward equal parent weights |

The exact loaded model commits and some inherited decoding defaults were not recorded. The settings above characterize the documented executions; they do not guarantee identical responses after a model, software, or hardware change.
