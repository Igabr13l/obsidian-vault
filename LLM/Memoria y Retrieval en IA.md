---
title: "Memoria y Retrieval (Humanos e IA)"
type: note
status: active
tags:
  - ia
  - rag
  - memoria
  - agentes
  - psicologia
  - aprendizaje
aliases:
  - Cuello de botella en Retrieval
  - Fases de la memoria en IA
  - Fases de la memoria humana
created: 2026-03-24
updated: 2026-03-24
source: Apuntes personales
---

# Memoria y Retrieval (Humanos e IA)

> [!INFO] Fuente
> Basado en apuntes sobre el modelo de memoria humana y su aplicación directa a sistemas de Inteligencia Artificial (especialmente arquitecturas RAG).

La memoria no falla tanto por guardar, sino por recuperar bien. Este concepto **aplica por igual a la mente humana y a los sistemas informáticos**, dividiendo el proceso de la memoria en 3 etapas fundamentales.

## 1. Encoding (Codificación)

Es el momento en que entra la información nueva y hay que transformarla en una forma que se pueda guardar. 

> [!DEFINITION] Codificación
> **En humanos:** Es cuando prestás atención, entendés algo y lo asociás con algo previo para formar una huella mental. Si no hubo atención o comprensión, la codificación fue mala.
> **En sistemas / IA:** Es cuando se recibe un dato, se procesa y se convierte en texto, metadatos, índices o embeddings para persistir.

**Punto clave:** No todo lo que entra se codifica igual de bien. Si la codificación es pobre (ej. guardar una nota en la cabeza o en un disco como "cosa importante" en lugar de "cliente pidió credenciales"), la recuperación posterior será muy pobre.

## 2. Storage (Almacenamiento)

Es la parte de mantener lo ya codificado para que no se pierda en el tiempo y quede disponible.

> [!NOTE] Persistencia
> **En humanos:** Sería la consolidación biológica y neurológica de los recuerdos.
> **En software:** Puede ser un disco, base de datos, vector store o caché persistente.

Muchos creen que "tener memoria" es solo guardar cosas. Pero podés tener una retentiva enorme, o millones de documentos guardados en una base impecable, y aun así tener una mala memoria si no encontrás lo correcto cuando lo necesitás.

## 3. Retrieval (Recuperación)

Este es el verdadero cuello de botella. Una memoria útil no es solo una que tiene información, sino una que puede traer la información correcta, en el momento correcto y con la forma correcta.

> [!WARNING] El Cuello de Botella de la Recuperación
> Recuperar no es solo buscar. Exige encontrar lo relevante, descartar lo irrelevante, resolver ambigüedad, traer el nivel correcto de detalle y hacerlo con buen contexto.

### Ejemplo cotidiano humano:
Imaginá dos personas:
- **Persona A:** Guarda todo (apuntes, links, capturas, PDFs) pero nunca encuentra nada cuando lo necesita.
- **Persona B:** Guarda menos, pero clasifica mejor y encuentra rápido lo importante.
¿Quién tiene mejor memoria en la práctica? La Persona B. Con los sistemas pasa exactamente igual.

### Por qué Retrieval es difícil (Sistemas y Mente):
1. **La información crece más rápido que la capacidad de usarla:** Guardar es barato y fácil, buscar bien en algo enorme es difícil.
2. **Problemas de relevancia:** El dato existe pero no aparece. En IA es por mala indexación o embeddings; en humanos es por falta de pistas de recuperación o asociaciones débiles.
3. **Exige interpretar intención:** Las consultas humanas ("dónde quedó lo de las credenciales") no son matches literales, requieren semántica, desambiguación y contexto.
4. **Demasiado recall también rompe:** Poco recall no encuentra nada útil, pero mucho recall sin precisión te ahoga en ruido (mezcla temas, trae recuerdos o documentos viejos).

---

## Aplicación en IA, RAG y Agentes

En sistemas RAG o copilots, aunque el LLM sea excelente, si la memoria operativa (el *retrieval*) trae mal contexto, el modelo responde mal, inventa cosas (alucinaciones) o usa datos obsoletos. 

> [!TIP] Buenas Prácticas para Mejorar el Retrieval en Sistemas
> 1. **Buena representación:** Guardar con estructura (título, fecha, fuente, tags, entidades relacionadas).
> 2. **Buen chunking:** El tamaño del fragmento importa (ni muy chico que pierda contexto, ni muy grande que meta ruido).
> 3. **Buenos índices:** Usar enfoques híbridos (Keyword + Full text + Embeddings + BM25).
> 4. **Re-ranking:** Búsqueda inicial amplia seguida de un ordenamiento y selección (*top-k*) más fino.
> 5. **Filtros temporales:** Considerar la recencia y la autoridad de la fuente (evitar traer guías viejas si hay nuevas oficiales).

**Conclusión:** Un sistema (o una persona) con almacenamiento enorme pero mala recuperación es, funcionalmente, un sistema con mala memoria.

---

## Ver también

- [[Taxonomía de Bloom]]
- [[RAG]]
- [[Agentes de IA]]
