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

## Paso 1 — Entender la intención real
- Qué quiere lograr el NEGOCIO, no solo qué pidió. Interpreta typos/español informal
  con generosidad; si hay 2 lecturas, elige la más valiosa y márcala como "Asunción".
- Detecta el **tipo de tarea** (define las fases del paso 3):
  `auditoría` · `feature nueva` · `fix/bug` · `migración/datos` · `diseño/UI` · `integración externa`.
- Si la idea son varias tareas independientes → propón dividir y entrega el prompt de la primera.

## Paso 2 — Investigación estructurada (2-5 min, SIEMPRE antes de escribir)
Checklist mínimo — todo lo citado en el prompt debe salir de aquí, no de memoria:
1. **Rutas y símbolos reales**: grep de los archivos/funciones que el prompt va a citar
   (file:line verificados — un prompt que cita rutas inventadas produce agentes perdidos).
2. **Sistemas canónicos existentes** que el prompt DEBE usar en vez de duplicar
   (en Vendau: `lib/capabilities.ts` + `cap()/resolveCap`, `lib/setup-score.ts`,
   `lib/lead-labels.ts`, `lib/brand.ts`/`brand.*`, `lib/templates.ts`, patrón de server
   actions con merge + `fail()` + `revalidatePath`, guard `requireTenantMember`).
3. **Dependencias y radio de impacto**: ¿quién más consume lo que se va a tocar?
   (grep de imports/usos). ¿Toca los DOS verticales? ¿Toca crons? ¿Toca el bundle cliente
   (server-only leaks)?
4. **Datos y envs**: qué tablas/campos reales participan (nombres verificados) y qué
   variables de entorno se necesitan — **solo NOMBRES, jamás valores**.
5. **Riesgo de producción**: ¿reversible (código = revert) o irreversible (datos)?
   ¿Necesita go-word (`PUBLICAR` código / `PUBLICAR-BD` datos)? ¿Hay que ejecutar crons
   una vez a mano tras cambiarlos?

## Paso 3 — Escribir el prompt (plantilla)
- **Título** (una línea con emoji)
- **Contexto** — estado real relevante con rutas/sistemas confirmados en el paso 2
- **Objetivo** — medible, en UNA frase ("certificar que…", "dejar en vivo…")
- **Fases numeradas** según el tipo de tarea:
  - *Auditoría*: lentes en paralelo (3-5 agentes con foco distinto y schema de findings
    con file:line+fix+severity) → síntesis dedup/priorizada → corrección high/medium →
    re-verificación con evidencia.
  - *Feature*: investigar → diseñar canónico (¿dónde vive la verdad única?) → implementar
    → cablear consumidores → probar en vivo → deploy.
  - *Fix*: reproducir PRIMERO (evidencia del bug) → causa raíz (no el síntoma) → fix
    mínimo → probar que ya no reproduce → verificar que nada más se rompió.
  - *Migración/datos*: dry-run + snapshot + reversal ensayada ANTES del forward +
    go-word `PUBLICAR-BD` + verificación externa después.
  - *Diseño/UI*: direcciones anti-convergentes (shotgun) → elegir → construir template-aware
    → verificar en vivo desktop+móvil con screenshots.
  - Siempre separar: **investigar → cambiar → probar EN VIVO con evidencia**.
- **Pruebas de conexión** — punta a punta (dato → BD → sitio/panel → verificación EXTERNA
  con curl/screenshot). Incluir SIEMPRE el caso multi-tenant si aplica: "¿funciona igual
  para un cliente nuevo de otro giro/idioma con datos vacíos?" y el caso "tenant nuevo
  sin filas" (null-safety).
- **Reglas duras** — las del proyecto (aislamiento autos↔casas byte-idéntico, verdad
  única/canónico, nada de PII/secretos —solo nombres de envs—, BD prod solo-lectura salvo
  prueba reversible y revertida, go-words) + las específicas de la tarea.
- **Entregable** — formato exacto (tabla estado/probado/corregido/pendiente, veredictos
  con evidencia, top-N priorizado por impacto en el dueño del negocio).
- **Asunciones y preguntas** — lo asumido + lo que Sergio debería confirmar.
- Cierre: "¿Lo ejecuto tal cual o ajustas algo?"

## Paso 4 — Auto-check antes de entregar (si falla uno, corrige el prompt)
- [ ] ¿El objetivo cabe en una frase y es verificable?
- [ ] ¿Cada file:line citado existe (lo verifiqué con grep)?
- [ ] ¿Toda afirmación final exigirá EVIDENCIA (screenshot/curl), nunca "deploy succeeded"?
- [ ] ¿Está definido qué pasa con el OTRO vertical/tenant/idioma?
- [ ] ¿Está claro qué es reversible y qué necesita go-word?
- [ ] ¿El entregable tiene formato exacto (no "haz un reporte" a secas)?
- [ ] ¿Dimensioné los agentes al tamaño real (3-5 lentes para auditar; 0 agentes para un fix simple)?

## Anti-patrones (nunca en el prompt entregado)
- Verbos vagos sin criterio de éxito ("mejora", "revisa") sin definir qué es "mejor/bien".
- Pedir cambios sin pedir la prueba de que funcionaron.
- Mezclar 3 tareas en un prompt (divide).
- Citar archivos/sistemas de memoria sin verificar.
- Ceremonia vacía: si la tarea es trivial, di honesto "esto no necesita prompt, lo hago
  en 1 minuto — ¿procedo?".

## Ejemplo (entrada → esencia de la salida)
Entrada: `/prompt quiero q el % del dashboard tambien cuente si probaron la llamada IA`
Salida (esencia): título; contexto con `lib/setup-score.ts` y la tabla `voice_calls`
verificadas por grep; objetivo "agregar paso 13 'Primera llamada IA' con dato real";
fases feature (agregar item al registro → query en dashboard → probar E2E subiendo/bajando
el dato → deploy); prueba de conexión (insertar fila de prueba → % sube → borrar → baja);
reglas (registro único, no tocar autos); entregable (screenshot antes/después del %);
cierre "¿Lo ejecuto?".

## Guardado selectivo (no todo merece archivo)
El contexto de la conversación se compacta y muere — pero guardar todo es basura.
Regla: al entregar el prompt, clasifícalo:
- **REUTILIZABLE** (auditorías periódicas, certificación de cliente nuevo, plantillas de
  migración/onboarding — cualquier cosa que se volverá a disparar): ofrece guardarlo en
  `.planning/prompts/AAAA-MM-DD-<slug>.md` del proyecto y guárdalo si Sergio acepta
  (o si él ya pidió historial). Al re-usarlo después, actualiza los file:line con un
  grep rápido antes de disparar (el repo cambió desde entonces).
- **ONE-SHOT** (un fix, un campo, una tarea puntual): no ofrezcas guardar — el valor
  vive en ejecutarlo, no en archivarlo.

## Longitud objetivo
300-600 palabras — blindado pero ejecutable, no burocracia.
