# MCP Inspector SDK — Integración Scriptorium

> **Submódulo #20**: escrivivir-co/mcp-inspector-sdk  
> **Fork de**: modelcontextprotocol/inspector  
> **Rama**: `integration/scriptorium/beta`  
> **Puerto UI**: 6274  
> **Puerto Proxy**: 6277

---

## Resumen

Interfaz visual para inspeccionar, depurar y probar servidores MCP. Permite conectarse a cualquier servidor MCP vía:

- **stdio** (spawn proceso local)
- **SSE** (Server-Sent Events)
- **Streamable HTTP** (HTTP streaming)

## Integración con Scriptorium

### Servidores Pre-Configurados

El archivo `scriptorium-config.json` contiene los servidores MCP del ecosistema:

| Servidor                    | Puerto | Transporte      | Descripción                    |
| --------------------------- | ------ | --------------- | ------------------------------ |
| **launcher-server**         | 3050   | Streamable HTTP | Orquestador de servidores MCP  |
| **prolog-mcp-server**       | 3006   | Streamable HTTP | Queries Prolog + KB management |
| **typed-prompt-mcp-server** | 3020   | Streamable HTTP | Validación de ontologías       |
| **copilot-logs-mcp-server** | 3100   | Streamable HTTP | Snapshots y métricas Copilot   |
| **devops-mcp-server**       | 3003   | Streamable HTTP | Automatización DevOps          |
| **AlephAlpha**              | 3066   | Streamable HTTP | Novelist MCP Server            |
| **wiki-browser-server**     | 3002   | Streamable HTTP | Wikipedia browsing             |
| **state-machine-server**    | 3004   | Streamable HTTP | X+1 state machine              |

### Uso Rápido

```bash
# Arrancar Inspector (genera URL con token de auth)
cd MCPGallery/mcp-inspector-sdk
npm start

# Output ejemplo:
# 🚀 MCP Inspector is up and running at:
#    http://localhost:6274/?MCP_PROXY_AUTH_TOKEN=<token>
#
# ⚠️ El token es requerido para autenticación del proxy

# O con VS Code Task
# → "INS: Start [Inspector]"
```

### Auth Token

El Inspector genera un **session token** al arrancar. Este token es requerido para que el proxy pueda comunicarse con los servidores MCP.

**Opciones:**

1. Usar la URL completa que genera `npm start` (incluye el token)
2. Deshabilitar auth (solo desarrollo): `DANGEROUSLY_OMIT_AUTH=true npm start`

### Conectar a Servidor Específico

```bash
# Via CLI con config
npx @modelcontextprotocol/inspector --config scriptorium-config.json --server launcher-server

# Via Query Params (en browser después de arrancar)
http://localhost:6274?transport=sse&serverUrl=http://localhost:3006/sse
```

### URLs Directas por Servidor

| Servidor    | URL Inspector                                                                          |
| ----------- | -------------------------------------------------------------------------------------- |
| Launcher    | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3050/http` |
| Prolog      | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3006/http` |
| TypedPrompt | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3020/http` |
| Novelist    | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3066/http` |
| DevOps      | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3003/http` |
| CopilotLogs | `http://localhost:6274?transport=streamable-http&serverUrl=http://localhost:3100/http` |

---

## Arquitectura

```
┌──────────────────────────────────────────────────────────────┐
│                    MCP Inspector                              │
│                     (puerto 6274)                             │
├──────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────────────────────┐  │
│  │   Inspector UI  │◄──►│      MCP Inspector Proxy        │  │
│  │   (Vite/React)  │    │        (puerto 6277)            │  │
│  └─────────────────┘    └──────────────┬──────────────────┘  │
│                                        │                      │
│                                        │ SSE/HTTP             │
│                                        ▼                      │
│            ┌───────────────────────────────────────┐         │
│            │        Scriptorium MCP Servers         │         │
│            ├───────────────────────────────────────┤         │
│            │  launcher-server (3050)               │         │
│            │  prolog-mcp-server (3006)             │         │
│            │  typed-prompt-mcp-server (3020)       │         │
│            │  AlephAlpha (3066)                    │         │
│            │  copilot-logs-mcp-server (3100)       │         │
│            │  devops-mcp-server (3003)             │         │
│            └───────────────────────────────────────┘         │
└──────────────────────────────────────────────────────────────┘
```

---

## Features del Inspector

### 1. **Tools Tab**

Inspecciona todas las herramientas expuestas por el servidor MCP:

- Lista de tools con schemas de input/output
- Ejecución interactiva con formularios auto-generados
- Visualización de respuestas JSON

### 2. **Resources Tab**

Explora recursos expuestos:

- URIs de recursos disponibles
- Lectura de contenido
- Suscripciones a cambios

### 3. **Prompts Tab**

Prueba prompts del servidor:

- Lista de prompts disponibles
- Ejecución con argumentos
- Vista de mensajes generados

### 4. **Console Tab**

Log de comunicación MCP:

- Requests y responses
- Notificaciones
- Errores y warnings

### 5. **Auth Debugger**

Para servidores con OAuth:

- Flujo de autorización
- Token inspection
- Refresh handling

---

## Tasks de VS Code

| Task                       | Acción                                    |
| -------------------------- | ----------------------------------------- |
| `INS: Start [Inspector]`   | Arranca UI en 6274 + Proxy en 6277        |
| `INS: Open Browser`        | Abre `http://localhost:6274`              |
| `INS: Connect to Launcher` | Abre Inspector pre-conectado al Launcher  |
| `INS: Connect to Prolog`   | Abre Inspector pre-conectado a Prolog MCP |

---

## Desarrollo

```bash
# Dev mode (hot reload)
npm run dev

# Build
npm run build

# Test
npm test
```

---

## Troubleshooting

### CORS Issues

El Inspector usa un proxy (puerto 6277) para evitar problemas de CORS. Si conectas directo, el servidor MCP debe tener CORS habilitado.

### Servidor No Responde

1. Verificar que el servidor esté corriendo (`curl http://localhost:PORT/sse`)
2. El servidor debe implementar SSE endpoint en `/sse` o `/sse/message`
3. Ver logs en Console Tab del Inspector

### Transport Type

- **SSE**: Para servidores HTTP que exponen `/sse`
- **Streamable HTTP**: Para servidores con streaming bidireccional
- **stdio**: Para procesos locales (spawn)

---

## Changelog Scriptorium

| Fecha      | Cambio                                                                 |
| ---------- | ---------------------------------------------------------------------- |
| 2026-01-06 | ✅ Integración inicial como submódulo #20                              |
| 2026-01-06 | ✅ Configuración de servidores Scriptorium (`scriptorium-config.json`) |
| 2026-01-06 | ✅ Tasks de VS Code para arranque y navegación                         |
| 2026-01-06 | ✅ Integración en Demo Gallery                                         |

---

## Links

- [MCP Inspector Upstream](https://github.com/modelcontextprotocol/inspector)
- [MCP Protocol](https://modelcontextprotocol.io)
- [Scriptorium MCP Gallery](../README-SCRIPTORIUM.md)
