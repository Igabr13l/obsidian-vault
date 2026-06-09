---
title: "Workflows y Skills en Agentes IA"
type: note
status: active
tags:
  - ia
  - agentes
  - llm
  - automatizacion
  - antigravity
aliases:
  - Workflows en IA
  - Diferencia entre Tools, Skills y Workflows
created: 2026-03-24
updated: 2026-03-24
source: Apuntes personales sobre ecosistemas de agentes (Antigravity)
---

# Workflows y Skills en Agentes IA

> [!INFO] Fuente
> Notas sobre la arquitectura de agentes autónomos, frameworks como Google Antigravity y la orquestación de LLMs.

A medida que los Modelos de Lenguaje (LLMs) evolucionan de ser simples "chatbots" a convertirse en **Agentes Autónomos** capaces de ejecutar tareas complejas, surgen tres conceptos que trabajan en conjunto pero hacen cosas distintas: **Tools (Herramientas), Skills (Habilidades) y Workflows (Flujos de trabajo).**

## ¿Qué es un Workflow?

Un **Workflow** (Flujo de trabajo) es la **receta, el proceso paso a paso o el Procedimiento Operativo Estándar (SOP)** que un agente de IA debe seguir para completar una tarea compleja sin perderse ni alucinar.

> [!DEFINITION] Workflow en IA
> Es una secuencia estructurada de pasos, decisiones condicionales (If/Else) y llamadas a herramientas que orquestan el comportamiento del LLM para garantizar un resultado predecible y repetible.

Mientras que un LLM suelto intenta adivinar qué hacer en cada interacción (lo que a menudo lleva a errores en tareas largas), un workflow le pone "rieles" o "barreras de contención" (guardrails) a su razonamiento.

---

## La Analogía del Chef

Para entender cómo se relacionan en sistemas modernos como Antigravity:

| Concepto | En la Cocina (Metáfora) | En IA / Agentes |
|----------|-------------------------|-----------------|
| **LLM** | El Chef | El motor de razonamiento básico (ej. GPT-4, Gemini). |
| **Tools** | Cuchillos, Sartenes, Horno | Capacidades atómicas: Buscar en Google, Leer archivo, Ejecutar código Python. |
| **Skills** | Saber hacer comida italiana | Conocimiento de dominio especializado (ej. "Skill de React/Apollo", "Skill de Data Analysis"). |
| **Workflows** | La receta exacta de la Lasaña | "Paso 1: Leé la base de datos. Paso 2: Si hay error, avisá al usuario. Paso 3: Transformá a JSON..." |

---

## ¿Por qué son vitales en sistemas como Antigravity?

Frameworks avanzados o modelos de nueva generación (como los asociados a proyectos tipo *Antigravity*) se centran en la **autonomía a largo plazo**. Para que un agente trabaje solo durante horas sin que el usuario intervenga, necesita Workflows por tres razones:

1. **Reducción de Alucinaciones en Tareas Largas:** Si le pedís a un LLM que "cree una app entera", en el paso 5 se olvida de lo que hizo en el paso 1. Un workflow divide el trabajo en nodos discretos (Planificar -> Escribir Backend -> Testear -> Escribir Frontend).
2. **Human-in-the-loop (HITL):** Los workflows permiten establecer "puntos de control". Por ejemplo: el agente investiga, arma un plan, **se detiene y pide aprobación al usuario**, y solo continúa cuando el humano le da el OK.
3. **Manejo de Errores Predictivo:** Un buen workflow tiene ramas condicionales. "Intentá compilar el código. ¿Falló? Pasale el error a la Skill de Debugging. ¿Funcionó? Pasá al paso de despliegue."

### Las Skills contienen Workflows

En la arquitectura moderna de agentes, cuando vos "instalás" o "cargás" una **Skill** (por ejemplo, una Skill para manejar tu Obsidian), no solo le estás dando al LLM contexto o instrucciones sueltas. Le estás inyectando **Workflows predefinidos** (ej: "Cuando el usuario pida crear una nota, primero verificá si existe el archivo, luego aplicá esta plantilla YAML, luego escribí el contenido").

---

## Ver también

- [[Agentes de IA]]
- [[Memoria y Retrieval (Humanos e IA)]]
