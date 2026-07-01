# Triage de Leads VIP — Automatización con IA

Sistema de automatización que recibe leads comerciales, los clasifica automáticamente con inteligencia artificial (VIP / Normal / Descartar), y gestiona la aprobación humana y el contacto final para los leads de alto valor.

## Stack tecnológico

| Componente | Herramienta |
|---|---|
| Orquestación | [n8n](https://n8n.io) |
| Base de datos | Airtable |
| Motor de IA | Google Gemini (`gemini-2.5-flash`) |
| Aprobación humana (HITL) | Gmail — Send and Wait for Response |
| Notificación final | Gmail |

> **Nota sobre el motor de IA:** la consigna original solicitaba OpenAI o Anthropic. Se optó por Google Gemini por su tier gratuito, decisión **consultada y aprobada por el profesor** antes de la entrega final.

## Arquitectura

```
Webhook (nuevo lead)
   │
   ▼
IF ¿Campos completos?
   │ NO → Airtable: Logs de Error → FIN
   │ SI
   ▼
Airtable: crear Lead (Estado = Pendiente)
   │
   ▼
Gemini: clasificar lead (VIP / Normal / Descartar + justificación)
   │ ERROR → Airtable: Logs de Error + Estado = Error → FIN
   │ OK
   ▼
Airtable: actualizar Clasificación + Justificación (Estado = Procesado por IA)
   │
   ▼
IF ¿Es VIP?
   │ NO → Airtable: Estado = Descartado → FIN
   │ SI
   ▼
Airtable: Estado = Esperando Aprobación
   │
   ▼
Gmail: Send and Wait for Response (aprobación humana)
   │
   ▼
IF ¿Aprobado?
   │ NO → Airtable: Estado = Rechazado → FIN
   │ SI
   ▼
Gmail: enviar email de contacto al lead
   │
   ▼
Airtable: Estado = Contactado → FIN
```

## Estructura del repositorio

```
├── README.md
├── docs/
│   └── Documentacion_Triage_Leads_VIP.pdf   # Documentación técnica completa
├── workflow/
│   └── Triage_de_Leads_VIP.json             # Workflow exportado de n8n
└── capturas/
    ├── 01_ejecucion_vip_aprobado.png
    ├── 02_airtable_leads.png
    ├── 03_email_aprobacion.png
    ├── 04_bounce_email.png
    ├── 05_ejecucion_descartado.png
    ├── 06_airtable_logs_error.png
    ├── 07_ejecucion_rechazado.png
    └── 08_airtable_leads_justificacion.png
```

## Modelo de datos (Airtable)

Base: **CRM Leads IA**

**Tabla `Leads`**: Nombre, Email, Empresa, Mensaje, Estado, Clasificación IA, Justificación IA, Fecha creación, Fecha aprobación.

**Tabla `Logs de Error`**: Lead (link a Leads), Nodo que falló, Mensaje de error, Fecha.

## Casos de prueba

| # | Caso | Resultado |
|---|---|---|
| 1 | Lead VIP aprobado | Estado final: Contactado |
| 2 | Campos incompletos | Registrado en Logs de Error |
| 3 | Lead no relevante (Normal/Descartar) | Estado final: Descartado |
| 4 | Lead VIP rechazado manualmente | Estado final: Rechazado |

Ver detalle completo, capturas y justificación de cada decisión técnica en [`docs/Documentacion_Triage_Leads_VIP.pdf`](docs/Documentacion_Triage_Leads_VIP.pdf).

## Enlaces

- **Repositorio:** https://github.com/thiagosilva9877/triage-leads-vip
- **Base de datos (solo lectura):** https://airtable.com/appByuC301aOkfq9u/shroYMETdT9SLcixW
- **Documentación completa:** `docs/Documentacion_Triage_Leads_VIP.pdf`
