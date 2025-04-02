## 1. LLaMA 3
### Experimental Setup

| **Aspect**                  | **Details**                                                                                           |
|-----------------------------|-------------------------------------------------------------------------------------------------------|
| **Initial Setup**           | Followed the experimental settings from the original paper. Performed SFT on UltraChat-200k to obtain a preliminary instruction-tuned model. |
| **Preference Optimization** | Conducted on UltraFeedback dataset, using the SFT model as the starting point.                        |
| **Fine-Tuning Parameters**  | Learning rate: 2e-5, Batch size: 64, Max sequence length: 2048, Cosine learning rate schedule with 10% warmup steps for 1 epoch. |
| **Hardware**                | Tested memory usage on 2 H100 GPUs with ZeRO3.                                                       |

## Baseline Selection

| **Baseline** | **Description**                                                                                       |
|--------------|-------------------------------------------------------------------------------------------------------|
| **DPO**      | Theoretical foundation of this work; widely adopted, requires only reference and policy models for training. |
| **TDPO**     | Extends DPO to token level with a token-level objective and chosen-rejected KL divergence term to control output diversity. |
| **SimPO**    | Simplifies DPO by eliminating the reference model and includes measures to mitigate length reward hacking. |

### Benchmark Selection

| **Benchmark Task**                          | **Details**                                                                                           |
|---------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **Instruction-Following Datasets**          | Selected AlpacaEval 2, Arena-Hard, and MT-Bench, following the original paper’s settings.             |
| **Diversity Control Metrics**               | Evaluated KL Divergence Margin, Feature-level MSE Loss, and Diversity (Entropy) on UltraFeedback dataset. |
| **Sampling Temperature Impact**             | Measured winning rate on AlpacaEval 2 across different temperatures (aligned with Figure 3, right panel, in the paper). |
| **GPU Memory Consumption**                  | Recorded peak memory usage during training, aligning with Hardware Efficiency of FPO (Section 5.1).   |


### Experimental Results:

| Method v.s. FPO | AlpacaEval 2 | Arena-Hard | MT-Bench |
| --- | --- | --- | --- |
| SFT | 59.0% | 58.1% | +0.3 |
| DPO | 54.2% | 59.0% | +0.3 |
| SimPO | 55.1% | 52.2% | +0.4 |
| TDPO | 54.7% | 51.0% | +0.2 |
| FPO | 50.0% | 50.0% | 0.0 |

Winning rate, 0.56 means there is 56% of cases FPO is winning (the larger the better)

**KL Divergence and MSE Loss**

| Metric | SFT | DPO | SimPO | TDPO | FPO |
| --- | --- | --- | --- | --- | --- |
| Training KL Divergence Margin (avg $\times 10^{-2}$) | - | 4.1 | 4.3 | **2.2** | 2.3 |
| Testing KL Divergence Margin (avg $\times 10^{-2}$) | - | 4.6 | 4.4 | 2.4 | **2.3** |
| Feature-level MSE Loss | - | 0.12 | 0.11 | 0.04 | **0.01** |
| Diversity (Entropy) | 1.50 | 1.58 | 1.47 | 1.69 | **1.79** |

Training KL Divergence Margin, Testing KL Divergence Margin, and Feature-level MSE Loss: The lower the better.

Diversity (Entropy): The higher the better.

**Winning rate of AlpacaEval 2 in different temperatures.**

| Temperature | SFT | DPO | SimPO | TDPO | FPO |
| --- | --- | --- | --- | --- | --- |
| 0.3 | 62.0% | 58.0% | 59.0% | 57.5% | 50.0% |
| 0.5 | 60.5% | 56.5% | 57.0% | 56.0% | 50.0% |
| 0.7 | 59.0% | 54.2% | 55.1% | 54.7% | 50.0% |
| 0.9 | 57.5% | 53.0% | 54.0% | 53.0% | 50.0% |


**GPU Memory Consumption (Peak Memory in GB)**

| Method | Peak Memory (GB) |
| --- | --- |
| SFT | 54 |
| DPO | 66 |
| SimPO | 55 |
| TDPO | 67 |
| FPO | 58 |

## 2. MLSAE

**Gemma-2 2b** 

| Method v.s. FPO | AlpacaEval 2 | Arena-Hard | MT-Bench |
| --- | --- | --- | --- |
| SFT | 59.8% | 58.4% | +0.4 |
| DPO | 56.2% | 56.4% | +0.3 |
| SimPO | 56.2% | 54.8% | +0.4 |
| TDPO | 56.8% | 59.0% | +0.2 |
| FPO | 50.0% | 50.0% | 0.0 |

**Gemma-2 9b** 

| Method v.s. FPO | AlpacaEval 2 | Arena-Hard | MT-Bench |
| --- | --- | --- | --- |
| SFT | 58.4% | 56.8% | +0.5 |
| DPO | 54.8% | 57.0% | +0.4 |
| SimPO | 56.6% | 55.2% | +0.2 |
| TDPO | 54.8% | 58.0% | +0.3 |
| FPO | 50.0% | 50.0% | 0.0 |

**LLaMA-3 8b** 

| Method v.s. FPO | AlpacaEval 2 | Arena-Hard | MT-Bench |
| --- | --- | --- | --- |
| SFT | 59.0% | 58.0% | +0.3 |
| DPO | 57.2% | 56.8% | +0.5 |
| SimPO | 56.0% | 55.8% | +0.4 |
| TDPO | 56.2% | 58.0% | +0.4 |
| FPO | 50.0% | 50.0% | 0.0 |
