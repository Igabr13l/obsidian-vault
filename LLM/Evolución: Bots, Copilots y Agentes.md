---
title: "Evolución: Bots, Copilots y Agentes"
type: note
status: active
tags:
  - ia
  - conceptos
  - agentes
  - bots
aliases:
  - Diferencia entre Bot y Agente
  - Qué es un Copilot
created: 2026-03-24
updated: 2026-03-24
source: Apuntes sobre la evolución de los asistentes digitales
---

# Evolución: Bots, Copilots y Agentes

> [!INFO] Fuente
> Clasificación conceptual sobre cómo han evolucionado las interfaces conversacionales y los sistemas automatizados.

En el mundo del software y la Inteligencia Artificial, los términos "Bot", "Copiloto" y "Agente" suelen usarse como sinónimos, pero técnicamente representan tres paradigmas evolutivos completamente distintos, separados por su **nivel de autonomía** y su **forma de razonar**.

## 1. Bots (Sistemas Deterministas)

Son la generación más antigua. Un Bot es un programa que ejecuta tareas repetitivas basándose en reglas estrictas creadas por un programador.

> [!DEFINITION] Bot Tradicional
> Funciona bajo la lógica de **"If-This-Then-That"** (Si pasa esto, hacé aquello). No hay "inteligencia" ni comprensión del lenguaje, solo árboles de decisión (Decision Trees).

- **Ejemplo:** El típico chatbot de atención al cliente de un banco donde tenés que apretar "Opción 1", "Opción 2". Si le escribís algo fuera del guion, responde: *"No entiendo tu consulta"*.
- **Autonomía:** Nula. Solo sigue el camino trazado.

## 2. Copilots (Asistentes Reactivos)

Con la llegada de los LLMs (como GPT o Claude), nacen los Copilotos. Entienden lenguaje natural y son increíblemente inteligentes, pero son fundamentalmente **reactivos**. 

> [!DEFINITION] Copilot
> Un asistente impulsado por IA que trabaja "hombro a hombro" con el humano. El humano es el piloto que tiene el control del volante; la IA sugiere, autocompleta o responde preguntas.

- **Ejemplo:** GitHub Copilot (sugiere código mientras escribís) o ChatGPT estándar.
- **Autonomía:** Muy baja. Un Copilot no toma la iniciativa. Espera tu *prompt*, genera una respuesta (texto, código, imagen) y se detiene hasta que le vuelvas a hablar.

## 3. Agentes Autónomos (Sistemas Proactivos)

Es la frontera actual de la IA. Un Agente es un sistema que recibe un **objetivo de alto nivel** y tiene la capacidad de planificar, usar herramientas y corregir sus propios errores hasta cumplirlo.

> [!IMPORTANT] El Salto al Agente
> En un Agente, **el humano delega el volante**. El humano dice *"Investigá a estos 3 competidores, armá un reporte en PDF y mandáselo a mi jefe"*, y el Agente desglosa la tarea en pasos, busca en la web, procesa datos, crea el archivo y lo envía.

- **Características Clave:** Tienen [[Arquitectura Cognitiva de Agentes]] (Memoria, Tools, Skills y Workflows).
- **Ejemplo:** Devin (creando software solo), Google Antigravity, AutoGPT.
- **Autonomía:** Alta. Entran en ciclos de retroalimentación (*Loops*) donde evalúan su propio progreso.

---

## Ver también

- [[Arquitectura Cognitiva de Agentes]]
- [[Sistemas Multi-Agente (MAS)]]
- [[Function Calling en LLMs]]
