---
title: "MCP is dead, Long live the CLI"
type: note
status: active
tags:
  - llm
  - mcp
  - cli
  - opinion
aliases:
  - MCP is Dead Long Live the CLI
created: 2026-03-01
updated: 2026-03-01
source: https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html
---

# MCP is dead, Long live the CLI

> [!INFO] Fuente
> Basado en: [MCP is dead. Long live the CLI](https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html) por Eric Holmes.

---

## tesis Principal

El autor argumenta que MCP (Model Context Protocol) ya está muriendo y que los CLIs son una mejor alternativa para conectar LLMs con herramientas externas.

## Argumento

### Los LLMs no necesitan un protocolo especial

Los LLMs son buenos usando herramientas de línea de comandos. Han sido entrenados con millones de man pages, Stack Overflow y repos de GitHub. No necesitan un nuevo protocolo.

### Los CLIs son para humanos también

Cuando Claude hace algo inesperado con Jira, puedes ejecutar el mismo comando `jira issue view` y ver exactamente lo que vio. Con MCP, solo existe dentro de la conversación del LLM.

### Composabilidad

Los CLIs se pueden combinar:
- Pipe a través de `jq`
- Encadenar con `grep`
- Redirigir a archivos

### Auth ya funciona

MCP es innecesariamentedogmático sobre autenticación. Los CLIs usan flujos probados:
- `aws` usa perfiles y SSO
- `gh` usa `gh auth login`
- `kubectl` usa kubeconfig

### Sin partes móviles

Los servidores MCP locales son procesos que necesitan iniciarse y mantenerse corriendo. Los CLIs son simplemente binarios en disco.

## Dolor práctico

- **Inicialización es inestable**: Servidores MCP no arrancan correctamente
- **Re-auth nunca termina**: Autenticar cada herramienta MCP
- **Permisos son todo o nada**: No se puede restringir a operaciones de solo lectura

## Cuándo tiene sentido MCP

Si una herramienta genuinamente no tiene equivalente CLI, MCP podría ser la opción correcta.

## Conclusión

Los mejores herramientas son las que funcionan para humanos y máquinas. Los CLIs tienen décadas de iteración de diseño.

> [!QUOTE] Citado
> "If you're a company investing in an MCP server but you don't have an official CLI, stop and rethink what you're doing. Ship a good API, then ship a good CLI. The agents will figure it out."

---

## Ver también

- [[]]
