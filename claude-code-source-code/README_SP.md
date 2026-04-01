# Claude Code v2.1.88 — Análisis del Código Fuente

> **Aviso legal**: Todo el código fuente en este repositorio es propiedad intelectual de **Anthropic y Claude**. Este repositorio se proporciona estrictamente para investigación técnica, estudio e intercambio educativo entre entusiastas. **El uso comercial está estrictamente prohibido.** Ningún individuo, organización o entidad puede usar este contenido con fines comerciales, actividades lucrativas, actividades ilegales o cualquier otro escenario no autorizado. Si algún contenido infringe sus derechos legales, propiedad intelectual u otros intereses, comuníquese con nosotros y lo verificaremos y eliminaremos de inmediato.

> Extraído del paquete npm `@anthropic-ai/claude-code` versión **2.1.88**.
> El paquete publicado se distribuye en un único archivo `cli.js` (~12MB). El directorio `src/` en este repositorio contiene el **código fuente TypeScript sin agrupar** extraído del archivo tar de npm.

**Idioma**: [English](README.md) | [中文](README_CN.md) | [한국어](README_KR.md) | [日本語](README_JA.md) | **Español** | [हिंदी](README_HN.md) | [Português](README_PR.md)

---

## Tabla de Contenidos

- [Informes de Análisis Profundo (`docs/`)](#informes-de-análisis-profundo-docs) — Telemetría, nombres en clave, modo encubierto, control remoto, hoja de ruta
- [Aviso de Módulos Faltantes](#aviso-de-módulos-faltantes-108-módulos) — 108 módulos controlados por características no incluidos en el paquete npm
- [Descripción General de la Arquitectura](#descripción-general-de-la-arquitectura) — Entrada → Motor de Consultas → Herramientas/Servicios/Estado
- [Sistema de Herramientas y Permisos](#sistema-de-herramientas) — Más de 40 herramientas, flujo de permisos, sub-agentes
- [Notas de Compilación](#notas-de-compilación) — Por qué este código fuente no es directamente compilable

---

## Informes de Análisis Profundo (`docs/`)

Informes de análisis de código fuente derivados de la versión descompilada 2.1.88.

| # | Tema | Hallazgos Clave |
|---|-------|-------------|
| 01 | **Telemetría y Privacidad** | Dos sumideros de análisis (1P → Anthropic, Datadog). Huella digital del entorno, métricas del proceso, hash del repositorio en cada evento. **Sin opción de exclusión (opt-out) expuesta en la interfaz** para registros de terceros. |
| 02 | **Funciones Ocultas y Nombres en Clave** | Nombres en clave de animales (Capybara v8, Tengu, Fennec→Opus 4.6, **Numbat** próximo). Banderas de características que usan pares de palabras aleatorias. Los comandos ocultos incluyen `/btw`, `/stickers`. |
| 03 | **Modo Encubierto** | Los empleados de Anthropic ingresan automáticamente en modo encubierto en los repositorios públicos. El modelo instruido: *"No reveles tu identidad"* y escribe confirmaciones "como lo haría un desarrollador humano". |
| 04 | **Control Remoto** | Consulta cada hora a `/api/claude_code/settings`. Seis o más interruptores de emergencia (killswitches). |
| 05 | **Hoja de Ruta Futura** | Nombre en clave **Numbat** confirmado. Opus 4.7 / Sonnet 4.8 en desarrollo. **KAIROS** = modo de agente totalmente autónomo. Modo de voz listo pero inactivo. 17 herramientas no lanzadas encontradas. |

---

## Aviso de Módulos Faltantes (108 módulos)

> **Este código fuente está incompleto.** 108 módulos referenciados por ramas `feature()` **no están incluidos** en el paquete npm. Solo existen en el monorepositorio interno de Anthropic.

### ¿Por qué faltan?

Bun's `feature()` es un componente compilable:
- Devuelve `true` en la versión interna de Anthropic → el código se mantiene.
- Devuelve `false` en la compilación pública → ocurre una eliminación por código inútil (DCE).

---

## Estadísticas

| Ítem | Cantidad |
|------|-------|
| Archivos fuente (.ts/.tsx) | ~1,884 |
| Líneas de código | ~512,664 |
| Mayor archivo individual | `query.ts` (~785KB) |
| Herramientas integradas | ~40+ |
| Comandos de barra (slash) | ~80+ |
| Dependencias | ~192 paquetes |
| Entorno de ejecución | Bun |

---

## El Patrón de Agente

```text
                    BÚCLE PRINCIPAL
                    ===============

    Usuario --> messages[] --> API de Claude --> Respuesta
                                           |
                                 ¿stop_reason == "tool_use"?
                                /                         \
                               sí                          no
                               |                            |
                         ejecutar herramientas          devolver texto
                         añadir tool_result
                         volver a iterar ----------> messages[]
```

Ese es el ciclo mínimo de un agente. Claude Code rodea este bucle con herramientas de producción: permisos, streaming (flujo), concurrencia, compactación, sub-agentes, persistencia y MCP.

---

## Derechos de Autor y Aviso

```text
Copyright (c) Anthropic. Todos los derechos reservados.

Todo el código fuente en este repositorio es propiedad intelectual de Anthropic y Claude.
Este repositorio se proporciona exclusivamente para investigación técnica y educativa.
El uso comercial está estrictamente prohibido.
```

*(Nota: Para diagramas extendidos y esquemas en profundidad de la arquitectura y variables, consulte el README principal original en Inglés.)*
