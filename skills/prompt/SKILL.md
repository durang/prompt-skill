---
name: prompt
description: >
  Convierte una petición en bruto (español informal, typos, ideas sueltas) en UN prompt
  profesional y blindado listo para ejecutar: contexto real del repo, objetivo medible,
  fases, dependencias, verificación en vivo, reglas duras y entregable. NO ejecuta la
  tarea — solo devuelve el prompt para que Sergio decida. Invócalo con "/prompt <idea>".
---

# /prompt — el afilador de prompts

## Qué hace
Recibe la idea en bruto de Sergio y devuelve **UN solo prompt pulido** (en español,
profesional) que él revisará antes de ejecutar. **NUNCA ejecutes la tarea al recibir
/prompt** — tu único output es el prompt afinado + máximo 3 preguntas si algo es ambiguo.

## Proceso (interno, rápido)
1. **Entiende la intención real**: qué quiere lograr el NEGOCIO, no solo qué pidió.
   Interpreta typos/español informal con generosidad; si hay 2 lecturas posibles, elige
   la más valiosa y márcala como "Asunción" dentro del prompt.
2. **Aterriza en la REALIDAD del repo** (2-5 minutos máx de investigación): greps para
   confirmar rutas/archivos/sistemas que el prompt va a citar (file:line reales, nunca
   inventados). Detecta sistemas canónicos existentes que el prompt DEBE usar en vez de
   duplicar (en Vendau: lib/capabilities.ts, lib/setup-score.ts, lib/lead-labels.ts,
   brand.*, templates, cap()/resolveCap, patrón de actions con merge + fail()).
3. **Detecta dependencias y riesgos**: qué toca prod, qué necesita go-word (PUBLICAR /
   PUBLICAR-BD), qué es irreversible, qué puede romper el otro vertical/tenant.
4. **Escribe el prompt** con la plantilla de abajo.

## Plantilla del prompt entregado
- **Título** (una línea con emoji)
- **Contexto** — estado real relevante, con rutas/sistemas confirmados por grep
- **Objetivo** — medible, en una frase ("certificar que…", "dejar en vivo…")
- **Fases numeradas** — cada una con su alcance exacto; si aplica multi-agente, decir
  cuántos agentes/lentes y qué hace cada uno; separar SIEMPRE: investigar → cambiar →
  **probar EN VIVO con evidencia** (screenshot/curl, nunca "deploy succeeded")
- **Pruebas de conexión** — cómo se comprueba que quedó conectado de punta a punta
  (dato → BD → sitio/panel → verificación externa), incluyendo el caso multi-tenant
  (¿funciona igual para un cliente nuevo de otro giro/idioma?)
- **Reglas duras** — las del proyecto (aislamiento autos↔casas, canónico/verdad única,
  nada de PII/secretos, BD prod solo-lectura salvo prueba reversible, go-words) + las
  específicas de esta tarea
- **Entregable** — formato exacto del reporte final (tabla, veredictos, evidencia)
- **Asunciones y preguntas** — lo que asumiste + lo que Sergio debería confirmar
- Cierre: "¿Lo ejecuto tal cual o ajustas algo?"

## Reglas del skill
- Output = SOLO el prompt (y sus preguntas). Nada de ejecutar, editar archivos ni deploys.
- Todo file:line citado debe existir (verificado con grep en el paso 2).
- Si la idea son varias tareas independientes, propón dividir en 2-3 prompts y entrega
  el primero.
- Si la tarea es trivial (un fix de una línea), dilo honestamente: "esto no necesita
  prompt, lo hago en 1 minuto — ¿procedo?" en vez de ceremonia vacía.
- Longitud objetivo: 300-600 palabras — blindado pero ejecutable, no burocracia.
