---
title: Cadena de Pensamiento (Chain of Thought)
type: note
status: active
tags:
  - ia
  - prompt-engineering
  - chain-of-thought
  - cot
  - deepseek
aliases:
  - Razonamiento Paso a Paso
  - CoT
  - Cadena de Pensamiento
created: 2024-02-15
updated: 2026-02-17
---

# Cadena de Pensamiento (Chain of Thought)

> [!INFO] Fuente
> Basado en documentación técnica de LLMs y apuntes personales.

La **Cadena de Pensamiento** (o *Chain of Thought - CoT*) se refiere a la capacidad de un modelo de lenguaje para descomponer problemas complejos en pasos intermedios de razonamiento lógico antes de dar una respuesta final.

## ¿La versión estándar tiene CoT?

> [!QUESTION] La respuesta corta: Sí y No
> Depende de si hablamos de **capacidad bruta** o de **comportamiento por defecto**.

### 1. Capacidad Inherente (Arquitectura)
Todos los [[Modelos Transformers|Transformers]] actuales (incluyendo la versión estándar) tienen la capacidad técnica de razonar internamente.
- **Potencial:** Pueden seguir pasos lógicos.
- **Activación:** Si usas [[Prompt Engineering]] explícito (ej: *"piensa paso a paso"*), el modelo estándar activará esta capacidad.

### 2. Diferencia de Comportamiento (Entrenamiento)

| Característica | Modelo Estándar (Generalista) | Modelo R1 (Reasoning / CoT) |
| :--- | :--- | :--- |
| **Enfoque** | Eficiencia y respuesta directa. | Profundidad y transparencia. |
| **Razonamiento** | Oculto (Caja negra). Tiende a saltar a la conclusión. | **Explícito**. Entrenado para "verbalizar" su pensamiento. |
| **Optimización** | Consultas generales y velocidad. | Problemas complejos de lógica, matemáticas o código. |
| **Origen** | Pre-entrenamiento estándar. | [[Fine-Tuning]] específico con ejemplos de razonamiento (RLHF). |

---

## 🧠 Flujo de Procesamiento

El modelo **R1** hace visible lo que el estándar hace "en silencio" o salta por eficiencia.

```mermaid
graph TD
    subgraph "Modelo Estándar"
    A1[Prompt] --> B1("Procesamiento Interno<br/>(Latente / Oculto)")
    B1 --> C1[Respuesta Directa]
    end

    subgraph "Modelo R1 (CoT Nativo)"
    A2[Prompt] --> B2[/"Tag: Thinking Start"/]
    B2 --> C2("Generación de Cadena de Pensamiento<br/>(Razonamiento visible paso a paso)")
    C2 --> D2[/"Tag: Thinking End"/]
    D2 --> E2[Respuesta Final Refinada]
    end
    
    style C2 fill:#ff9,stroke:#333,stroke-width:2px,color:black
```

---

## Ver también

- [[Fine-Tuning]]
- [[Glosario LLM]]
