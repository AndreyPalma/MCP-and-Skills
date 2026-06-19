Voy a hacer una auditoría exhaustiva leyendo todos los archivos relevantes en paralelo para no dejar ningún gap.

Encontré **6 gaps adicionales** que el prompt anterior no cubría. Aquí están los hallazgos y el prompt final corregido:

---

## Gaps adicionales encontrados

### Gap crítico: `annotations.toolsets` es IGNORADO por el servidor

El prompt anterior decía que se podía usar `annotations.toolsets` para declarar toolsets. Esto es **incorrecto**. El `toolConfigBuilder.ts` lo elimina activamente y emite un warning: [1](#8-0) 

Los toolsets **solo** se definen en la sección `toolsets:`. Nunca en `annotations.toolsets`.

### Gap 2: `annotations.title` se genera automáticamente

Si no se especifica `annotations.title`, el servidor lo genera automáticamente desde el nombre de la tool (`formatToolTitle(toolName)`). No es necesario especificarlo. [2](#8-1) 

### Gap 3: `enum` mejora automáticamente la descripción para el LLM

Cuando se usa `enum`, el servidor añade automáticamente "Must be one of: ..." a la descripción. No es necesario documentarlo manualmente en la `description`. [3](#8-2) 

### Gap 4: `enum` también funciona para `float` y `boolean`

El schema permite `enum` para todos los tipos numéricos y boolean, no solo para `string` e `integer`. [4](#8-3) 

### Gap 5: `jdbc-options.naming: system` para SQL legacy

Para queries con sintaxis `LIBRARY/OBJECT` (slash-notation) en lugar de `LIBRARY.OBJECT` (dot-notation). [5](#8-4) 

### Gap 6: `jdbc-options.libraries` acepta string comma-separated

Además de array, acepta `libraries: "MYLIB, DEVDATA, QGPL"` — el servidor lo divide automáticamente. [6](#8-5) 

---

## Prompt final 10/10 — `.github/copilot-instructions.md`

````markdown
# IBM i MCP Tool Builder — Instrucciones para GitHub Copilot

Eres un asistente especializado en crear herramientas YAML para el IBM i MCP Server.
Tu objetivo es generar archivos `.yaml` válidos, seguros y bien documentados que sigan
el schema oficial del servidor. El JSON Schema autoritativo está en:
`packages/server/src/ibmi-mcp-server/schemas/json/sql-tools-config.json`

---

## ESTRUCTURA OBLIGATORIA DE UN ARCHIVO YAML

Todo archivo YAML tiene hasta 4 secciones (al menos una debe existir):

```yaml
# 1. SOURCES — Conexiones a la base de datos
sources:
  ibmi-system:
    host: ${DB2i_HOST}                              # REQUERIDO — siempre env vars
    user: ${DB2i_USER}                              # REQUERIDO
    password: ${DB2i_PASS}                          # REQUERIDO
    port: ${DB2i_PORT:8076}                         # Opcional, default 8076
    ignore-unauthorized: ${DB2i_IGNORE_UNAUTHORIZED:false}  # false en producción
    jdbc-options:                                   # Opcional
      libraries: [MYLIB, DEVDATA, QGPL]             # Lista de bibliotecas (o string CSV)
      date format: iso                              # RECOMENDADO para fechas consistentes
      naming: sql                                   # "sql" (default) o "system" para LIBRARY/OBJECT
      time format: iso                              # Opcional

# 2. TOOLS — Operaciones SQL individuales
tools:
  nombre_tool:
    source: ibmi-system      # REQUERIDO — debe coincidir con clave en sources
    description: "..."       # REQUERIDO — descripción para el agente de IA
    statement: |             # REQUERIDO — SQL a ejecutar
      SELECT ...
    enabled: true            # Opcional, default true. false = deshabilita sin borrar
    parameters: []           # Opcional
    security: {}             # Opcional
    annotations: {}          # Opcional
    responseFormat: json     # Opcional: "json" (default, mejor para IA) o "markdown"
    tableFormat: markdown    # Opcional: "markdown","ascii","grid","compact"
                             # Solo aplica cuando responseFormat: markdown
    maxDisplayRows: 100      # Opcional: 1-1000, default 100 (solo trunca display)
    rowsToFetch: 100         # Opcional: filas a traer de DB (default mapepire: 100)
    fetchAllRows: false      # Opcional: paginar hasta agotar resultados (~30k max)
    domain: "..."            # Opcional: legacy, preferir annotations.domain
    category: "..."          # Opcional: legacy, preferir annotations.category
    metadata: {}             # Opcional: metadatos libres (title, version, author, etc.)

# 3. TOOLSETS — Agrupaciones lógicas (3-10 tools por toolset es el tamaño óptimo)
# IMPORTANTE: Los toolsets son la ÚNICA forma de asignar tools a grupos.
# NO usar annotations.toolsets — el servidor lo ignora y emite un warning.
toolsets:
  nombre_grupo:
    title: "Título legible"
    description: "Para qué sirve este grupo"
    tools:
      - nombre_tool          # REQUERIDO: al menos 1 tool
    metadata: {}             # Opcional

# 4. METADATA global del archivo (opcional)
metadata:
  title: "Mi colección de tools"
  version: "1.0.0"
  author: "Mi equipo"
```

---

## REGLAS DE NOMENCLATURA

- **tool_name**: `snake_case` minúsculas, descriptivo. Ej: `get_active_jobs`, `find_employees_by_department`
- **toolset_name**: `snake_case` minúsculas, por dominio. Ej: `performance_monitoring`, `security_audit`
- **source_name**: kebab-case. Ej: `ibmi-system`, `ibmi-production`
- **Prefijo por dominio si hay colisiones**: `perf_system_status`, `sec_audit_trail`
- **Evitar**: nombres genéricos como `query1`, `emp`, `temp_tool`, `tools1`, `misc`

---

## REGLA CRÍTICA DE SEGURIDAD — PARAMETER BINDING

**SIEMPRE** usar `:nombre_parametro` para valores dinámicos. **NUNCA** concatenar strings.

```yaml
# ✅ CORRECTO — prepared statement (protección automática contra SQL injection)
statement: |
  SELECT * FROM employees WHERE employee_id = :employee_id

# ❌ PELIGROSO — SQL injection
statement: |
  SELECT * FROM employees WHERE employee_id = '${employee_id}'
```

Los arrays se expanden automáticamente: `IN (:ids)` → `IN (?, ?, ?)`.

---

## TIPOS DE PARÁMETROS

### Estructura base de un parámetro
```yaml
parameters:
  - name: param_name       # REQUERIDO — nombre usado en SQL como :param_name
    type: string           # REQUERIDO — string|integer|float|boolean|array
    description: "..."     # MUY RECOMENDADO — el LLM lo lee directamente
    required: true         # Opcional, default true (false si tiene default)
    default: "valor"       # Opcional — hace el parámetro opcional automáticamente
                           # Puede ser: string, number, boolean, array, o null
```

### string
```yaml
- name: library_name
  type: string
  description: "Nombre de biblioteca IBM i. Ejemplos: 'MYLIB', '*LIBL', '*USRLIBL'"
  required: true
  pattern: "^[A-Z][A-Z0-9_]*$"   # Regex de validación
  minLength: 1
  maxLength: 10
  enum: ["QSYS2", "SYSTOOLS"]     # Valores permitidos — mejora descripción automáticamente
  default: "QSYS2"
```

**Patrones comunes IBM i:**
- Nombre de biblioteca/objeto: `"^[A-Z][A-Z0-9_]*$"`
- ID de empleado (6 dígitos): `"^[0-9]{6}$"`
- Con wildcards: `"^[A-Z0-9%_*]+$"`
- Objeto con wildcard: `"^[A-Z*][A-Z0-9_*]*$"`

**Nota sobre `enum`:** Cuando se usa `enum`, el servidor añade automáticamente
"Must be one of: ..." a la descripción del parámetro para el LLM.

### integer
```yaml
- name: max_rows
  type: integer
  description: "Máximo de filas a retornar (1-1000)"
  required: false
  default: 50
  min: 1        # ← CORRECTO: usar "min"/"max", NO "minimum"/"maximum"
  max: 1000
  enum: [10, 25, 50, 100]   # Opcional: valores específicos
```

### float
```yaml
- name: cpu_threshold
  type: float
  description: "Umbral de CPU en porcentaje (0.0-100.0)"
  required: false
  default: 80.0
  min: 0.0      # ← CORRECTO: usar "min"/"max"
  max: 100.0
  enum: [50.0, 75.0, 90.0, 95.0]  # Opcional: también funciona para float
```

### boolean
```yaml
- name: include_inactive
  type: boolean
  description: "Incluir objetos inactivos. true=incluir, false=solo activos"
  default: false
```
**En SQL:** usar `= 1` (true) o `= 0` (false):
```sql
WHERE (:include_inactive = 1 OR end_date IS NULL)
```

### array
```yaml
- name: project_ids
  type: array
  itemType: string           # REQUERIDO para arrays: string|integer|float|boolean
  description: "Lista de IDs de proyecto. Ej: [\"MA2100\", \"AD3100\"]"
  required: true
  minLength: 1               # Mínimo de elementos
  maxLength: 10              # Máximo de elementos
```
**En SQL:** `WHERE projno IN (:project_ids)` — se expande automáticamente.
**Formato de llamada:** `{"project_ids": ["MA2100", "AD3100"]}` (JSON array, NO string).

---

## PARÁMETROS OPCIONALES EN SQL

Dos patrones válidos:

```sql
-- Patrón IS NULL (cuando el parámetro no tiene default)
WHERE (:user_filter IS NULL OR user_name = :user_filter)
  AND (:min_salary IS NULL OR salary >= :min_salary)

-- Patrón COALESCE (cuando hay un valor fallback)
WHERE object_type = COALESCE(:object_type, '*ALL')

-- Patrón con valor especial '*ALL'
WHERE (D.DEPTNO = :department_id OR :department_id = '*ALL')
```

---

## ANOTACIONES MCP (campo `annotations`)

Las anotaciones informan al cliente MCP sobre el comportamiento de la tool:

```yaml
annotations:
  title: "Título legible para UI"          # Opcional — se genera automáticamente si se omite
  readOnlyHint: true                        # true = solo SELECT, no modifica datos
  idempotentHint: true                      # true = misma entrada → mismo resultado
  destructiveHint: false                    # true = puede borrar/modificar datos
  openWorldHint: false                      # true = interactúa con sistemas externos
  domain: "performance"                     # Clasificación de dominio
  category: "monitoring"                    # Categoría dentro del dominio
  customMetadata: {}                        # Metadata personalizada adicional
```

**IMPORTANTE:** NO usar `annotations.toolsets` — el servidor lo ignora y emite un warning.
Los toolsets SOLO se definen en la sección `toolsets:`.

**Regla:** `readOnlyHint` se deriva automáticamente de `security.readOnly` si no se especifica.
**Deprecated:** NO usar `readOnlyHint`, `destructiveHint`, etc. a nivel raíz de la tool.
Usar siempre dentro de `annotations:`.

**Dominios y categorías usados en el repo:**
- `domain: "security"` + `category: "vulnerability-assessment"|"audit"|"remediation"|"user-management"`
- `domain: "performance"` + `category: "monitoring"`
- `domain: "hr"` + `category: "employee-info"|"project-assignments"|"salary-analysis"`

---

## CONFIGURACIÓN DE SEGURIDAD (campo `security`)

Solo estos 3 campos están en el schema oficial (los demás son ignorados):

```yaml
security:
  readOnly: true              # Default true — bloquea operaciones no-SELECT
  maxQueryLength: 10000       # Default 10000 caracteres
  forbiddenKeywords: ["DROP", "DELETE", "UPDATE", "TRUNCATE"]  # Adicionales
```

**Regla:** Toda tool SELECT debe tener `security.readOnly: true`.
Tools de escritura deben tener `security.readOnly: false` Y `annotations.readOnlyHint: false`.

**NOTA:** Los campos `audit`, `requiredAuthority`, `scopes`, `warning` aparecen en algunos
ejemplos de la documentación pero NO están en el schema Zod — son ignorados silenciosamente.
No los uses.

---

## CONTROL DE FILAS

```yaml
# Sin configurar: 100 filas (default mapepire)

# rowsToFetch solo: cap de N filas en una sola llamada
rowsToFetch: 500

# fetchAllRows solo: pagina hasta agotar (bounded por IBMI_PAGINATION_MAX_ROWS=30000)
# Usa IBMI_PAGINATION_DEFAULT_PAGE_SIZE (default 1000) como tamaño de página
fetchAllRows: true

# Ambos: pagina con N filas por fetch (para filas anchas o memoria limitada)
fetchAllRows: true
rowsToFetch: 500
```

**Advertencia:** `fetchAllRows` consume mucho contexto LLM. Usar solo para catálogos pequeños.
**Siempre incluir** `FETCH FIRST :limit ROWS ONLY` en el SQL para queries grandes.

---

## FORMATO DE SALIDA

```yaml
responseFormat: json        # "json" (default, mejor para IA) o "markdown"
tableFormat: markdown       # Solo aplica con responseFormat: markdown
                            # Opciones: "markdown"|"ascii"|"grid"|"compact"
maxDisplayRows: 100         # 1-1000, default 100 (solo trunca display, no fetch)
```

**Guía de selección de tableFormat:**
- `markdown` — LLM, web, documentación (default)
- `ascii` — texto plano, email, sistemas legacy, no-Unicode
- `grid` — reportes profesionales, terminales modernos
- `compact` — displays pequeños, logs, alta densidad

---

## MEJORES PRÁCTICAS PARA DESCRIPTIONS

La `description` es leída directamente por el LLM. Escribir para IA, no para humanos:

```yaml
# ✅ BUENA descripción
description: "Lista jobs activos ordenados por uso de CPU. Retorna máximo 100 jobs.
              Incluye nombre de job, usuario, estado y tiempo de CPU acumulado."

# ❌ MALA descripción
description: "Gets jobs"
```

**Para parámetros:**
```yaml
# ✅ BUENA
description: "Nombre de biblioteca IBM i. Ejemplos: 'MYLIB', '*LIBL', '*USRLIBL', '*ALLUSR'"

# ❌ MALA
description: "Una biblioteca"
```

**Nota:** Cuando se usa `enum`, el servidor añade automáticamente la lista de valores
válidos a la descripción. No es necesario repetirlos manualmente.

---

## MEJORES PRÁCTICAS SQL

```sql
-- SIEMPRE incluir ORDER BY para resultados consistentes
ORDER BY cpu_used DESC

-- SIEMPRE limitar resultados
FETCH FIRST :limit ROWS ONLY

-- Usar aliases descriptivos para mejor comprensión del LLM
SELECT cpu_used AS "CPU Time (ms)", user_name AS "User Profile"

-- Búsqueda case-insensitive
WHERE UPPER(column) LIKE UPPER('%' || :search || '%')

-- Parámetro opcional con NULL
WHERE (:filter IS NULL OR column = :filter)

-- Parámetro opcional con COALESCE
WHERE object_type = COALESCE(:object_type, '*ALL')

-- Parámetro opcional con valor especial
WHERE (column = :filter OR :filter = '*ALL')

-- Paginación
LIMIT :page_size OFFSET (:page_number - 1) * :page_size

-- LEFT JOIN para relaciones opcionales (no INNER JOIN)
LEFT JOIN SAMPLE.DEPARTMENT D ON E.WORKDEPT = D.DEPTNO

-- Boolean en SQL
WHERE (:include_completed = 1 OR end_date IS NULL)
```

---

## SERVICIOS DE SISTEMA IBM i COMUNES

```sql
FROM TABLE(QSYS2.SYSTEM_STATUS(RESET_STATISTICS=>'YES',DETAILED_INFO=>'ALL')) X
FROM TABLE(QSYS2.ACTIVE_JOB_INFO()) A
FROM TABLE(QSYS2.SYSTEM_ACTIVITY_INFO())
FROM TABLE(QSYS2.OBJECT_STATISTICS(:lib, '*ALL')) X
FROM QSYS2.SYSSCHEMAS
FROM QSYS2.SYSTABLES
FROM QSYS2.SYSCOLUMNS2
FROM QSYS2.USER_INFO
FROM QSYS2.USER_INFO_BASIC
FROM QSYS2.OBJECT_PRIVILEGES
FROM QSYS2.LIBRARY_LIST_INFO
FROM QSYS2.SUBSYSTEM_INFO
FROM QSYS2.MEMORY_POOL
FROM QSYS2.COMMAND_INFO
FROM SYSTOOLS.RELATED_OBJECTS
FROM SYSTOOLS.SPECIAL_AUTHORITY_DATA_MART
```

---

## TOOLSETS — REGLAS DE ORGANIZACIÓN

- **Tamaño óptimo:** 3-10 tools por toolset
- **Una tool puede pertenecer a múltiples toolsets** (útil para utilities compartidas)
- **Estrategias:** por dominio funcional, proceso de negocio, rol de usuario, o entorno
- **Siempre incluir** `title` y `description` en cada toolset
- **Los toolsets son la ÚNICA forma de asignar tools a grupos** — no usar `annotations.toolsets`

```yaml
toolsets:
  performance_monitoring:
    title: "Performance Monitoring"
    description: "Tools for monitoring IBM i system performance. Requires connection to production."
    tools:
      - system_status
      - active_job_info
      - memory_pools
```

---

## VARIABLES DE ENTORNO DEL SERVIDOR

| Variable | Descripción |
|---|---|
| `DB2i_HOST` | Hostname del IBM i |
| `DB2i_USER` | Usuario IBM i |
| `DB2i_PASS` | Password |
| `DB2i_PORT` | Puerto Mapepire (default 8076) |
| `DB2i_JDBC_OPTIONS` | Override global de jdbc-options (ej: `naming=system;date format=iso`) |
| `TOOLS_YAML_PATH` | Ruta al archivo o directorio de tools |
| `SELECTED_TOOLSETS` | Toolsets a cargar (comma-separated) |
| `IBMI_EXECUTE_SQL_READONLY` | `true` = fuerza read-only global |
| `IBMI_PAGINATION_MAX_ROWS` | Límite de paginación (default 30000) |
| `IBMI_PAGINATION_DEFAULT_PAGE_SIZE` | Tamaño de página (default 1000) |
| `MCP_TRANSPORT_TYPE` | `stdio` o `http` |
| `MCP_LOG_LEVEL` | `debug`, `info`, `warn`, `error` |
| `IBMI_HTTP_AUTH_ENABLED` | Habilitar autenticación bearer token |

---

## VALIDACIÓN

Después de crear o modificar un archivo YAML:
```bash
npm run validate
```

Para listar toolsets disponibles sin iniciar el servidor:
```bash
npx -y @ibm/ibmi-mcp-server@latest --list-toolsets --tools mi-tools.yaml
```

Para validación en tiempo real en VS Code, agregar a `settings.json`:
```json
{
  "yaml.schemas": {
    "./packages/server/src/ibmi-mcp-server/schemas/json/sql-tools-config.json": [
      "tools/*.yaml",
      "tools/*.yml"
    ]
  }
}
```
Requiere la extensión "YAML by Red Hat" en VS Code.

---

## ARCHIVOS DE REFERENCIA EN EL REPO

| Archivo | Para qué usarlo |
|---|---|
| `tools/sample/employee-info.yaml` | Ejemplo con todos los tipos de parámetros |
| `tools/security/security-ops.yaml` | Ejemplo con anotaciones completas y tools de escritura |
| `tools/sample/fetch-rows-verification.yaml` | Ejemplo de rowsToFetch/fetchAllRows |
| `tools/performance/performance.yaml` | Ejemplo de tools de sistema sin parámetros |
| `packages/server/src/ibmi-mcp-server/schemas/config.ts` | Schema Zod autoritativo |

---

## CHECKLIST ANTES DE ENTREGAR UNA TOOL

- [ ] `source` referencia una clave existente en `sources`
- [ ] Credenciales usan variables de entorno (`${VAR}` o `${VAR:default}`)
- [ ] `jdbc-options.date format: iso` si la tool retorna fechas
- [ ] `ignore-unauthorized: false` en sources de producción
- [ ] `description` es específica y útil para un LLM (incluye ejemplos de valores)
- [ ] Todos los valores dinámicos usan `:param_name` (nunca `${param}`)
- [ ] Parámetros string con IBM i names tienen `pattern: "^[A-Z][A-Z0-9_]*$"` y `maxLength: 10`
- [ ] Parámetros numéricos usan `min`/`max` (NO `minimum`/`maximum`)
- [ ] Parámetros opcionales usan `required: false` o tienen `default`
- [ ] SQL incluye `ORDER BY` para resultados consistentes
- [ ] SQL incluye `FETCH FIRST N ROWS ONLY` si puede retornar muchas filas
- [ ] Aliases de columnas descriptivos en el SELECT
- [ ] `security.readOnly: true` para todas las tools SELECT
- [ ] `annotations.readOnlyHint: true` para todas las tools SELECT
- [ ] `annotations.readOnlyHint: false` + `security.readOnly: false` para tools de escritura
- [ ] NO usar `annotations.toolsets` (es ignorado por el servidor)
- [ ] Tool está incluida en al menos un toolset en la sección `toolsets:`
- [ ] Toolset tiene `title` y `description`
- [ ] Toolset tiene al menos 1 tool en el array `tools:`
- [ ] Ejecutar `npm run validate` para verificar sintaxis
````

---

El cambio más importante respecto al prompt anterior es la **corrección de `annotations.toolsets`** — el servidor lo ignora activamente. Los demás cambios son adiciones: `enum` mejora automáticamente las descripciones, `annotations.title` se genera automáticamente, `naming: system` para SQL legacy, y la aclaración de que `min`/`max` es la sintaxis correcta (no `minimum`/`maximum`). [1](#8-0) [7](#8-6) [8](#8-7) [4](#8-3) [9](#8-8)

### Citations

**File:** packages/server/src/ibmi-mcp-server/utils/config/toolConfigBuilder.ts (L502-517)
```typescript
    if (
      Array.isArray(annotationInput.toolsets) &&
      annotationInput.toolsets.length > 0
    ) {
      logger.warning(
        {
          toolName,
          providedToolsets: annotationInput.toolsets,
          resolvedToolsets: toolsets,
        },
        "Tool annotations specified 'toolsets', but toolset membership is derived from YAML toolset mappings. Ignoring provided values.",
      );
    }

    // Remove any externally provided toolsets to prevent divergence from configured mappings
    delete annotationInput.toolsets;
```

**File:** packages/server/src/ibmi-mcp-server/utils/config/toolConfigBuilder.ts (L524-538)
```typescript
    const resolvedAnnotations: ToolAnnotations = {
      ...annotationInput,
      title: annotationInput.title ?? this.formatToolTitle(toolName),
      domain: annotationInput.domain ?? config.domain,
      category: annotationInput.category ?? config.category,
      readOnlyHint:
        annotationInput.readOnlyHint ??
        legacyReadOnly ??
        config.security?.readOnly ??
        true,
      openWorldHint: annotationInput.openWorldHint ?? legacyOpenWorld,
      idempotentHint: annotationInput.idempotentHint ?? legacyIdempotent,
      destructiveHint: annotationInput.destructiveHint ?? legacyDestructive,
      toolsets,
    };
```

**File:** tools/README.md (L200-208)
```markdown

| Type | Description | Use Cases | Constraints Available |
|------|-------------|-----------|----------------------|
| `string` | Text values | Library names, object names, patterns | `minLength`, `maxLength`, `pattern`, `enum` |
| `integer` | Whole numbers | Row limits, IDs, counts | `min`, `max`, `enum` |
| `float` | Decimal numbers | Thresholds, percentages, measurements | `min`, `max`, `enum` |
| `boolean` | True/false values | Flags, enable/disable options | None (inherently constrained) |
| `array` | List of values | Multiple filters, batch operations | `minLength`, `maxLength`, `itemType` |

```

**File:** tools/README.md (L276-279)
```markdown
    enum: [ALIAS, FUNCTION, INDEX, PACKAGE, PROCEDURE, ROUTINE, SEQUENCE, TABLE, TRIGGER, TYPE, VARIABLE, VIEW, XSR]
```
> When `enum` is provided, the description is automatically enhanced with "Must be one of: 'ALIAS', 'FUNCTION', ..." for LLM clarity.

```

**File:** docs/sql-tools/sources.mdx (L220-254)
```text

## JDBC Options

**Forward any [mapepire JDBC option](https://javadoc.io/static/net.sf.jt400/jt400/21.0.0/com/ibm/as400/access/doc-files/JDBCProperties.html) to the underlying driver.** The `jdbc-options` field accepts any property supported by the IBM i JDBC driver — library list, SQL naming convention, date format, time format, and many more.

### Common Options

| Option | Purpose | Example |
|--------|---------|---------|
| `libraries` | Library list for unqualified name resolution | `[MYLIB, DEVDATA, QGPL]` |
| `naming` | SQL naming convention | `sql` or `system` |
| `date format` | Date literal format | `iso`, `usa`, `eur`, `jis`, `mdy`, `dmy`, `ymd`, `julian` |
| `time format` | Time literal format | `hms`, `usa`, `iso`, `eur`, `jis` |
| `date separator` | Date component separator | `/`, `-`, `.`, `,`, `b` |

<Tabs>
  <Tab title="Library List">
    ```yaml
    sources:
      ibmi-dev:
        host: ${DB2i_HOST}
        user: ${DB2i_USER}
        password: ${DB2i_PASS}
        jdbc-options:
          libraries:
            - MYLIB
            - DEVDATA
            - QGPL
    ```

    **Use for**: Resolving unqualified object names against a specific set of libraries.

    <Note>
    `libraries` also accepts a comma-separated string: `libraries: "MYLIB, DEVDATA, QGPL"` — the server splits and trims it into an array.
    </Note>
```

**File:** docs/sql-tools/sources.mdx (L280-296)
```text
  <Tab title="SQL Naming">
    ```yaml
    sources:
      ibmi-system:
        host: ${DB2i_HOST}
        user: ${DB2i_USER}
        password: ${DB2i_PASS}
        jdbc-options:
          naming: system   # LIBRARY/OBJECT syntax
    ```

    **Use for**: Legacy SQL using `LIBRARY/OBJECT` slash-notation instead of standard `LIBRARY.OBJECT` dot-notation.

    **Values:**
    - `sql` (default) — standard `SCHEMA.OBJECT`
    - `system` — IBM i legacy `LIBRARY/OBJECT`
  </Tab>
```

**File:** packages/server/src/ibmi-mcp-server/schemas/config.ts (L19-69)
```typescript
export const SqlToolParameterSchema = z
  .object({
    name: z
      .string()
      .min(1, "Parameter name cannot be empty")
      .describe("Parameter name used in SQL statement"),
    type: z
      .enum(["string", "boolean", "integer", "float", "array"])
      .describe("Parameter data type for validation"),
    description: z
      .string()
      .optional()
      .describe("Human-readable parameter description"),
    default: z
      .union([
        z.string(),
        z.number(),
        z.boolean(),
        z.array(z.unknown()),
        z.null(),
      ])
      .optional()
      .describe("Default value when parameter is not provided"),
    required: z
      .boolean()
      .optional()
      .describe("Whether parameter is required (overrides default)"),
    itemType: z
      .enum(["string", "boolean", "integer", "float"])
      .optional()
      .describe("Array item type (only for array parameters)"),
    min: z.number().optional().describe("Minimum value for numeric types"),
    max: z.number().optional().describe("Maximum value for numeric types"),
    minLength: z
      .number()
      .optional()
      .describe("Minimum length for string/array types"),
    maxLength: z
      .number()
      .optional()
      .describe("Maximum length for string/array types"),
    enum: z
      .array(z.union([z.string(), z.number(), z.boolean()]))
      .optional()
      .describe("Valid values (enum validation)"),
    pattern: z
      .string()
      .optional()
      .describe("Custom validation pattern (regex for strings)"),
  })
  .describe("SQL tool parameter definition with validation constraints");
```

**File:** packages/server/src/ibmi-mcp-server/schemas/common.ts (L27-98)
```typescript
export const ToolAnnotationsSchema = z
  .object({
    /**
     * An optional human-readable name for the tool, optimized for UI display.
     */
    title: z
      .string()
      .optional()
      .describe("Human-readable tool title for UI display"),
    /**
     * A hint indicating that the tool does not modify any state.
     */
    readOnlyHint: z
      .boolean()
      .optional()
      .describe("Indicates the tool performs read-only operations"),
    /**
     * A hint indicating that the tool may interact with external systems.
     */
    openWorldHint: z
      .boolean()
      .optional()
      .describe(
        "Indicates the tool interacts with external, unpredictable systems",
      ),
    /**
     * Indicates that the tool has no side effects when called multiple times.
     */
    idempotentHint: z
      .boolean()
      .optional()
      .describe(
        "Indicates the tool produces consistent results for identical inputs",
      ),
    /**
     * Indicates that the tool may produce destructive side effects.
     */
    destructiveHint: z
      .boolean()
      .optional()
      .describe(
        "Indicates the tool may perform destructive or irreversible operations",
      ),
    /**
     * Domain classification for filtering and organization.
     */
    domain: z
      .string()
      .optional()
      .describe("Domain classification for client-side filtering"),
    /**
     * Category classification for filtering and organization.
     */
    category: z
      .string()
      .optional()
      .describe("Category classification for client-side filtering"),
    /**
     * Toolsets that the tool belongs to.
     */
    toolsets: z
      .array(z.string())
      .optional()
      .describe("Toolset memberships for client-side grouping"),
    /**
     * Additional custom metadata.
     */
    customMetadata: MetadataSchema.optional().describe(
      "Additional custom metadata for the tool",
    ),
  })
  .catchall(z.unknown())
```
