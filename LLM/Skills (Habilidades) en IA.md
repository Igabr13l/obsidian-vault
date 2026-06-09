---
title: "Skills (Habilidades) en IA"
type: note
status: active
tags:
  - ia
  - llm
  - agentes
  - desarrollo
aliases:
  - Skill en Modelos de Lenguaje
  - Empaquetado de contexto
created: 2026-03-24
updated: 2026-03-24
source: Apuntes de estudio sobre ecosistemas de agentes
---

# Skills (Habilidades) en IA

> [!INFO] Concepto
> Una **Skill** (Habilidad) es un "paquete de conocimiento de dominio" que se inyecta temporalmente en la mente del LLM para convertirlo en un experto en un tema específico.

Si el LLM es un actor de teatro generalista, cargarle una **Skill** es darle el guion, el vestuario y las motivaciones de un personaje específico antes de que se abra el telón.

## Anatomía de una Skill

En arquitecturas de agentes, una Skill no es magia negra ni fine-tuning (re-entrenar la red neuronal). Es una inyección inteligente de contexto (*In-context learning*). Una buena Skill suele contener:

1. **System Prompt Especializado:** Instrucciones estrictas sobre cómo comportarse ("Eres un experto en React, tu código debe usar Hooks y seguir este estilo exacto").
2. **Ejemplos (Few-Shot Prompting):** Casos de "así se hace bien" vs "así se hace mal" para que el modelo imite el patrón.
3. **Acceso a Tools Específicas:** Una Skill de "Análisis de Datos" le dará permiso al LLM para usar una herramienta de Python (Pandas), mientras que una Skill de "Redacción SEO" no necesitará esa herramienta.
4. **Reglas de Negocio:** Limitaciones sobre qué NO debe hacer.

### Diferencia entre Skill y Tool

> [!TIP] Regla de Oro
> - La **Tool** (Herramienta) es una *acción* (ej. `get_weather(city)`).
> - La **Skill** (Habilidad) es el *conocimiento* sobre *cuándo y por qué* usar esa acción, y cómo interpretar el resultado.

## ¿Por qué usamos Skills? (El Problema del Context Window)

¿Por qué no darle al LLM absolutamente todas las instrucciones y conocimientos del mundo en un solo megapront desde el principio?

Porque los modelos sufren de **Degradación de Atención**. Si le pasás un *prompt* de 100 páginas que explica cómo hacer 50 cosas distintas, el modelo se "marea", mezcla conceptos (alucina) y rinde peor.

Las Skills resuelven esto cargándose de forma **modular y bajo demanda**. Si el usuario pide analizar un Excel, el sistema carga la *Data Analysis Skill*. Si luego pide dibujar un gráfico, descarga la anterior y carga la *Mermaid Visualizer Skill*.

---

## Ver también

- [[Arquitectura Cognitiva de Agentes]]
- [[Workflows (Flujos de Trabajo) en IA]]
