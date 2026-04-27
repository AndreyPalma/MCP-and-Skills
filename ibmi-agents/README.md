# Arquitectura de Agentes IBM i v2 — 6 Agentes Adaptativos

## Principio de diseño
Un solo flujo que se adapta automáticamente a cualquier complejidad,
desde cambiar la lógica de un subprocedimiento hasta implementar un PRD completo.

## Estructura de archivos

```
tu-proyecto/
└── .claude/
    └── agents/
        ├── 01-intake-agent.md       ← Captura el requerimiento
        ├── 02-router-agent.md       ← Clasifica complejidad y enruta
        ├── 03-direct-agent.md       ← Ejecución directa (SIMPLE)
        ├── 04-predev-agent.md       ← Análisis + spec (MEDIO/COMPLEJO/ANÁLISIS)
        ├── 05-rpg-orchestrator.md   ← Desarrollo RPG ILE
        ├── 06-db2-orchestrator.md   ← Desarrollo DB2 for i
        └── 07-postdev-agent.md      ← Verificación + documentación
```

## Flujos por nivel de complejidad

### SIMPLE (2 agentes activos)
```
intake → router → direct-agent → postdev
```
Para: cambios puntuales en 1 programa o 1 tabla, sin análisis previo necesario.

### MEDIO (4 agentes activos)
```
intake → router → predev-agent → [tu aprobación] → rpg-orch o db2-orch → postdev
```
Para: refactorizaciones, módulos nuevos de dominio único, análisis requerido.

### COMPLEJO (5 agentes activos)
```
intake → router → predev-agent → [tu aprobación] → db2-orch → [DB2 estable] → rpg-orch → postdev
```
Para: implementación de PRD, cambios cross-domain, múltiples componentes.

### ANÁLISIS PURO (3 agentes activos)
```
intake → router → predev-agent (modo análisis) → postdev
```
Para: entender el sistema, mapear impactos sin modificar nada.

## Tus puntos de aprobación

| Momento | Qué apruebas | Rutas |
|---------|--------------|-------|
| Después del router | Clasificación de complejidad | Todas |
| Después de predev | Technical Spec antes de codificar | MEDIO, COMPLEJO |
| Entre DB2 y RPG | DB2 estable antes de modificar RPG | Solo COMPLEJO |

## Estructura OpenSpec recomendada

```
tu-proyecto/
└── openspec/
    ├── specs/
    │   ├── programs/[NOMBRE]/spec.md
    │   ├── tables/[NOMBRE]/spec.md
    │   └── cross/[NOMBRE]/spec.md
    └── changes/
        └── REQ-[ID]/
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            ├── technical-spec.md
            ├── ddl-applied.sql      ← generado por db2-orchestrator
            └── execution-summary.md ← generado por postdev-agent
```

## Cuándo agregar subagentes

Los orquestadores manejan explorar+aplicar+verificar internamente.
Solo agrega subagentes separados si un programa RPG supera ~800 líneas
de lógica relevante al cambio y el contexto se satura. En ese caso
agrega `rpg-explorar`, `rpg-aplicar`, `rpg-verificar` como archivos
separados y actualiza el rpg-orchestrator para que los invoque con Task.