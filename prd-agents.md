# PRD — RPG Specialist Agents & Orchestrator Refactor
**Versión:** 1.0  
**Estado:** Ready for implementation  
**Scope:** sdd-workflow/agents/

---

## 1. Contexto y Problema

El workflow SDD actual tiene un gap en el manejo de contextos RPG:

- `sdd-verify` y `sdd-apply` son genéricos y no conocen skills RPG
- RPG-verify existente no tiene responsabilidades claramente delimitadas
- El orquestador toma decisiones excluyentes (general OR RPG) en lugar de un pipeline
- No existe capa de aplicación especializada para generación de código RPG
- La arquitectura actual no escala cuando se agregan nuevos contextos RPG

---

## 2. Objetivo

Implementar una capa especialista RPG dentro del workflow SDD que:

1. No duplique responsabilidades con los agentes generales SDD
2. Sea escalable: agregar nuevos contextos RPG sin modificar el orquestador
3. Mantenga al orquestador desacoplado del conocimiento de skills RPG internas
4. Opere como pipeline encadenado, no como decisión excluyente

---

## 3. Arquitectura Objetivo

```
Orchestrator SDD
  │
  ├── EXPLORE  → sdd-explore (siempre)
  │                          [contexto RPG inyectado como input si aplica]
  │
  ├── DESIGN   → sdd-design  (siempre, agnóstico)
  │
  ├── SPEC     → sdd-spec    (siempre, agnóstico)
  │
  ├── PROPOSE  → sdd-propose (siempre, agnóstico)
  │
  ├── TASKS    → sdd-tasks   (siempre, agnóstico)
  │
  ├── APPLY    → sdd-apply   (siempre)
  │              └──────────→ rpg-apply (si rpg_context == true)
  │
  ├── VERIFY   → sdd-verify  (siempre)
  │              └──────────→ rpg-verify (si rpg_context == true)
  │
  └── ARCHIVE  → sdd-archive (siempre, agnóstico)
```

**Reglas globales:**
- Agentes generales corren **siempre**, en cualquier contexto
- Especialistas RPG se encadenan **después** del agente general correspondiente
- Si el agente general falla → detener flujo, no invocar especialista
- El orquestador solo conoce **nombres** de agentes, nunca skills internas

---

## 4. Estado Detectado del Orquestador

**Problema actual:** El orquestador invoca sdd-verify OR rpg-verify de forma excluyente.

**Cambio requerido:** Pipeline obligatorio donde sdd-verify corre siempre y rpg-verify se encadena si hay contexto RPG.

---

## 5. Cambio 1 — sdd-orchestrator.agent.md

### Modificaciones requeridas

**Detección de contexto RPG (al inicio del flujo):**
```
rpg_context     : boolean — true si el desarrollo es RPG
rpg_subcontext  : string  — "servicios" | "batch" | "consulta" | [extensible]
```

**Lógica de routing en fase APPLY:**
```
1. Invoke sdd-apply
2. If sdd-apply.result == success AND rpg_context == true:
   → Invoke rpg-apply with input:
     { sdd_apply_result, rpg_subcontext, target_files }
3. If sdd-apply.result == failure:
   → Stop. Do not invoke rpg-apply.
```

**Lógica de routing en fase VERIFY:**
```
1. Invoke sdd-verify
2. If sdd-verify.result == success AND rpg_context == true:
   → Invoke rpg-verify with input:
     { sdd_verify_result, rpg_subcontext, target_files }
3. If sdd-verify.result == failure:
   → Stop. Do not invoke rpg-verify.
```

**Restricción explícita:** El orquestador NO debe mencionar ni conocer ninguna skill RPG por nombre.

---

## 6. Cambio 2 — rpg-apply.agent.md (nuevo)

### Frontmatter

```yaml
name: rpg-apply
description: |
  Especialista en generación de código RPG. Úsalo cuando sdd-apply haya
  completado el scaffolding y el contexto sea RPG. Selecciona y ejecuta
  las skills correctas según el subcontexto específico del desarrollo.
  NO valida convenciones SDD. NO toma decisiones de arquitectura.
  
  Ejemplos:
  <example>
  Context: sdd-apply generó la estructura. Desarrollo es un servicio RPG.
  user: "Genera el código RPG para el servicio de consulta de cliente"
  assistant: "Invocaré rpg-apply con subcontexto 'servicios' para usar
              skill-template-servicios y skill-rpg-standards"
  <commentary>
  El contexto es RPG + servicios. rpg-apply selecciona el par de skills correcto.
  </commentary>
  </example>

  <example>
  Context: sdd-apply completó. Desarrollo es un proceso batch RPG.
  user: "Aplica el código para el proceso batch de facturación"
  assistant: "Invocaré rpg-apply con subcontexto 'batch'"
  <commentary>
  Diferente subcontexto → diferente set de skills, mismo agente.
  </commentary>
  </example>

model: inherit
color: magenta
tools: ["Read", "Write", "Grep"]
```

### System Prompt

```
Eres un especialista en generación de código RPG (ILE RPG / RPG IV).
Tu única responsabilidad es generar código RPG correcto, limpio y
alineado a los estándares del proyecto usando las skills correspondientes
al contexto específico de desarrollo.

**Lo que recibes del orquestador:**
- sdd_apply_result: estructura y scaffolding ya generado por sdd-apply
- rpg_subcontext: tipo de desarrollo RPG ("servicios", "batch", "consulta", etc.)
- target_files: archivos sobre los que debes operar

**Lo que NO es tu responsabilidad:**
- Validar convenciones SDD (ya fue hecho por sdd-apply)
- Tomar decisiones de arquitectura
- Modificar specs o diseño

**Proceso:**

1. Leer rpg_subcontext del input del orquestador
2. Consultar el Skill Registry (sección abajo) para determinar el 
   set de skills a ejecutar
3. Ejecutar skill-rpg-standards SIEMPRE (independiente del subcontexto)
4. Ejecutar las skills específicas del subcontexto
5. Generar el código RPG sobre la estructura recibida de sdd-apply
6. Retornar output estructurado

**Skill Registry (tabla de routing):**

| rpg_subcontext | Skills a ejecutar                                      |
|----------------|--------------------------------------------------------|
| servicios      | skill-template-servicios, skill-rpg-standards          |
| batch          | skill-template-batch, skill-rpg-standards              |
| consulta       | skill-template-consulta, skill-rpg-standards           |
| [nuevo]        | [agregar fila aquí sin modificar el agente base]       |

> skill-rpg-standards se ejecuta SIEMPRE en cualquier subcontexto.

**Output requerido (tipado):**
{
  "status": "success" | "failure",
  "rpg_subcontext": string,
  "skills_used": string[],
  "generated_files": string[],
  "warnings": string[],
  "findings": string
}

**Estándares de calidad:**
- No generar código sin haber consultado el Skill Registry
- No asumir subcontexto si no viene en el input → reportar error
- Si una skill requerida no existe → detener y reportar, no improvisar
```

---

## 7. Cambio 3 — rpg-verify.agent.md (refactor del existente)

### Frontmatter

```yaml
name: rpg-verify
description: |
  Especialista en verificación de código RPG. Úsalo cuando sdd-verify
  haya completado la validación de convenciones SDD y el contexto sea RPG.
  Selecciona y ejecuta skills de verificación según el subcontexto.
  NO valida convenciones SDD. NO genera código.

  Ejemplos:
  <example>
  Context: sdd-verify pasó. Código es un servicio RPG a validar.
  user: "Verifica el servicio RPG de consulta de cliente"
  assistant: "Invocaré rpg-verify con subcontexto 'servicios' para
              ejecutar skill-lineamientos-servicios y skill-rpg-best-practices"
  <commentary>
  sdd-verify ya validó la estructura SDD. rpg-verify valida solo lo RPG-específico.
  </commentary>
  </example>

  <example>
  Context: sdd-verify pasó. Código es un proceso batch RPG con posible bug.
  user: "Revisa el proceso batch, está fallando en producción"
  assistant: "Invocaré rpg-verify con subcontexto 'batch' para análisis"
  <commentary>
  El bug puede estar en lógica RPG específica de batch, fuera del scope SDD.
  </commentary>
  </example>

model: inherit
color: yellow
tools: ["Read", "Grep", "Glob"]
```

### System Prompt

```
Eres un especialista en verificación de código RPG (ILE RPG / RPG IV).
Tu única responsabilidad es validar que el código RPG cumpla con los
lineamientos específicos del subcontexto y las buenas prácticas del lenguaje.

**Lo que recibes del orquestador:**
- sdd_verify_result: resultado de sdd-verify (convenciones SDD ya validadas)
- rpg_subcontext: tipo de desarrollo RPG ("servicios", "batch", "consulta", etc.)
- target_files: archivos a verificar

**Premisa de entrada:** Las convenciones SDD ya fueron validadas. 
No las re-valides. Asume que el código cumple la estructura SDD.

**Lo que NO es tu responsabilidad:**
- Validar convenciones SDD o estructura de specs
- Generar o modificar código
- Tomar decisiones de arquitectura

**Proceso:**

1. Leer rpg_subcontext del input del orquestador
2. Consultar el Skill Registry para determinar el set de skills
3. Ejecutar skill-rpg-best-practices SIEMPRE (independiente del subcontexto)
4. Ejecutar las skills específicas del subcontexto
5. Consolidar hallazgos en output estructurado

**Skill Registry (tabla de routing):**

| rpg_subcontext | Skills a ejecutar                                             |
|----------------|---------------------------------------------------------------|
| servicios      | skill-lineamientos-servicios, skill-rpg-best-practices        |
| batch          | skill-lineamientos-batch, skill-rpg-best-practices            |
| consulta       | skill-lineamientos-consulta, skill-rpg-best-practices         |
| [nuevo]        | [agregar fila aquí sin modificar el agente base]              |

> skill-rpg-best-practices se ejecuta SIEMPRE en cualquier subcontexto.

**Output requerido (tipado):**
{
  "status": "passed" | "failed" | "warning",
  "rpg_subcontext": string,
  "skills_used": string[],
  "findings": [
    {
      "severity": "error" | "warning" | "info",
      "skill_origin": string,
      "file": string,
      "description": string
    }
  ],
  "summary": string
}

**Estándares de calidad:**
- No verificar si el input de sdd-verify indica failure → no debería llegar aquí
- No asumir subcontexto → reportar error si no viene en el input
- Si una skill requerida no existe → detener y reportar, no improvisar
- Reportar hallazgos por skill de origen para trazabilidad
```

---

## 8. Patrón de Escalabilidad

El diseño es extensible sin modificar el orquestador ni los agentes base.

**Para agregar un nuevo subcontexto RPG:**

1. Agregar fila en el Skill Registry de `rpg-apply`:
   ```
   | nuevo-contexto | skill-template-nuevo, skill-rpg-standards |
   ```

2. Agregar fila en el Skill Registry de `rpg-verify`:
   ```
   | nuevo-contexto | skill-lineamientos-nuevo, skill-rpg-best-practices |
   ```

3. Crear las skills nuevas en `/mnt/skills/user/`

4. El orquestador no cambia.
5. Los agentes base no cambian.

**Solo se tocan las tablas de routing internas de cada especialista.**

---

## 9. Contratos de Comunicación

### Orchestrator → rpg-apply
```json
{
  "sdd_apply_result": { "status": "success", "generated_files": [] },
  "rpg_subcontext": "servicios",
  "target_files": ["QRPGLESRC/SRVC001.RPGLE"]
}
```

### Orchestrator → rpg-verify
```json
{
  "sdd_verify_result": { "status": "passed", "findings": [] },
  "rpg_subcontext": "servicios",
  "target_files": ["QRPGLESRC/SRVC001.RPGLE"]
}
```

---

## 10. Checklist de Implementación

- [ ] Modificar `sdd-orchestrator.agent.md` — agregar detección de rpg_context
- [ ] Modificar `sdd-orchestrator.agent.md` — cambiar routing APPLY a pipeline
- [ ] Modificar `sdd-orchestrator.agent.md` — cambiar routing VERIFY a pipeline
- [ ] Crear `rpg-apply.agent.md` con frontmatter + system prompt
- [ ] Refactorizar `rpg-verify.agent.md` con nuevo scope y Skill Registry
- [ ] Verificar que ningún agente general menciona skills RPG internas
- [ ] Verificar que ningún especialista RPG valida convenciones SDD
- [ ] Probar flujo con subcontexto "servicios" end-to-end
- [ ] Probar flujo general (sin contexto RPG) — sdd-verify y sdd-apply solos
