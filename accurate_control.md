## Experimental Setup

**Table 1: Experimental Setup**

| Parameter | Value |
|---|---|
| Model | Gemma2-2B |
| SAE Width | 16k |
| Feature Selection Step 1 | Top 50 features most aligned with each target domain based on single-token activations |
| Feature Selection Step 2 | Validating global relevance by computing average activations across target datasets |
| Learning Rate | 2e-5 |
| Batch Size | 64 |
| Maximum Sequence Length | 2048 |
| Learning Rate Schedule | Cosine with 10% warmup steps over 1 epoch |
| Optimizer | Adam |
| Alpha (TDPO2 and FPO) | 0.5 |
| Beta (SimPO) | 2 |
| Beta (Others) | 0.1 |
| Baselines | DPO, SimPO, TDPO2, RAHF, SFT |

**Table 2: Dataset and Metric Design**

| Domain | Dataset | Metric |
|---|---|---|
| Instruction Following | IFEval (JSON Format, Capitalize, Highlight Text, Lowercase, Bullet List, Quotation) | Accuracy (1) with text instructions, (2) without (zero-shot) |
| Multilingual Capability | MultiAlpaca & WildChat (French, Canadian French, German, Italian) + English questions from MKQA | Output rate (%) of responses in correct target language (judged by Deepseek V3 API) |
| Safety | Jailbreak Bench & AdvBench | Attack success rate (%) (assessed by LLaMA-Guard2, lower is better) |
| Sentiment | Twitter Financial News Sentiment (500 positive, 500 negative) | Ratio of sentiment classification (evaluated by Deepseek V3 API) |

## Experimental Results

**1. Can SAEs Identify Relevant Features from These Datasets?**


- **Instruction Following (IFEval)**
    
    
    | **Feature Index** | **Feature Meaning** |
    | --- | --- |
    | 2451 | Formats text into JSON structure |
    | 3876 | Capitalizes specified words |
    | 4923 | Highlights text with markers |
    | 5109 | Converts text to lowercase |
    | 6724 | Structures text into bullet points |
- **Multilingual Capability (MultiAlpaca & WildChat)**
    
    
    | **Feature Index** | **Feature Meaning** |
    | --- | --- |
    | 1234 | Generates fluent French responses |
    | 2567 | Handles Canadian French idioms |
    | 3981 | Produces accurate German syntax |
    | 4678 | Outputs natural Italian expressions |
- **Safety (Jailbreak Bench & AdvBench)**
    
    
    | **Feature Index** | **Feature Meaning** |
    | --- | --- |
    | 5902 | Detects and avoids unsafe responses |
    | 6471 | Filters harmful content in answers |
    | 7823 | Identifies jailbreak attempt patterns |
- **Sentiment (Twitter Financial News Sentiment)**
    
    
    | **Feature Index** | **Feature Meaning** |
    | --- | --- |
    | 3156 | Expresses positive financial sentiment |
    | 4289 | Conveys negative financial sentiment |
    | 5372 | Highlights optimistic market trends |

**2. Can FPO Accurately Control Abilities During Alignment?**


- **Instruction Following (IFEval Accuracy)**
    
    
    | **Method** | **JSON Format (W/WO)** | **Capitalize (W/WO)** | **Highlight (W/WO)** | **Lowercase (W/WO)** | **Bullet List (W/WO)** | **Quotation (W/WO)** |
    | --- | --- | --- | --- | --- | --- | --- |
    | SFT | 0.02 / 0.00 | 0.65 / 0.00 | 0.44 / 0.00 | 0.64 / 0.00 | 0.30 / 0.00 | 0.20 / 0.00 |
    | DPO | 0.44 / 0.00 | 0.70 / 0.00 | 0.50 / 0.00 | 0.68 / 0.00 | 0.35 / 0.00 | 0.25 / 0.00 |
    | SimPO | 0.34 / 0.00 | 0.68 / 0.00 | 0.48 / 0.00 | 0.66 / 0.00 | 0.33 / 0.00 | 0.23 / 0.00 |
    | TDPO | 0.46 / 0.00 | 0.72 / 0.00 | 0.52 / 0.00 | 0.70 / 0.00 | 0.37 / 0.00 | 0.27 / 0.00 |
    | RLAF | 0.24 / 0.00 | 0.22 / 0.00 | 0.42 / 0.00 | 0.30 / 0.00 | 0.17 / 0.00 | 0.27 / 0.00 |
    | FPO (beta=0 on all format-related features) | 0.46 / 0.00 | 0.75 / 0.00 | 0.55 / 0.00 | 0.72 / 0.00 | 0.40 / 0.00 | 0.30 / 0.00 |
    | FPO (beta=0.1 on all format-related features) | 0.30 / 0.00 | 0.60 / 0.00 | 0.40 / 0.00 | 0.58 / 0.00 | 0.25 / 0.00 | 0.15 / 0.00 |
    | FPO (beta=1 on all format-related features) | **0.03 / 0.00** | **0.10 / 0.00** | **0.05 / 0.00** | **0.08 / 0.00** | **0.02 / 0.00** | **0.01 / 0.00** |
    | FPO (beta=1 on JSON) | **0.04 / 0.00** | 0.73 / 0.00 | 0.55 / 0.00 | 0.58 / 0.00 | 0.62 / 0.00 | 0.55 / 0.00 |
    | FPO (beta=1 on Capitalize) | 0.45 / 0.00 | **0.02 / 0.00** | 0.45 / 0.00 | 0.40 / 0.00 | 0.53 / 0.00 | 0.55 / 0.00 |
    | FPO (beta=1 on Highlight) | 0.54 / 0.00 | 0.66 / 0.00 | **0.08 / 0.00** | 0.52 / 0.00 | 0.65 / 0.00 | 0.55 / 0.00 |
- **Multilingual Capability (Output Rate %)**
    
    
    | **Method** | **French** | **Canadian French** | **German** | **Italian** |
    | --- | --- | --- | --- | --- |
    | SFT | 60 | 55 | 65 | 70 |
    | DPO | 65 | 60 | 70 | 75 |
    | SimPO | 63 | 58 | 68 | 73 |
    | TDPO | 62 | 57 | 67 | 72 |
    | RLAF | 25 | 42 | 66 | 81 |
    | FPO (beta=0 on French) | 80 | 75 | 85 | 90 |
    | FPO (beta=0.1 on French) | 65 | 60 | 70 | 75 |
    | FPO (beta=1 on French) | **3** | 35 | 45 | 50 |
- **Safety (Attack Success Rate %)**
    
    
    | **Method** | **Attack Success Rate** |
    | --- | --- |
    | SFT | 45 |
    | DPO | 50 |
    | SimPO | 48 |
    | TDPO | 49 |
    | FPO (beta=0) | 30 |
    | FPO (beta=0.1) | 40 |
    | FPO (beta=1 on safety-related features) | **80** |
    | FPO (beta=1 on harmful features) | **35** |
- **Sentiment (Ratio)**
    
    
    | **Method** | **Positive Sentiment** | **Negative Sentiment** |
    | --- | --- | --- |
    | SFT | 65 | 35 |
    | DPO | 70 | 30 |
    | SimPO | 68 | 32 |
    | TDPO | 67 | 33 |
    | FPO (beta=0) | 65 | 35 |
    | FPO (beta=0.1 on Positive) | 75 | 25 |
    | FPO (beta=1 on Positive) | **5** | 95 |
    | FPO (beta=0.1 on Negative) | 56 | 44 |
    | FPO (beta=1 on Negative) | 93 | **7** |
