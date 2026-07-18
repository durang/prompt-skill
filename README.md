# Claude/Codex PROMPT SKILL — `/prompt`

El **afilador de prompts**: convierte una petición en bruto (español informal, typos,
ideas sueltas) en **UN prompt profesional y blindado** listo para ejecutar — sin
ejecutar nada. Tú revisas el prompt y decides.

## Qué te devuelve
- **Contexto real del repo** (rutas verificadas con grep — nunca inventadas)
- **Objetivo medible** en una frase
- **Fases**: investigar → cambiar → **probar EN VIVO con evidencia** (screenshot/curl)
- **Pruebas de conexión** punta a punta (dato → BD → sitio → verificación externa)
- **Reglas duras** (aislamiento, canónico, PII, go-words de producción)
- **Entregable exacto** + asunciones marcadas
- Cierre: *"¿Lo ejecuto tal cual o ajustas algo?"*

## Instalar

### Claude Code
```bash
mkdir -p ~/.claude/skills/prompt
curl -o ~/.claude/skills/prompt/SKILL.md \
  https://raw.githubusercontent.com/durang/prompt-skill/master/skills/prompt/SKILL.md
```
O clona el repo y copia `skills/prompt/` a `~/.claude/skills/` (global) o
`.claude/skills/` (por proyecto).

### Codex / otros agentes
Pega el contenido de `skills/prompt/SKILL.md` como instrucción de sistema o
snippet reutilizable — es markdown puro, sin dependencias.

## Uso
```
/prompt quiero notificaciones push cuando llegue un lead y que se vean en el cel
```
→ Devuelve el prompt blindado. Nada se ejecuta hasta que tú lo pegues/apruebes.

## Filosofía
- **Anti-burocracia**: si la tarea es trivial, lo dice ("esto no necesita prompt").
- **Canónico**: el prompt generado usa los sistemas existentes del repo en vez de duplicar.
- **Evidencia o no pasó**: "deploy succeeded" nunca cuenta como verificación.

MIT — úsalo, cámbialo, compártelo.
