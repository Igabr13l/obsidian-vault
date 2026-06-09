---
title: "Arquitectura Cognitiva de Agentes"
type: note
status: active
tags:
  - ia
  - agentes
  - arquitectura
  - llm
aliases:
  - Anatomía de un Agente IA
  - Sistema de Agentes
created: 2026-03-24
updated: 2026-03-24
source: Apuntes de estudio sobre ecosistemas de agentes (Antigravity)
---

# Arquitectura Cognitiva de Agentes

> [!INFO] Fuente
> Notas sobre cómo se estructuran los sistemas de Inteligencia Artificial modernos para lograr autonomía más allá del simple chat.

Un Modelo de Lenguaje (LLM) "crudo" no es un agente. Un LLM es simplemente un motor estadístico de predicción de tokens. Para que ese motor pueda realizar tareas útiles de forma autónoma (leer correos, investigar, programar, corregir errores), necesita estar envuelto en una **Arquitectura Cognitiva**.

## Componentes de un Agente

En ecosistemas avanzados como Google Antigravity, AutoGen o LangChain, un agente se compone de 5 piezas fundamentales:

1. **El Motor de Razonamiento (El LLM):** 
   Es el "cerebro" (ej. GPT-4, Claude, Gemini). Su única función es procesar texto, planificar y tomar decisiones en base al contexto que recibe.
   
2. **Memoria (Contexto):**
   Mantiene el estado de lo que está pasando. Se divide en corto plazo (el historial del chat actual) y largo plazo (bases de datos vectoriales y RAG). *Ver [[Memoria y Retrieval (Humanos e IA)]].*

3. **Tools (Herramientas / Acción):**
   Son las "manos" del agente. Funciones atómicas de código que el LLM puede invocar para interactuar con el mundo exterior. (Ej: `buscar_en_google()`, `leer_archivo()`, `ejecutar_bash()`).

4. **Skills (Habilidades / Conocimiento):**
   Es el contexto experto empaquetado. Le enseña al LLM las "reglas del juego" de un dominio específico antes de que empiece a trabajar.

5. **Workflows (Flujos de Trabajo / Orquestación):**
   Son los "rieles" lógicos. Definen el proceso paso a paso que el agente debe seguir para no perderse en tareas complejas.

> [!IMPORTANT] La clave de la Autonomía
> La autonomía real no se logra haciendo un LLM más grande, sino diseñando mejores **Skills** y **Workflows** alrededor del LLM para estructurar su forma de "pensar" y "actuar".

---

## Ver también

- [[Skills (Habilidades) en IA]]
- [[Workflows (Flujos de Trabajo) en IA]]
- [[Memoria y Retrieval (Humanos e IA)]]
