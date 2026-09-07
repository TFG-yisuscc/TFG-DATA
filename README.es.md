# TFG-DATA

Repositorio de datos de los experimentos de inferencia con modelos de lenguaje cuantizados en Raspberry Pi 5.
> **IMPORTANTE** la carpeta.v1_discard contiene los datos de runs anteriores con fallos en la toma de metricas relativas al ventilador y a la cabezas kv
## Estructura general

```
TFG-DATA/
├── E0-FAN/          # Experimento 0 — línea base con ventilador activo
├── E0-NOFAN/        # Experimento 0 — sin ventilador (datos de ollama no utilizados)
├── E1/              # Experimento 1 — variación de cuantización
├── E2/              # Experimento 2 — variación de tamaño de contexto
├── E3/              # Experimento 3 — variación de batch size
├── E5/              # Experimento 5 — acelerador Hailo-10h(Ai HAt 2+)
├── Perplejidad/     # Medición de perplejidad sobre WikiText-2
└── deepseek_type2/  # Traza detallada de RAM y CPU durante inferencia con DeepSeek (TYPE_2)
```

> **Nota sobre E4:** Los datos de E4 se generaron en la misma sesión de medición que E0-FAN. Por limitaciones de almacenamiento no se han duplicado como carpeta independiente en este repositorio.

---

### Estructura interna de cada experimento (E0–E5)

Cada experimento contiene subdirectorios nombrados con el timestamp de inicio del run (nanosegundos Unix). Dentro de cada run hay tres ficheros:

```
<run_id>/
├── <metric_id>_hw_metrics_<modelo>_<test>.jsonl      # Métricas hardware (CPU, RAM, temperatura...)
├── <metric_id>_prompt_metrics_<modelo>_<test>.jsonl  # Métricas de inferencia (tokens/s, latencia...)
└── resumen.json                                       # Configuración y metadatos del run
```

### Estructura interna de Perplejidad

Cada subdirectorio corresponde a una medición individual (un modelo + cuantización), nombrado con el timestamp de inicio en nanosegundos Unix. Dentro de cada run hay dos ficheros:

```
<run_id>/
├── <run_id>_perplexity_<modelo>-<cuantización>.jsonl  # Perplejidad por chunk (token negativo log-likelihood)
└── resumen.json                                        # Configuración, metadatos y resultado final (PPL)
```

---

## Descripción de los experimentos

### E0-FAN — Línea base con ventilador activo

| Parámetro | Valor |
|---|---|
| Motor de inferencia | OLLAMA y llama.cpp |
| Modelos | Ministral-3B, DeepSeek-R1-Distill-Qwen-1.5B, Gemma-4.5B, Granite-4.0-h-micro, Llama-3.2-1B, Llama-3.2-3B |
| Cuantización | Q4_K_M |
| Contexto | 4096 tokens |
| Runs | 12 (6 modelos × 2 backends) |
| Condición térmica | Active Cooler instalado y activo |

Experimento de referencia. Los datos de E0-FAN y E4 se generaron en la misma sesión; por limitaciones de almacenamiento, E4 no se ha añadido como carpeta independiente.

---

### E0-NOFAN — Sin ventilador (Active Cooler retirado)

| Parámetro | Valor |
|---|---|
| Motor de inferencia | llama.cpp |
| Modelos | mismos que E0-FAN |
| Cuantización | Q4_K_M |
| Contexto | 4096 tokens |
| Runs | 12 (6 modelos × 2 backends) |
| Condición térmica | Active Cooler retirado físicamente del dispositivo |

Los runs con Ollama se realizaron como prueba inicial sin ventilador y **no se utilizaron** en el análisis final del proyecto. En este experimento se retiró físicamente el Active Cooler de la Raspberry Pi.

---

### E1 — Variación de cuantización

| Parámetro | Valor |
|---|---|
| Motor de inferencia | llama.cpp |
| Modelos | mismos que E0-FAN |
| Cuantizaciones | Q3_K_M, Q4_0, Q4_K_M, Q5_K_M, Q8_0 |
| Contexto | 4096 tokens (fijo) |
| Runs | 30 (6 modelos × 5 cuantizaciones) |
| Condición térmica | Active Cooler instalado y activo |

---

### E2 — Variación de tamaño de contexto

| Parámetro | Valor |
|---|---|
| Motor de inferencia | llama.cpp |
| Modelos | mismos que E0-FAN |
| Cuantización | Q4_K_M (fija) |
| Contextos | 512, 1024, 2048, 4096, 5120 tokens |
| Runs | 30 (6 modelos × 5 tamaños de contexto) |
| Condición térmica | Active Cooler instalado y activo |

---

### E3 — Variación de batch size

| Parámetro | Valor |
|---|---|
| Motor de inferencia | llama.cpp |
| Modelos | mismos que E0-FAN |
| Cuantización | Q4_K_M (fija) |
| Contexto | 4096 tokens (fijo) |
| Batch sizes | 128, 256, 512, 1024, 2048 |
| Runs | 31 (6 modelos × 5 batch sizes |
| Condición térmica | Active Cooler instalado y activo |

---

### E5 — Acelerador Hailo-10h

| Parámetro | Valor |
|---|---|
| Motores de inferencia | llama.cpp y HAILO_OLLAMA |
| Modelos | DeepSeek-R1-Distill-Qwen-1.5B, Llama-3.2-1B, Llama-3.2-3B |
| Cuantización | Q4_K_M |
| Contexto | 2048 tokens |
| Runs | 5 (3 con llama.cpp + 2 con HAILO_OLLAMA) |
| Condición térmica | Active Cooler instalado y activo |

Los runs ejecutados con llama.cpp en este experimento se realizaron con el HAT del acelerador Hailo-10h físicamente colocado en la Raspberry Pi, pero **sin hacer uso de él** (acelerador desactivado). Esto permite una comparación directa con los runs de HAILO_OLLAMA bajo las mismas condiciones físicas. En el resto de experimentos (E0-FAN, E0-NOFAN, E1, E2, E3) el HAT no estaba colocado.

---

### deepseek_type2 — Traza de RAM y CPU durante inferencia

| Parámetro | Valor |
|---|---|
| Motor de inferencia | OLLAMA |
| Modelo | DeepSeek-R1-Distill-Qwen-1.5B (Q4_K_M) |
| Cuantización | Q4_K_M |
| Contexto | 4096 tokens |
| Batch size | 512 |
| `test_type` | TYPE_2 |
| `hardware_period` | 0.25 s (muestreo cada 250 ms) |
| Condición térmica | Sin ventilador, sin acelerador |

Datos de tipo TYPE_2 orientados a obtener una traza temporal de alta frecuencia del uso de RAM y CPU durante la inferencia. El periodo de muestreo de 0.25 s permite observar la evolución de la ocupación de memoria y la carga de los núcleos a lo largo de cada prompt.
