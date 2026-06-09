---
title: "Como escribir prompts para shippear 10x mas rapido"
type: note
status: active
tags:
  - llm
  - prompt-engineering
  - prompts
  - productividad
aliases:
  - Como escribir prompts para enviar mas rapido
  - How to write prompts to ship 100x faster
created: 2026-03-08
updated: 2026-03-08
source: https://x.com/nothiingf4/status/2030682331670056964
---

# Como escribir prompts para shippear 10x mas rapido

> [!INFO] Fuente
> Basado en: [How to write Prompts to ship 100x faster](https://x.com/nothiingf4/status/2030682331670056964) de `@nothiingf4`.

---

## Idea principal

El post sostiene que la calidad del resultado de un LLM depende menos de "usar magia" y mas de escribir instrucciones con **claridad, estructura e intencion**.

> [!IMPORTANT] Punto clave
> Un mal prompt no solo produce una mala respuesta: tambien puede producir mal codigo, malas decisiones y horas perdidas de depuracion.

---

## Errores frecuentes al prompting

### Pedir demasiadas cosas a la vez

Si mezclas tareas no relacionadas en un mismo prompt, el modelo reparte su atencion y baja la precision.

### No definir el espacio negativo

No alcanza con decir que queres. Tambien conviene decir que queres evitar.

### Hacer un solo intento

El primer output conviene tratarlo como un borrador. La mejora suele aparecer en la iteracion.

### No permitir incertidumbre

Agregar reglas como `si no estas seguro, decilo en vez de adivinar` reduce alucinaciones innecesarias.

### Creer que mas texto siempre ayuda

Los prompts largos no son automaticamente mejores. El exceso de contexto irrelevante agrega ruido.

---

## Que es prompt engineering

> [!DEFINITION] Definicion
> [[Prompt Engineering]] = el arte de comunicarte con un LLM de forma lo bastante clara y estrategica como para obtener una salida util, controlable y accionable.

El modelo mental del post es simple: un LLM se parece mas a un contratista muy capaz que a un lector de mentes. Si no recibe contexto suficiente, completa los huecos por probabilidad.

---

## Tacticas que mas impacto tienen

### 1. Dar contexto claro

El autor propone pensar en tres preguntas:

- que tarea hay que hacer
- quien es la audiencia
- que formato o resultado se espera

Cuanto menos ambiguedad tenga el pedido, mejor se acota el espacio de respuesta.

### 2. Ser especifico

Pedir `escribe algo sobre Python` abre demasiadas posibilidades. En cambio, pedir una funcion concreta, con filtros, orden, type hints y docstring reduce la entropia y mejora mucho la salida.

### 3. Descomponer en pasos

Separar la tarea en pasos numerados obliga al modelo a procesar secuencialmente y reduce atajos incorrectos.

Esto es especialmente util para explicaciones, debugging y generacion de codigo.

### 4. Pedir formato de salida

Si necesitas JSON, bullets, Markdown o una plantilla fija, decilo explicitamente. Eso vuelve la salida mas reutilizable y parseable.

### 5. Usar [[Chain of Thought]] cuando haga falta

Pedir que piense paso a paso antes de responder ayuda en decisiones, comparaciones y diagnosticos tecnicos.

### 6. Mostrar ejemplos

El few-shot prompting le enseña al modelo el patron exacto que debe imitar en tono, estructura y nivel de detalle.

### 7. Agregar restricciones utiles

Limites como `maximo 100 palabras`, `sin jerga tecnica` o `prioriza seguridad antes que estilo` funcionan como guardrails.

---

## Framework operativo

El post sugiere combinar varias capas dentro de un mismo prompt:

1. contexto del rol o dominio
2. instrucciones paso a paso
3. formato de salida
4. ejemplo de referencia
5. input especifico

> [!TIP] Regla practica
> Los mejores prompts no eligen una sola tecnica. Apilan varias capas compatibles.

---

## Plantillas reutilizables

### Code review

Sirve para revisar codigo con prioridades claras: seguridad, performance y legibilidad.

### Escritura o contenido

Sirve para pedir tono, audiencia, longitud y estructura sin caer en intros genericas.

### Toma de decisiones

Sirve para comparar opciones, explicitar criterios, riesgos y recomendacion final.

### Debugging

Sirve para pedir causa mas probable, causas alternativas, solucion y snippet corregido.

---

## Implicancias para agentes de codigo

Con herramientas como Claude Code, Cursor o Codex, el prompt deja de ser solo una consulta: pasa a ser una especificacion de trabajo.

Por eso, cuanto mejor escribis:

- menos ambiguedad operativa hay
- menos cambios innecesarios aparecen
- mejor se alinea el resultado con tu objetivo
- menos tiempo perdes corrigiendo despues

---

## Conclusión

La tesis de fondo es que prompt engineering no trata de palabras magicas sino de reducir ambiguedad y guiar el espacio de decisiones del modelo.

Una buena forma de aplicarlo es empezar por una sola mejora - por ejemplo, agregar contexto o pedir formato - y luego sumar capas.

---

## Ver tambien

- [[Prompt Engineering]]
- [[Chain of Thought]]
- [[AI Second Brain con Obsidian + Claude Code]]
