# Secondary Tables

Supplementary results for *Confidence-Guided Bayesian Network Elicitation with LLMs and Ranked Nodes*. The tables cover all 24 evaluated outputs, network-specific uniform baselines, error decompositions, row provenance, persona pooling, and smoothing sensitivity.

Each of the 16 primary conditions has one generation. The eight persona pools are deterministic combinations of those generations. Uniform baselines are repeated under both model sizes for ease of comparison; there are two distinct uniform networks.

## 1. Conditions and reporting conventions

The models are Qwen3.5-4B and Qwen3.5-9B, using checkpoints `Qwen/Qwen3.5-4B` and `Qwen/Qwen3.5-9B`. Generation uses a confidence threshold of 0.80, 4-bit NF4 with double quantization and FP16 computation on an NVIDIA Tesla T4. The base seed is 42. Detailed generation and evaluation settings are given in the [methodological description](methodological-details.md).

**Table S1. Condition labels**

| Label | seismic | electricvehicle |
| --- | --- | --- |
| C | Control, empty system instruction | Control, empty system instruction |
| P1 | Structural engineer | Electric vehicle systems engineer |
| P2 | Seismologist | Automotive fire safety and reliability specialist |
| P3 | Probabilistic risk analyst | Probabilistic risk analyst |
| Equal | Equal pool of P1–P3 | Equal pool of P1–P3 |
| Weighted | Configured pool of P1–P3 | Configured pool of P1–P3 |
| Uniform | 1/3 per child state | 1/2 per child state |

The configured pool uses P1/P2/P3 weights of 0.50/0.30/0.20, with seismic overrides of 0.15/0.70/0.15 for SeismicHazard and 0.70/0.10/0.20 for Fragility. Control is excluded from both pools.

Unless stated otherwise, KL uses base-2 logarithms and epsilon `1e-8`. **BN KL** denotes the reference-weighted sum of smoothed row divergences; **CPT KL** is the unweighted mean over all CPT rows. BN KL sums node contributions and is not divided by network size. Because the original reference parent marginals are held fixed during smoothing, this score is not generally the exact joint KL between two fully smoothed networks.

Lower values indicate smaller error, except the percentages within error tolerances, where higher is better. Values are rounded to four decimal places and percentages to one decimal place. Calculations use unrounded values. Bold in the primary-score tables identifies the lowest unrounded error in each column, including the uniform baseline. A tie after rounding need not be an exact tie.

## 2. Complete primary scores

**Table S2. seismic — Qwen3.5-4B**

| Condition | BN KL (bits) | CPT KL (bits) | MAE | RMSE | Mean TV |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 5.5247 | 0.6997 | 0.2231 | 0.2851 | 0.3346 |
| P1 | 4.1567 | 0.4216 | **0.1761** | **0.2269** | **0.2641** |
| P2 | 4.1447 | 0.5522 | 0.1973 | 0.2540 | 0.2959 |
| P3 | 5.0014 | 0.6748 | 0.2144 | 0.2774 | 0.3216 |
| Equal | **3.5521** | **0.4172** | 0.1847 | 0.2350 | 0.2770 |
| Weighted | 3.7597 | 0.4622 | 0.1904 | 0.2438 | 0.2856 |
| Uniform | 5.7130 | 0.4560 | 0.2046 | 0.2588 | 0.3069 |

**Table S3. seismic — Qwen3.5-9B**

| Condition | BN KL (bits) | CPT KL (bits) | MAE | RMSE | Mean TV |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 7.3764 | 0.7929 | **0.1767** | 0.2279 | **0.2651** |
| P1 | 8.0698 | 0.6877 | 0.1950 | 0.2463 | 0.2925 |
| P2 | 4.9287 | 0.4342 | 0.1779 | 0.2291 | 0.2668 |
| P3 | **4.1073** | 0.4263 | 0.1804 | 0.2275 | 0.2706 |
| Equal | 4.2477 | **0.4241** | 0.1783 | **0.2269** | 0.2674 |
| Weighted | 4.4610 | 0.4330 | 0.1805 | 0.2296 | 0.2707 |
| Uniform | 5.7130 | 0.4560 | 0.2046 | 0.2588 | 0.3069 |

**Table S4. electricvehicle — Qwen3.5-4B**

| Condition | BN KL (bits) | CPT KL (bits) | MAE | RMSE | Mean TV |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 9.1131 | 0.8839 | 0.4005 | 0.4767 | 0.4005 |
| P1 | 17.8188 | 0.9941 | 0.3934 | 0.4741 | 0.3934 |
| P2 | 11.4584 | 0.8578 | **0.3667** | 0.4483 | **0.3667** |
| P3 | 31.3402 | 1.1653 | 0.3947 | 0.4798 | 0.3947 |
| Equal | **5.6100** | 0.7895 | 0.3705 | 0.4540 | 0.3705 |
| Weighted | 5.8970 | 0.7956 | 0.3739 | 0.4553 | 0.3739 |
| Uniform | 12.0939 | **0.6087** | 0.4041 | **0.4140** | 0.4041 |

**Table S5. electricvehicle — Qwen3.5-9B**

| Condition | BN KL (bits) | CPT KL (bits) | MAE | RMSE | Mean TV |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | **10.6434** | 1.6441 | 0.4170 | 0.4892 | 0.4170 |
| P1 | 22.6972 | 2.1156 | 0.3992 | 0.5105 | 0.3992 |
| P2 | 13.1395 | 1.5482 | 0.3971 | 0.4897 | 0.3971 |
| P3 | 24.0429 | 2.9540 | 0.4277 | 0.5154 | 0.4277 |
| Equal | 11.8444 | 0.9435 | 0.3986 | 0.4798 | 0.3986 |
| Weighted | 11.5647 | 0.9370 | **0.3967** | 0.4815 | **0.3967** |
| Uniform | 12.0939 | **0.6087** | 0.4041 | **0.4140** | 0.4041 |

For electricvehicle, mean TV equals MAE. For seismic, mean TV equals 1.5 × MAE because every node has three states. These relationships follow from the metric definitions and do not provide independent evidence.

## 3. Supplementary error summaries

KL and Jensen–Shannon (JS) divergences are in bits. Hellinger is the mean row distance, without squaring; RNM fitting uses squared Hellinger instead. Median and P90 refer to the distribution across CPT rows in the first set of tables. In the scalar-error tables, they refer to individual probability entries. Tolerance percentages count entries with absolute error no greater than 0.05 or 0.10.

**Table S6. seismic — Qwen3.5-4B: row distances**

| Condition | Median KL | P90 KL | Median TV | Hellinger | Mean JS | P90 JS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.4303 | 1.5979 | 0.2991 | 0.2979 | 0.1429 | 0.3172 |
| P1 | 0.3373 | 0.7257 | 0.2611 | 0.2399 | 0.0920 | 0.1668 |
| P2 | 0.3952 | 1.2225 | 0.2913 | 0.2671 | 0.1143 | 0.2323 |
| P3 | 0.4286 | 1.5979 | 0.2991 | 0.2890 | 0.1357 | 0.3156 |
| Equal | 0.3653 | 0.7381 | 0.2785 | 0.2455 | 0.0964 | 0.1916 |
| Weighted | 0.3908 | 0.8040 | 0.2824 | 0.2540 | 0.1038 | 0.1968 |
| Uniform | 0.2609 | 1.1160 | 0.2667 | 0.2630 | 0.1249 | 0.3120 |

**Table S7. seismic — Qwen3.5-4B: scalar errors**

| Condition | MSE | Max. error | Median error | P90 error | ≤0.05 (%) | ≤0.10 (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.0813 | 0.8518 | 0.1805 | 0.4746 | 16.7 | 27.6 |
| P1 | 0.0515 | 0.6585 | 0.1518 | 0.3637 | 21.5 | 34.1 |
| P2 | 0.0645 | 0.7985 | 0.1718 | 0.4037 | 19.1 | 32.9 |
| P3 | 0.0769 | 0.8518 | 0.1729 | 0.4593 | 17.5 | 30.1 |
| Equal | 0.0552 | 0.6925 | 0.1585 | 0.3961 | 20.3 | 34.6 |
| Weighted | 0.0595 | 0.7508 | 0.1606 | 0.3991 | 20.7 | 35.0 |
| Uniform | 0.0670 | 0.6667 | 0.1667 | 0.4317 | 19.9 | 30.1 |

**Table S8. seismic — Qwen3.5-9B: row distances**

| Condition | Median KL | P90 KL | Median TV | Hellinger | Mean JS | P90 JS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.3764 | 0.8870 | 0.2585 | 0.2415 | 0.0951 | 0.1670 |
| P1 | 0.3924 | 1.4047 | 0.2811 | 0.2630 | 0.1079 | 0.2145 |
| P2 | 0.3885 | 0.7257 | 0.2720 | 0.2393 | 0.0936 | 0.1668 |
| P3 | 0.3485 | 0.7173 | 0.2679 | 0.2445 | 0.0932 | 0.1658 |
| Equal | 0.3616 | 0.7257 | 0.2693 | 0.2423 | 0.0924 | 0.1710 |
| Weighted | 0.3839 | 0.7257 | 0.2720 | 0.2450 | 0.0943 | 0.1898 |
| Uniform | 0.2609 | 1.1160 | 0.2667 | 0.2630 | 0.1249 | 0.3120 |

**Table S9. seismic — Qwen3.5-9B: scalar errors**

| Condition | MSE | Max. error | Median error | P90 error | ≤0.05 (%) | ≤0.10 (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.0519 | 0.6585 | 0.1518 | 0.3702 | 22.4 | 34.1 |
| P1 | 0.0607 | 0.6585 | 0.1585 | 0.4000 | 19.5 | 30.9 |
| P2 | 0.0525 | 0.6585 | 0.1518 | 0.3718 | 22.4 | 33.3 |
| P3 | 0.0518 | 0.6585 | 0.1585 | 0.3787 | 19.1 | 33.3 |
| Equal | 0.0515 | 0.6585 | 0.1518 | 0.3528 | 19.9 | 33.3 |
| Weighted | 0.0527 | 0.6585 | 0.1552 | 0.3620 | 21.1 | 31.7 |
| Uniform | 0.0670 | 0.6667 | 0.1667 | 0.4317 | 19.9 | 30.1 |

**Table S10. electricvehicle — Qwen3.5-4B: row distances**

| Condition | Median KL | P90 KL | Median TV | Hellinger | Mean JS | P90 JS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.5860 | 2.2773 | 0.4103 | 0.3509 | 0.2116 | 0.4946 |
| P1 | 0.7436 | 2.6771 | 0.4166 | 0.3554 | 0.2134 | 0.4946 |
| P2 | 0.6201 | 2.0268 | 0.3700 | 0.3375 | 0.1947 | 0.4820 |
| P3 | 0.5860 | 2.6771 | 0.4166 | 0.3509 | 0.2160 | 0.5027 |
| Equal | 0.5438 | 1.9779 | 0.4040 | 0.3306 | 0.1943 | 0.4844 |
| Weighted | 0.5645 | 1.9982 | 0.4040 | 0.3338 | 0.1955 | 0.4844 |
| Uniform | 0.6201 | 0.9327 | 0.4262 | 0.3545 | 0.1764 | 0.2842 |

**Table S11. electricvehicle — Qwen3.5-4B: scalar errors**

| Condition | MSE | Max. error | Median error | P90 error | ≤0.05 (%) | ≤0.10 (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.2273 | 0.9372 | 0.4103 | 0.7572 | 5.7 | 17.7 |
| P1 | 0.2247 | 0.9372 | 0.4166 | 0.7572 | 2.5 | 20.3 |
| P2 | 0.2010 | 0.9098 | 0.3700 | 0.7436 | 6.3 | 21.5 |
| P3 | 0.2302 | 0.9372 | 0.4166 | 0.7687 | 8.9 | 21.5 |
| Equal | 0.2061 | 0.8834 | 0.4040 | 0.7466 | 7.0 | 24.1 |
| Weighted | 0.2073 | 0.8834 | 0.4040 | 0.7466 | 3.8 | 20.3 |
| Uniform | 0.1714 | 0.5000 | 0.4262 | 0.4927 | 0.0 | 0.0 |

**Table S12. electricvehicle — Qwen3.5-9B: row distances**

| Condition | Median KL | P90 KL | Median TV | Hellinger | Mean JS | P90 JS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.5645 | 2.6938 | 0.4103 | 0.3650 | 0.2230 | 0.5066 |
| P1 | 0.6289 | 3.0076 | 0.3500 | 0.3598 | 0.2433 | 0.5984 |
| P2 | 0.7725 | 2.9468 | 0.4000 | 0.3638 | 0.2277 | 0.5869 |
| P3 | 0.8426 | 4.4324 | 0.4286 | 0.3955 | 0.2562 | 0.6336 |
| Equal | 0.6289 | 2.4331 | 0.4103 | 0.3586 | 0.2179 | 0.5363 |
| Weighted | 0.6289 | 2.3205 | 0.3670 | 0.3559 | 0.2176 | 0.5286 |
| Uniform | 0.6201 | 0.9327 | 0.4262 | 0.3545 | 0.1764 | 0.2842 |

**Table S13. electricvehicle — Qwen3.5-9B: scalar errors**

| Condition | MSE | Max. error | Median error | P90 error | ≤0.05 (%) | ≤0.10 (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| C | 0.2394 | 0.9262 | 0.4103 | 0.7726 | 5.1 | 13.9 |
| P1 | 0.2606 | 0.9772 | 0.3500 | 0.8289 | 13.9 | 27.8 |
| P2 | 0.2398 | 0.9500 | 0.4000 | 0.8226 | 6.3 | 21.5 |
| P3 | 0.2656 | 0.9900 | 0.4286 | 0.8326 | 5.1 | 13.9 |
| Equal | 0.2302 | 0.9467 | 0.4103 | 0.8021 | 6.3 | 16.5 |
| Weighted | 0.2318 | 0.9330 | 0.3670 | 0.7837 | 6.3 | 19.0 |
| Uniform | 0.1714 | 0.5000 | 0.4262 | 0.4927 | 0.0 | 0.0 |

## 4. Root and conditional error decomposition

Root KL and Conditional KL are additive components of BN KL, using the same reference weights and epsilon as the primary analysis. Root share is the percentage of total BN KL attributable to roots. Root mean KL and Conditional mean KL are unweighted means within the corresponding row subsets; they do not add to BN KL.

Each seismic output has four root rows and 78 conditional rows. Each electricvehicle output has 15 root rows and 64 conditional rows.

**Table S14. seismic — Qwen3.5-4B**

| Condition | Root KL | Conditional KL | Root share (%) | Root mean KL | Conditional mean KL |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 1.2721 | 4.2527 | 23.0 | 0.3180 | 0.7192 |
| P1 | 1.9725 | 2.1841 | 47.5 | 0.4931 | 0.4179 |
| P2 | 1.6472 | 2.4975 | 39.7 | 0.4118 | 0.5594 |
| P3 | 1.4376 | 3.5638 | 28.7 | 0.3594 | 0.6910 |
| Equal | 1.4076 | 2.1445 | 39.6 | 0.3519 | 0.4205 |
| Weighted | 1.5270 | 2.2327 | 40.6 | 0.3817 | 0.4664 |
| Uniform | 1.2721 | 4.4409 | 22.3 | 0.3180 | 0.4631 |

**Table S15. seismic — Qwen3.5-9B**

| Condition | Root KL | Conditional KL | Root share (%) | Root mean KL | Conditional mean KL |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 1.2721 | 6.1043 | 17.2 | 0.3180 | 0.8172 |
| P1 | 2.6738 | 5.3960 | 33.1 | 0.6685 | 0.6887 |
| P2 | 2.7399 | 2.1888 | 55.6 | 0.6850 | 0.4213 |
| P3 | 1.6651 | 2.4422 | 40.5 | 0.4163 | 0.4268 |
| Equal | 1.9576 | 2.2901 | 46.1 | 0.4894 | 0.4207 |
| Weighted | 2.1072 | 2.3537 | 47.2 | 0.5268 | 0.4282 |
| Uniform | 1.2721 | 4.4409 | 22.3 | 0.3180 | 0.4631 |

**Table S16. electricvehicle — Qwen3.5-4B**

| Condition | Root KL | Conditional KL | Root share (%) | Root mean KL | Conditional mean KL |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 4.0499 | 5.0632 | 44.4 | 0.2700 | 1.0278 |
| P1 | 12.7555 | 5.0632 | 71.6 | 0.8504 | 1.0278 |
| P2 | 8.3396 | 3.1187 | 72.8 | 0.5560 | 0.9285 |
| P3 | 26.2770 | 5.0632 | 83.8 | 1.7518 | 1.0278 |
| Equal | 1.8701 | 3.7399 | 33.3 | 0.1247 | 0.9453 |
| Weighted | 2.0836 | 3.8134 | 35.3 | 0.1389 | 0.9495 |
| Uniform | 5.2159 | 6.8780 | 43.1 | 0.3477 | 0.6699 |

**Table S17. electricvehicle — Qwen3.5-9B**

| Condition | Root KL | Conditional KL | Root share (%) | Root mean KL | Conditional mean KL |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 5.2159 | 5.4276 | 49.0 | 0.3477 | 1.9479 |
| P1 | 8.2614 | 14.4359 | 36.4 | 0.5508 | 2.4823 |
| P2 | 6.6513 | 6.4881 | 50.6 | 0.4434 | 1.8072 |
| P3 | 12.8598 | 11.1831 | 53.5 | 0.8573 | 3.4454 |
| Equal | 6.1808 | 5.6636 | 52.2 | 0.4121 | 1.0680 |
| Weighted | 6.0462 | 5.5185 | 52.3 | 0.4031 | 1.0621 |
| Uniform | 5.2159 | 6.8780 | 43.1 | 0.3477 | 0.6699 |

## 5. Routing, confidence, and anchor coverage

Counts refer to the 16 primary generations. **Root FD** is a directly elicited root distribution; **Conditional FD** is a directly elicited conditional row that also serves as an anchor. **Anchored RNM** completes a row using a fit to that node's anchors. **Unanchored RNM** uses equal parent weights, dispersion 0.16, and the screened qualitative directions.

Conditional direct (%) divides the Conditional FD count by 78 for seismic or 64 for electricvehicle. Confidence summaries cover conditional rows only. **Fitted nodes** counts nodes with at least one anchor; **Completion fits** counts those whose parameters actually supply at least one completed row. **All-direct fits** counts fitted nodes for which every row remains directly elicited. Pools have no single generation route and are excluded.

**Table S18. seismic — Qwen3.5-4B: row counts**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM | Conditional direct (%) |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 4 | 3 | 0 | 75 | 3.8 |
| P1 | 4 | 0 | 0 | 78 | 0.0 |
| P2 | 4 | 0 | 0 | 78 | 0.0 |
| P3 | 4 | 0 | 0 | 78 | 0.0 |

**Table S19. seismic — Qwen3.5-4B: confidence and fits**

| Condition | Mean confidence | Median confidence | Fitted nodes | Completion fits | All-direct fits |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 0.0346 | 0.0000 | 1 | 0 | 1 |
| P1 | 0.0192 | 0.0000 | 0 | 0 | 0 |
| P2 | 0.0000 | 0.0000 | 0 | 0 | 0 |
| P3 | 0.0000 | 0.0000 | 0 | 0 | 0 |

**Table S20. seismic — Qwen3.5-9B: row counts**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM | Conditional direct (%) |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 4 | 12 | 0 | 66 | 15.4 |
| P1 | 4 | 19 | 2 | 57 | 24.4 |
| P2 | 4 | 12 | 0 | 66 | 15.4 |
| P3 | 4 | 10 | 2 | 66 | 12.8 |

**Table S21. seismic — Qwen3.5-9B: confidence and fits**

| Condition | Mean confidence | Median confidence | Fitted nodes | Completion fits | All-direct fits |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 0.1462 | 0.0000 | 2 | 0 | 2 |
| P1 | 0.2481 | 0.0000 | 3 | 1 | 2 |
| P2 | 0.1462 | 0.0000 | 2 | 0 | 2 |
| P3 | 0.1385 | 0.0000 | 2 | 1 | 1 |

**Table S22. electricvehicle — Qwen3.5-4B: row counts**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM | Conditional direct (%) |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 15 | 0 | 0 | 64 | 0.0 |
| P1 | 15 | 0 | 0 | 64 | 0.0 |
| P2 | 15 | 2 | 6 | 56 | 3.1 |
| P3 | 15 | 0 | 0 | 64 | 0.0 |

**Table S23. electricvehicle — Qwen3.5-4B: confidence and fits**

| Condition | Mean confidence | Median confidence | Fitted nodes | Completion fits | All-direct fits |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 0.1172 | 0.0000 | 0 | 0 | 0 |
| P1 | 0.0938 | 0.0000 | 0 | 0 | 0 |
| P2 | 0.1453 | 0.0000 | 2 | 2 | 0 |
| P3 | 0.0000 | 0.0000 | 0 | 0 | 0 |

**Table S24. electricvehicle — Qwen3.5-9B: row counts**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM | Conditional direct (%) |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 15 | 4 | 0 | 60 | 6.2 |
| P1 | 15 | 23 | 9 | 32 | 35.9 |
| P2 | 15 | 18 | 6 | 40 | 28.1 |
| P3 | 15 | 17 | 7 | 40 | 26.6 |

**Table S25. electricvehicle — Qwen3.5-9B: confidence and fits**

| Condition | Mean confidence | Median confidence | Fitted nodes | Completion fits | All-direct fits |
| --- | ---: | ---: | ---: | ---: | ---: |
| C | 0.0594 | 0.0000 | 1 | 0 | 1 |
| P1 | 0.4506 | 0.1500 | 5 | 2 | 3 |
| P2 | 0.3141 | 0.0000 | 5 | 2 | 3 |
| P3 | 0.3805 | 0.1500 | 5 | 2 | 3 |

Across all primary generations, the 1,136 conditional rows comprise 120 directly elicited rows (10.6%), 32 anchor-fitted RNM rows (2.8%), and 984 unanchored RNM rows (86.6%). There are also 152 root FullDist rows. Of 28 fitted conditional nodes, ten supply completed rows and 18 are fully direct. No final row is labeled as a uniform error fallback.

## 6. Weighted error by construction source

Each cell reports **BN KL contribution in bits (share of that generation’s total BN KL)**. The four contributions sum to the score in Section 2. Percentages can differ slightly from 100% after rounding. An absent source has zero contribution. Error shares should be read together with row counts in Section 5; they are neither row fractions nor mean route errors.

**Table S26. seismic — Qwen3.5-4B**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM |
| --- | ---: | ---: | ---: | ---: |
| C | 1.2721 (23.0%) | 0.9084 (16.4%) | 0.0000 (0.0%) | 3.3443 (60.5%) |
| P1 | 1.9725 (47.5%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 2.1841 (52.5%) |
| P2 | 1.6472 (39.7%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 2.4975 (60.3%) |
| P3 | 1.4376 (28.7%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 3.5638 (71.3%) |

**Table S27. seismic — Qwen3.5-9B**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM |
| --- | ---: | ---: | ---: | ---: |
| C | 1.2721 (17.2%) | 4.5069 (61.1%) | 0.0000 (0.0%) | 1.5975 (21.7%) |
| P1 | 2.6738 (33.1%) | 3.8040 (47.1%) | 0.4542 (5.6%) | 1.1378 (14.1%) |
| P2 | 2.7399 (55.6%) | 0.5913 (12.0%) | 0.0000 (0.0%) | 1.5975 (32.4%) |
| P3 | 1.6651 (40.5%) | 0.3905 (9.5%) | 0.4542 (11.1%) | 1.5975 (38.9%) |

**Table S28. electricvehicle — Qwen3.5-4B**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM |
| --- | ---: | ---: | ---: | ---: |
| C | 4.0499 (44.4%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 5.0632 (55.6%) |
| P1 | 12.7555 (71.6%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 5.0632 (28.4%) |
| P2 | 8.3396 (72.8%) | 0.0967 (0.8%) | 0.8157 (7.1%) | 2.2064 (19.3%) |
| P3 | 26.2770 (83.8%) | 0.0000 (0.0%) | 0.0000 (0.0%) | 5.0632 (16.2%) |

**Table S29. electricvehicle — Qwen3.5-9B**

| Condition | Root FD | Conditional FD | Anchored RNM | Unanchored RNM |
| --- | ---: | ---: | ---: | ---: |
| C | 5.2159 (49.0%) | 2.6482 (24.9%) | 0.0000 (0.0%) | 2.7794 (26.1%) |
| P1 | 8.2614 (36.4%) | 13.2230 (58.3%) | 0.4426 (2.0%) | 0.7702 (3.4%) |
| P2 | 6.6513 (50.6%) | 5.3103 (40.4%) | 0.1934 (1.5%) | 0.9844 (7.5%) |
| P3 | 12.8598 (53.5%) | 9.4170 (39.2%) | 1.1460 (4.8%) | 0.6201 (2.6%) |

## 7. Changes produced by persona pooling

Signed percentage change is calculated as `100 × (pool error / comparator error − 1)`. A negative value is a reduction in error; a positive value is an increase. These percentages describe changes in the specified metric and should not be interpreted as accuracy percentages. Root and conditional changes use their own control-component denominators.

**Table S30. Pool versus control**

| Network / model | Pool | BN KL Δ (%) | Root KL Δ (%) | Conditional KL Δ (%) | CPT KL Δ (%) |
| --- | --- | ---: | ---: | ---: | ---: |
| seismic / 4B | Equal | -35.7 | +10.7 | -49.6 | -40.4 |
| seismic / 4B | Weighted | -31.9 | +20.0 | -47.5 | -33.9 |
| seismic / 9B | Equal | -42.4 | +53.9 | -62.5 | -46.5 |
| seismic / 9B | Weighted | -39.5 | +65.7 | -61.4 | -45.4 |
| electricvehicle / 4B | Equal | -38.4 | -53.8 | -26.1 | -10.7 |
| electricvehicle / 4B | Weighted | -35.3 | -48.6 | -24.7 | -10.0 |
| electricvehicle / 9B | Equal | +11.3 | +18.5 | +4.3 | -42.6 |
| electricvehicle / 9B | Weighted | +8.7 | +15.9 | +1.7 | -43.0 |

**Table S31. Pool versus uniform baseline**

| Network / model | Pool | BN KL Δ (%) | CPT KL Δ (%) | MAE Δ (%) | RMSE Δ (%) |
| --- | --- | ---: | ---: | ---: | ---: |
| seismic / 4B | Equal | -37.8 | -8.5 | -9.8 | -9.2 |
| seismic / 4B | Weighted | -34.2 | +1.4 | -7.0 | -5.8 |
| seismic / 9B | Equal | -25.6 | -7.0 | -12.9 | -12.3 |
| seismic / 9B | Weighted | -21.9 | -5.0 | -11.8 | -11.3 |
| electricvehicle / 4B | Equal | -53.6 | +29.7 | -8.3 | +9.7 |
| electricvehicle / 4B | Weighted | -51.2 | +30.7 | -7.5 | +10.0 |
| electricvehicle / 9B | Equal | -2.1 | +55.0 | -1.4 | +15.9 |
| electricvehicle / 9B | Weighted | -4.4 | +53.9 | -1.8 | +16.3 |

## 8. Sensitivity to KL smoothing

The following tables vary epsilon while retaining the estimated probabilities, pooling weights, and original reference parent-marginal weights. They are numerical reevaluations of the same outputs, not new language-model generations. The `1e-8` column corresponds to the primary analysis. Four decimal places are retained throughout; small changes can disappear after rounding.

**Table S32. seismic — Qwen3.5-4B: BN KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 5.5247 | 5.5247 | 5.5245 | 5.5069 |
| P1 | 4.1567 | 4.1567 | 4.1565 | 4.1422 |
| P2 | 4.1447 | 4.1447 | 4.1445 | 4.1291 |
| P3 | 5.0014 | 5.0014 | 5.0011 | 4.9798 |
| Equal | 3.5521 | 3.5521 | 3.5519 | 3.5382 |
| Weighted | 3.7597 | 3.7597 | 3.7596 | 3.7456 |
| Uniform | 5.7130 | 5.7130 | 5.7129 | 5.7062 |

**Table S33. seismic — Qwen3.5-4B: mean CPT KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 0.6997 | 0.6997 | 0.6996 | 0.6975 |
| P1 | 0.4216 | 0.4216 | 0.4216 | 0.4207 |
| P2 | 0.5522 | 0.5522 | 0.5522 | 0.5507 |
| P3 | 0.6748 | 0.6748 | 0.6748 | 0.6726 |
| Equal | 0.4172 | 0.4172 | 0.4172 | 0.4163 |
| Weighted | 0.4622 | 0.4622 | 0.4622 | 0.4613 |
| Uniform | 0.4560 | 0.4560 | 0.4560 | 0.4555 |

**Table S34. seismic — Qwen3.5-9B: BN KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 8.5812 | 7.3764 | 6.1716 | 4.9579 |
| P1 | 8.9569 | 8.0698 | 7.1826 | 6.2895 |
| P2 | 4.9287 | 4.9287 | 4.9285 | 4.9172 |
| P3 | 4.1073 | 4.1073 | 4.1071 | 4.0964 |
| Equal | 4.2477 | 4.2477 | 4.2476 | 4.2348 |
| Weighted | 4.4610 | 4.4610 | 4.4608 | 4.4456 |
| Uniform | 5.7130 | 5.7130 | 5.7129 | 5.7062 |

**Table S35. seismic — Qwen3.5-9B: mean CPT KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 0.9063 | 0.7929 | 0.6794 | 0.5652 |
| P1 | 0.7517 | 0.6877 | 0.6237 | 0.5590 |
| P2 | 0.4342 | 0.4342 | 0.4342 | 0.4333 |
| P3 | 0.4263 | 0.4263 | 0.4263 | 0.4254 |
| Equal | 0.4241 | 0.4241 | 0.4241 | 0.4231 |
| Weighted | 0.4330 | 0.4330 | 0.4330 | 0.4320 |
| Uniform | 0.4560 | 0.4560 | 0.4560 | 0.4555 |

**Table S36. electricvehicle — Qwen3.5-4B: BN KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 9.1131 | 9.1131 | 9.1130 | 9.1029 |
| P1 | 19.8119 | 17.8188 | 15.8254 | 13.8167 |
| P2 | 12.1560 | 11.4584 | 10.7604 | 10.0282 |
| P3 | 37.2865 | 31.3402 | 25.3938 | 19.4357 |
| Equal | 5.6100 | 5.6100 | 5.6099 | 5.6014 |
| Weighted | 5.8970 | 5.8970 | 5.8969 | 5.8881 |
| Uniform | 12.0939 | 12.0939 | 12.0937 | 12.0833 |

**Table S37. electricvehicle — Qwen3.5-4B: mean CPT KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 0.8839 | 0.8839 | 0.8839 | 0.8833 |
| P1 | 1.0193 | 0.9941 | 0.9689 | 0.9429 |
| P2 | 0.8666 | 0.8578 | 0.8489 | 0.8392 |
| P3 | 1.2405 | 1.1653 | 1.0900 | 1.0141 |
| Equal | 0.7895 | 0.7895 | 0.7895 | 0.7889 |
| Weighted | 0.7956 | 0.7956 | 0.7956 | 0.7950 |
| Uniform | 0.6087 | 0.6087 | 0.6087 | 0.6083 |

**Table S38. electricvehicle — Qwen3.5-9B: BN KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 11.3266 | 10.6434 | 9.9602 | 9.2694 |
| P1 | 25.2511 | 22.6972 | 20.1424 | 17.4952 |
| P2 | 13.2184 | 13.1395 | 13.0585 | 12.8302 |
| P3 | 25.6358 | 24.0429 | 22.4472 | 20.6218 |
| Equal | 11.8444 | 11.8444 | 11.8440 | 11.8133 |
| Weighted | 11.5647 | 11.5647 | 11.5643 | 11.5327 |
| Uniform | 12.0939 | 12.0939 | 12.0937 | 12.0833 |

**Table S39. electricvehicle — Qwen3.5-9B: mean CPT KL in bits**

| Condition | 1e-10 | 1e-8 | 1e-6 | 1e-4 |
| --- | ---: | ---: | ---: | ---: |
| C | 1.8599 | 1.6441 | 1.4282 | 1.2118 |
| P1 | 2.4195 | 2.1156 | 1.8116 | 1.5060 |
| P2 | 1.7007 | 1.5482 | 1.3957 | 1.2397 |
| P3 | 3.4939 | 2.9540 | 2.4141 | 1.8708 |
| Equal | 0.9435 | 0.9435 | 0.9435 | 0.9425 |
| Weighted | 0.9370 | 0.9370 | 0.9370 | 0.9361 |
| Uniform | 0.6087 | 0.6087 | 0.6087 | 0.6083 |

**Table S40. Best electric-vehicle mean CPT KL versus uniform at each epsilon**

| Epsilon | Uniform KL | Best model / condition | Best output KL | Excess (bits) | Excess (%) |
| --- | ---: | --- | ---: | ---: | ---: |
| 1e-10 | 0.6087 | 4B / Equal | 0.7895 | 0.1808 | +29.7 |
| 1e-08 | 0.6087 | 4B / Equal | 0.7895 | 0.1808 | +29.7 |
| 1e-06 | 0.6087 | 4B / Equal | 0.7895 | 0.1808 | +29.7 |
| 1e-04 | 0.6083 | 4B / Equal | 0.7889 | 0.1807 | +29.7 |

All twelve electric-vehicle outputs have higher mean CPT KL than the uniform baseline at every tested epsilon. This statement concerns the unweighted row mean; the reference-weighted score can favor the same output over uniform.

## 9. Exact boundary probabilities before smoothing

Zero entries and One entries count scalar probabilities that equal 0 or 1 exactly in the final, unsmoothed CPTs. Boundary rows counts rows containing at least one such entry. This includes root and conditional rows. Near-zero RNM values are not counted as exact zeros. The denominators are 82 rows and 246 entries per seismic output, and 79 rows and 158 entries per electricvehicle output.

**Table S41. seismic — Qwen3.5-4B**

| Condition | Zero entries | One entries | Boundary rows | Rows (%) |
| --- | ---: | ---: | ---: | ---: |
| C | 0 | 0 | 0 | 0.0 |
| P1 | 0 | 0 | 0 | 0.0 |
| P2 | 0 | 0 | 0 | 0.0 |
| P3 | 0 | 0 | 0 | 0.0 |
| Equal | 0 | 0 | 0 | 0.0 |
| Weighted | 0 | 0 | 0 | 0.0 |
| Uniform | 0 | 0 | 0 | 0.0 |

**Table S42. seismic — Qwen3.5-9B**

| Condition | Zero entries | One entries | Boundary rows | Rows (%) |
| --- | ---: | ---: | ---: | ---: |
| C | 11 | 5 | 6 | 7.3 |
| P1 | 7 | 3 | 4 | 4.9 |
| P2 | 4 | 2 | 2 | 2.4 |
| P3 | 1 | 0 | 1 | 1.2 |
| Equal | 0 | 0 | 0 | 0.0 |
| Weighted | 0 | 0 | 0 | 0.0 |
| Uniform | 0 | 0 | 0 | 0.0 |

**Table S43. electricvehicle — Qwen3.5-4B**

| Condition | Zero entries | One entries | Boundary rows | Rows (%) |
| --- | ---: | ---: | ---: | ---: |
| C | 0 | 0 | 0 | 0.0 |
| P1 | 2 | 2 | 2 | 2.5 |
| P2 | 1 | 1 | 1 | 1.3 |
| P3 | 1 | 1 | 1 | 1.3 |
| Equal | 0 | 0 | 0 | 0.0 |
| Weighted | 0 | 0 | 0 | 0.0 |
| Uniform | 0 | 0 | 0 | 0.0 |

**Table S44. electricvehicle — Qwen3.5-9B**

| Condition | Zero entries | One entries | Boundary rows | Rows (%) |
| --- | ---: | ---: | ---: | ---: |
| C | 4 | 4 | 4 | 5.1 |
| P1 | 6 | 6 | 6 | 7.6 |
| P2 | 3 | 3 | 3 | 3.8 |
| P3 | 10 | 10 | 10 | 12.7 |
| Equal | 0 | 0 | 0 | 0.0 |
| Weighted | 0 | 0 | 0 | 0.0 |
| Uniform | 0 | 0 | 0 | 0.0 |

The eight primary electric-vehicle generations contain 27 boundary rows, corresponding to the accepted FullDist distributions selected for the targeted response audit. The counts above describe the retained numerical outputs; they do not classify whether a response endorsed, rejected, or merely illustrated the accepted tuple. Pooling results describe these particular outputs and do not establish that a linear pool always removes zeros or ones.

## 10. Node contributions to weighted BN KL

Each entry is the node's sum of row KL divergences weighted by the reference probability of the parent configuration, in bits. Root nodes contribute their single row divergence with weight one. Within each column, node contributions sum to the full BN KL score. Node names are shown in alphabetical order. Totals use unrounded values.

**Table S45. seismic — Qwen3.5-4B**

| Node | C | P1 | P2 | P3 | Equal | Weighted |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| ConstructionQuality | 0.4282 | 0.8867 | 0.4282 | 0.4282 | 0.5520 | 0.6231 |
| Distance | 0.0000 | 0.0001 | 0.0001 | 0.1656 | 0.0170 | 0.0066 |
| Fragility | 0.3215 | 0.3215 | 0.3215 | 0.3215 | 0.3215 | 0.3215 |
| LiveLoad | 0.6630 | 0.9781 | 0.6630 | 0.6630 | 0.7538 | 0.8042 |
| Magnitude | 0.1809 | 0.1076 | 0.5558 | 0.1809 | 0.0849 | 0.0930 |
| SeismicHazard | 0.7775 | 0.4641 | 0.7775 | 0.7775 | 0.4653 | 0.5846 |
| SeismicRisk | 0.4597 | 0.4597 | 0.4597 | 0.4597 | 0.4597 | 0.4597 |
| ShakingIntensity | 1.4335 | 0.3672 | 0.3672 | 1.4335 | 0.3263 | 0.2953 |
| StrengthDegradation | 0.9084 | 0.2195 | 0.2195 | 0.2195 | 0.2195 | 0.2195 |
| Vulnerability | 0.3521 | 0.3521 | 0.3521 | 0.3521 | 0.3521 | 0.3521 |
| **Total** | **5.5247** | **4.1567** | **4.1447** | **5.0014** | **3.5521** | **3.7597** |

**Table S46. seismic — Qwen3.5-9B**

| Node | C | P1 | P2 | P3 | Equal | Weighted |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| ConstructionQuality | 0.4282 | 0.8867 | 1.5577 | 0.6473 | 0.9538 | 0.9771 |
| Distance | 0.0000 | 0.0779 | 0.0000 | 0.1938 | 0.0528 | 0.0480 |
| Fragility | 0.3215 | 0.3215 | 0.3215 | 0.3215 | 0.3215 | 0.3215 |
| LiveLoad | 0.6630 | 1.5191 | 0.6431 | 0.6431 | 0.8552 | 0.9839 |
| Magnitude | 0.1809 | 0.1902 | 0.5391 | 0.1809 | 0.0958 | 0.0982 |
| SeismicHazard | 0.4641 | 0.4641 | 0.4641 | 0.4641 | 0.4641 | 0.4641 |
| SeismicRisk | 0.4597 | 3.1659 | 0.4597 | 0.4597 | 0.4896 | 0.5393 |
| ShakingIntensity | 4.4419 | 0.5799 | 0.3549 | 0.3323 | 0.2695 | 0.2728 |
| StrengthDegradation | 0.0650 | 0.5124 | 0.2365 | 0.5124 | 0.3933 | 0.4039 |
| Vulnerability | 0.3521 | 0.3521 | 0.3521 | 0.3521 | 0.3521 | 0.3521 |
| **Total** | **7.3764** | **8.0698** | **4.9287** | **4.1073** | **4.2477** | **4.4610** |

**Table S47. electricvehicle — Qwen3.5-4B**

| Node | C | P1 | P2 | P3 | Equal | Weighted |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| ACF | 0.4706 | 0.0544 | 0.0544 | 0.0138 | 0.0363 | 0.0428 |
| AM | 0.0659 | 1.0602 | 1.4007 | 0.0659 | 0.1235 | 0.2793 |
| AWE | 0.5001 | 2.4234 | 0.2438 | 0.0098 | 0.0356 | 0.0845 |
| BEP | 0.5310 | 0.5310 | 0.5289 | 0.0037 | 0.0473 | 0.1240 |
| BF | 0.1068 | 0.4977 | 0.4977 | 0.4977 | 0.4977 | 0.4977 |
| BO | 0.2985 | 4.3479 | 0.1420 | 0.2985 | 0.0001 | 0.0311 |
| CBI | 0.2380 | 0.2380 | 0.2367 | 0.2380 | 0.2373 | 0.2374 |
| CEF | 0.0177 | 0.2730 | 0.2730 | 0.2730 | 0.2730 | 0.2730 |
| CI | 0.7302 | 2.5522 | 1.5756 | 0.5012 | 0.0017 | 0.0484 |
| DTH | 0.4706 | 0.0544 | 0.0544 | 0.1164 | 0.0703 | 0.0635 |
| EC | 0.1860 | 0.1860 | 0.1860 | 0.1860 | 0.1860 | 0.1860 |
| ECF | 0.0067 | 0.0688 | 0.3189 | 0.0024 | 0.0324 | 0.0516 |
| FFE | 0.0177 | 0.2149 | 0.0000 | 0.1013 | 0.0121 | 0.0469 |
| HF | 0.4626 | 0.4626 | 0.6756 | 0.4626 | 0.4811 | 0.4748 |
| IS | 0.0126 | 0.0354 | 2.3058 | 23.3004 | 0.2306 | 0.0704 |
| ISC | 0.1321 | 0.1321 | 0.1321 | 0.1321 | 0.1321 | 0.1321 |
| MMA | 0.5310 | 0.2084 | 0.2084 | 0.5310 | 0.0310 | 0.0005 |
| OFE | 3.2788 | 3.2788 | 1.1225 | 3.2788 | 1.9377 | 2.0174 |
| REI | 0.1276 | 0.1276 | 0.1276 | 0.1276 | 0.1276 | 0.1276 |
| SBB | 0.1585 | 0.3795 | 0.4554 | 0.5565 | 0.4554 | 0.4305 |
| SCB | 0.3020 | 0.3020 | 0.3020 | 0.3020 | 0.3020 | 0.3020 |
| TLD | 0.1319 | 0.0544 | 0.2807 | 0.0054 | 0.0232 | 0.0394 |
| VFD | 0.3361 | 0.3361 | 0.3361 | 0.3361 | 0.3361 | 0.3361 |
| **Total** | **9.1131** | **17.8188** | **11.4584** | **31.3402** | **5.6100** | **5.8970** |

**Table S48. electricvehicle — Qwen3.5-9B**

| Node | C | P1 | P2 | P3 | Equal | Weighted |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| ACF | 0.4706 | 0.0544 | 0.0544 | 0.2807 | 0.0904 | 0.0739 |
| AM | 0.0659 | 0.6267 | 0.0659 | 0.3274 | 0.0672 | 0.0948 |
| AWE | 0.5001 | 0.0098 | 0.0895 | 1.1026 | 0.2458 | 0.1363 |
| BEP | 0.5310 | 0.2084 | 0.5310 | 0.0484 | 0.0387 | 0.0237 |
| BF | 0.1068 | 1.1764 | 1.1764 | 2.1972 | 1.3329 | 1.2634 |
| BO | 0.2985 | 1.1932 | 0.1796 | 0.1796 | 0.2681 | 0.3345 |
| CBI | 0.2380 | 0.8564 | 0.1912 | 0.8564 | 0.2091 | 0.2122 |
| CEF | 0.2220 | 0.2730 | 0.2730 | 1.5152 | 0.3853 | 0.3340 |
| CI | 0.0420 | 1.5756 | 1.2047 | 1.5756 | 1.4209 | 1.4345 |
| DTH | 0.4706 | 0.0544 | 0.2807 | 0.3943 | 0.1619 | 0.1209 |
| EC | 0.1860 | 0.4334 | 0.1860 | 0.1860 | 0.2307 | 0.2617 |
| ECF | 0.4426 | 0.2016 | 0.0688 | 0.0688 | 0.0975 | 0.1160 |
| FFE | 0.3902 | 0.3902 | 0.0000 | 0.3902 | 0.1905 | 0.2074 |
| HF | 0.4626 | 8.9256 | 1.1396 | 1.7421 | 0.4935 | 0.6634 |
| IS | 0.5154 | 0.5631 | 0.2259 | 0.5631 | 0.3569 | 0.3684 |
| ISC | 0.1321 | 0.1321 | 0.4963 | 0.1321 | 0.2327 | 0.2219 |
| MMA | 0.5310 | 0.0298 | 0.2084 | 2.1885 | 0.1216 | 0.0837 |
| OFE | 1.1225 | 3.1837 | 3.5441 | 6.0314 | 3.5576 | 3.2847 |
| REI | 2.6482 | 0.2665 | 0.2360 | 0.2364 | 0.2274 | 0.2353 |
| SBB | 0.1585 | 1.8503 | 1.2277 | 0.9630 | 1.2029 | 1.3135 |
| SCB | 0.3020 | 0.3020 | 0.3020 | 0.3020 | 0.3020 | 0.3020 |
| TLD | 0.4706 | 0.0544 | 1.0653 | 1.0653 | 0.2002 | 0.1411 |
| VFD | 0.3361 | 0.3361 | 0.3928 | 1.6967 | 0.4105 | 0.3373 |
| **Total** | **10.6434** | **22.6972** | **13.1395** | **24.0429** | **11.8444** | **11.5647** |

## 11. Recorded model requests

Accepted FD rows counts the final FullDist distributions retained in the primary outputs. Additional FD attempts are parsing retries beyond those accepted rows. Total requests includes confidence and all FullDist attempts, excluding the preliminary familiarity query. Max. FD attempt is the highest observed parsing-attempt index, not the configured retry limit.

**Table S49. seismic**

| Model | Confidence | Accepted FD rows | FD requests | Additional FD attempts | Total requests | Max. FD attempt |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Qwen3.5-4B | 24 | 19 | 24 | 5 | 48 | 3 |
| Qwen3.5-9B | 24 | 69 | 70 | 1 | 94 | 2 |

**Table S50. electricvehicle**

| Model | Confidence | Accepted FD rows | FD requests | Additional FD attempts | Total requests | Max. FD attempt |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Qwen3.5-4B | 32 | 62 | 83 | 21 | 115 | 2 |
| Qwen3.5-9B | 32 | 122 | 134 | 12 | 166 | 2 |

There are 112 confidence requests and 311 FullDist requests, including 39 additional FullDist attempts: 423 generation requests in total. Four separate familiarity queries bring the recorded overall total to 427. Confidence requests all succeed on the first parsing attempt. Pooling and numerical evaluation require no further language-model requests.

## 12. Reading the comparisons

The tables describe one primary generation per condition. CPT rows, retry attempts, personas, and derived pools do not provide independent repeated runs, and no confidence intervals or significance tests across random seeds are inferred from them. Reference agreement is evaluated against the bnRep parameterization rather than observed outcomes.

Default-epsilon scores and supplementary summaries retain the recorded evaluation values. Uniform baselines, component totals, percentage changes, boundary counts, and sensitivity values are calculated from the aligned probabilities and fixed reference weights. The same state-label alignment is used throughout, including the electricvehicle permutation between generated `no, yes` and reference `yes, no` orders. Rounding is applied only for display.
