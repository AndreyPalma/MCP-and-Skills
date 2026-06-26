```markdown
# Búsqueda de Documentación: Sin MCP vs Con MCP

## El problema: cómo busca el AI sin MCP

Cuando un AI (como Copilot, Cursor, Claude, etc.) necesita información de tu documentación
**sin un servidor MCP**, tiene que hacer esto:

```
1. Explorar carpetas para saber qué archivos existen
2. Abrir el archivo que parece relevante → lee TODO el archivo completo
3. Si no encontró lo que buscaba, abrir otro archivo → lee TODO ese también
4. Repetir hasta encontrar la respuesta
```

**El resultado:**

- El AI lee archivos enteros aunque solo necesite 2 párrafos
- Un archivo de documentación típico puede tener 10,000 a 50,000 caracteres
- Si necesita revisar 3 o 4 archivos, puede consumir 100,000+ caracteres de contexto
- Más contexto = más tokens = más costo y respuestas más lentas
- El AI puede "perderse" información importante enterrada en un archivo largo

---

## La solución: búsqueda con MCP (`search_docs`)

El MCP server indexa toda tu documentación en una base de datos con búsqueda inteligente.
En lugar de leer archivos completos, el AI hace una búsqueda y recibe **solo los fragmentos
más relevantes**.

```
AI → search_docs(library="mi-docs", query="reglas de negocio tabla tarjetas")
     → recibe 5 fragmentos relevantes (~7,500 caracteres en total)
```

**El resultado:**

- El AI recibe solo lo que necesita, no el archivo completo
- No necesita explorar carpetas ni saber qué archivos existen
- Los resultados ya vienen ordenados por relevancia
- Costo de tokens: 2,000–15,000 caracteres para el mismo tema

### Comparación directa

| Aspecto | Sin MCP | Con MCP |
|---------|---------|---------|
| Caracteres consumidos | 30,000–200,000 | 2,000–15,000 |
| Necesita explorar carpetas | Sí | No |
| Resultados ordenados por relevancia | No | Sí |
| Encuentra reglas específicas en docs largos | Difícil | Directo |

---

## Cómo funciona la búsqueda internamente

### Paso 1: Indexación (se hace una sola vez)

Cuando indexas documentación, el servidor divide cada documento en **fragmentos (chunks)**
de aproximadamente 1,500 caracteres. Cada fragmento se guarda en la base de datos junto
con información sobre de dónde viene (URL, título, sección del documento).

```
Documento original (10,000 chars)
    ↓ se divide en
Chunk 1: "Tabla tarjetas - descripción general..." (1,500 chars)
Chunk 2: "Campos obligatorios: id, estado, fecha..." (1,500 chars)
Chunk 3: "Reglas de negocio: el campo estado solo..." (1,500 chars)
...y así sucesivamente
```

### Paso 2: Búsqueda (cada vez que el AI consulta)

Cuando el AI hace una búsqueda, el sistema usa **dos mecanismos** para encontrar
los fragmentos más relevantes:

#### Mecanismo 1: Búsqueda por texto (FTS - Full Text Search)

Busca los términos de la query en el contenido de los fragmentos. Funciona similar
a un buscador: encuentra los documentos que contienen las palabras buscadas.

Para decidir cuál resultado es más relevante, usa un algoritmo llamado **BM25**.
Este algoritmo considera:

- Si el término buscado aparece muchas veces en el fragmento → más relevante
- Si el término es raro en toda la documentación → más relevante (es más específico)
- Si el término aparece en el título o la URL → también suma puntos

**¿Qué significa que la URL tenga más "peso" que el título?**

Imagina que buscas "tarjetas". El sistema revisa cada fragmento y le da una puntuación:

```
Fragmento A:
  URL:     /ventas/tarjetas/schema.md   ← contiene "tarjetas" en la URL
  Título:  "Schema de la base de datos"
  Contenido: "La tabla principal almacena..."

Fragmento B:
  URL:     /docs/general/overview.md
  Título:  "Documentación general"
  Contenido: "Las tarjetas son el objeto central del sistema..."
```

El Fragmento A sube más en el ranking porque "tarjetas" aparece en su URL.
El Fragmento B también sube, pero menos, porque "tarjetas" solo aparece en el contenido.

Esto significa que **los nombres de carpetas y archivos importan para la búsqueda**.
Una carpeta llamada `/tarjetas/tablas/` ayuda más al ranking que `/docs/data/`.

#### Mecanismo 2: Búsqueda semántica (Vector Search, opcional)

Si está configurado con una API key de OpenAI, el sistema también convierte cada
fragmento en un vector matemático que representa su "significado". Esto permite
encontrar resultados relevantes aunque no usen exactamente las mismas palabras.

```
Query: "restricciones al cerrar una tarjeta"
  → encuentra fragmentos sobre "validaciones al cambiar estado a CERRADO"
     aunque no contengan la palabra "restricciones"
```

### Paso 3: Expansión de contexto

Cuando encuentra un fragmento relevante, el sistema automáticamente incluye
fragmentos vecinos (el anterior, el siguiente, el padre en la jerarquía de headings)
para que el AI tenga contexto suficiente sin recibir el documento completo.

---

## Cuándo usar `search_docs` y cuándo no

### Ideal para `search_docs`

- Buscar información específica: "¿qué campos tiene la tabla X?"
- Encontrar reglas de negocio: "¿cuándo no se puede usar el campo Y?"
- Investigación iterativa: ir construyendo una solución pregunta por pregunta
- Tablas viejas con particularidades: las reglas documentadas aparecen directas

**Ejemplo de flujo iterativo para diagramar:**
```
1. search_docs("tarjetas schema relaciones")
2. search_docs("movimientos relacion tarjetas")
3. search_docs("tabla estados restricciones legacy")
→ el AI construye el diagrama con la info acumulada
→ si falta algo: search_docs("campo fecha_cierre comportamiento especial")
```

### Limitación de `search_docs`

`search_docs` devuelve los **N resultados más relevantes** (por defecto 5).
No existe un modo "dame todos los documentos de una vez".

Si necesitas una visión completa de TODA la documentación de un módulo en una sola
operación (por ejemplo, un mapa de relaciones de 50 tablas de una vez), tendrías
que hacer múltiples búsquedas iterativas o usar `fetch_url` para obtener archivos
específicos cuya URL ya conoces.

---

## Consejos para que la búsqueda funcione mejor

### Usa nombres descriptivos en carpetas y archivos

Los nombres de carpetas forman parte de la URL que se indexa y ayudan al ranking:

```
✅ /ventas/tablas/schema-tarjetas.md
✅ /ventas/reglas-negocio/restricciones-estado.md
❌ /docs/data/file1.md
```

### Usa headings claros en tus documentos

El sistema divide los documentos por secciones (headings). Headings descriptivos
producen chunks más coherentes:

```markdown
## Tabla: tarjetas
### Campo: estado
#### Valores permitidos y cuándo usarlos
```

### Documenta las reglas de negocio en texto plano

Las reglas específicas de tablas viejas deben estar escritas explícitamente,
no solo en comentarios de código. El FTS busca en el texto del documento:

```markdown
> ⚠️ Restricción: El campo `fecha_cierre` solo puede setearse cuando
> `estado = 'CERRADO'`. En versiones anteriores a 2019 no genera error
> si se intenta en otro estado — simplemente se ignora silenciosamente.
```

Si está documentado así, una búsqueda como `"fecha_cierre restricción"` o
`"campo fecha_cierre comportamiento"` lo encuentra directamente.
```

Aquí continúa el documento con las secciones finales:

```markdown
---

## Cómo formular buenas queries para `search_docs`

La calidad de los resultados depende de cómo se formula la búsqueda.
El FTS busca coincidencias de palabras, así que queries más específicas
dan mejores resultados.

### Queries efectivas

```
✅ "tabla tarjetas campos obligatorios"
✅ "estado tarjeta valores permitidos restricciones"
✅ "fecha_cierre comportamiento legacy"
✅ "movimientos relacion tarjetas foreign key"
✅ "tabla X cuando no usar escenarios especiales"
```

### Queries poco efectivas

```
❌ "información sobre tarjetas"     → demasiado genérico
❌ "cómo funciona"                  → sin contexto específico
❌ "datos"                          → palabra muy común, poco discriminante
```

### Tip: combinar el nombre de la entidad con el aspecto que buscas

```
[nombre de tabla/campo] + [aspecto: reglas, restricciones, relaciones, valores, uso]

Ejemplos:
  "tarjetas estado reglas negocio"
  "movimientos tipos permitidos"
  "clientes campos deprecated"
  "tabla pagos restricciones legacy"
```

---

## Resumen: qué cambia con el MCP

```
Sin MCP:
  AI → lee archivos completos → procesa todo → encuentra la respuesta
  Costo: alto en tokens, lento, puede perder info en archivos largos

Con MCP:
  AI → busca por términos → recibe solo fragmentos relevantes → responde
  Costo: bajo en tokens, rápido, encuentra reglas específicas directamente
```

La diferencia más importante no es solo el costo en tokens — es que el AI
puede encontrar **reglas de negocio específicas y particularidades de tablas viejas**
que estarían enterradas en archivos largos y que sin MCP probablemente pasaría por alto.

Si la documentación está bien escrita (headings claros, reglas explícitas en texto,
nombres descriptivos en carpetas), el MCP convierte esa documentación en una fuente
de conocimiento que el AI puede consultar de forma precisa y eficiente.
```
