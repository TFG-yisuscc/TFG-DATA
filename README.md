# TFG-DATA

Data repository for inference experiments with quantized language models on Raspberry Pi 5.

🇪🇸 Spanish version: [README.es.md](README.es.md)

> **IMPORTANT:** the `.V1_discard` folder contains data from previous runs with issues in metric collection related to the fan and KV heads.

## General structure

```
TFG-DATA/
├── E0-FAN/          # Experiment 0 — baseline with active fan
├── E0-NOFAN/        # Experiment 0 — without fan (Ollama data not used)
├── E1/              # Experiment 1 — quantization variation
├── E2/              # Experiment 2 — context size variation
├── E3/              # Experiment 3 — batch size variation
├── E5/              # Experiment 5 — Hailo-10h accelerator (AI HAT 2+)
├── Perplejidad/     # Perplexity measurement on WikiText-2
└── deepseek_type2/  # Detailed RAM and CPU trace during inference with DeepSeek (TYPE_2)
```

> **Note about E4:** E4 data was generated in the same measurement session as E0-FAN. Due to storage limitations, it has not been duplicated as an independent folder in this repository.

---

### Internal structure of each experiment (E0–E5)

Each experiment contains subdirectories named with the run start timestamp (Unix nanoseconds). Inside each run there are three files:

```
<run_id>/
├── <metric_id>_hw_metrics_<model>_<test>.jsonl      # Hardware metrics (CPU, RAM, temperature...)
├── <metric_id>_prompt_metrics_<model>_<test>.jsonl  # Inference metrics (tokens/s, latency...)
└── resumen.json                                      # Run configuration and metadata
```

### Internal structure of Perplejidad

Each subdirectory corresponds to an individual measurement (one model + quantization), named with the run start timestamp in Unix nanoseconds. Inside each run there are two files:

```
<run_id>/
├── <run_id>_perplexity_<model>-<quantization>.jsonl  # Perplexity by chunk (negative token log-likelihood)
└── resumen.json                                       # Configuration, metadata, and final result (PPL)
```

---

## Experiment descriptions

### E0-FAN — Baseline with active fan

| Parameter | Value |
|---|---|
| Inference engine | OLLAMA and llama.cpp |
| Models | Ministral-3B, DeepSeek-R1-Distill-Qwen-1.5B, Gemma-4.5B, Granite-4.0-h-micro, Llama-3.2-1B, Llama-3.2-3B |
| Quantization | Q4_K_M |
| Context | 4096 tokens |
| Runs | 12 (6 models × 2 backends) |
| Thermal condition | Active Cooler installed and active |

Reference experiment. E0-FAN and E4 data were generated in the same session; due to storage limitations, E4 was not added as an independent folder.

---

### E0-NOFAN — Without fan (Active Cooler removed)

| Parameter | Value |
|---|---|
| Inference engine | llama.cpp |
| Models | same as E0-FAN |
| Quantization | Q4_K_M |
| Context | 4096 tokens |
| Runs | 12 (6 models × 2 backends) |
| Thermal condition | Active Cooler physically removed from the device |

Runs with Ollama were performed as an initial no-fan test and were **not used** in the final project analysis. In this experiment, the Active Cooler was physically removed from the Raspberry Pi.

---

### E1 — Quantization variation

| Parameter | Value |
|---|---|
| Inference engine | llama.cpp |
| Models | same as E0-FAN |
| Quantizations | Q3_K_M, Q4_0, Q4_K_M, Q5_K_M, Q8_0 |
| Context | 4096 tokens (fixed) |
| Runs | 30 (6 models × 5 quantizations) |
| Thermal condition | Active Cooler installed and active |

---

### E2 — Context size variation

| Parameter | Value |
|---|---|
| Inference engine | llama.cpp |
| Models | same as E0-FAN |
| Quantization | Q4_K_M (fixed) |
| Context sizes | 512, 1024, 2048, 4096, 5120 tokens |
| Runs | 30 (6 models × 5 context sizes) |
| Thermal condition | Active Cooler installed and active |

---

### E3 — Batch size variation

| Parameter | Value |
|---|---|
| Inference engine | llama.cpp |
| Models | same as E0-FAN |
| Quantization | Q4_K_M (fixed) |
| Context | 4096 tokens (fixed) |
| Batch sizes | 128, 256, 512, 1024, 2048 |
| Runs | 31 (6 models × 5 batch sizes) |
| Thermal condition | Active Cooler installed and active |

---

### E5 — Hailo-10h accelerator

| Parameter | Value |
|---|---|
| Inference engines | llama.cpp and HAILO_OLLAMA |
| Models | DeepSeek-R1-Distill-Qwen-1.5B, Llama-3.2-1B, Llama-3.2-3B |
| Quantization | Q4_K_M |
| Context | 2048 tokens |
| Runs | 5 (3 with llama.cpp + 2 with HAILO_OLLAMA) |
| Thermal condition | Active Cooler installed and active |

Runs executed with llama.cpp in this experiment were performed with the Hailo-10h accelerator HAT physically attached to the Raspberry Pi, but **without using it** (accelerator disabled). This enables a direct comparison with HAILO_OLLAMA runs under the same physical conditions. In the other experiments (E0-FAN, E0-NOFAN, E1, E2, E3), the HAT was not attached.

---

### deepseek_type2 — RAM and CPU trace during inference

| Parameter | Value |
|---|---|
| Inference engine | OLLAMA |
| Model | DeepSeek-R1-Distill-Qwen-1.5B (Q4_K_M) |
| Quantization | Q4_K_M |
| Context | 4096 tokens |
| Batch size | 512 |
| `test_type` | TYPE_2 |
| `hardware_period` | 0.25 s (sampling every 250 ms) |
| Thermal condition | Without fan, without accelerator |

TYPE_2 data is intended to provide a high-frequency temporal trace of RAM and CPU usage during inference. The 0.25 s sampling period allows observing memory occupancy and core load evolution across each prompt.
