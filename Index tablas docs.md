Aquí está la plantilla directamente en markdown, con todos los bloques de código correctamente cerrados:

---

```
---
title: "Tabla NOMBRE_TABLA — [qué almacena], módulo [MODULO], departamento [DEPARTAMENTO], consultada por [procesos: facturación, despacho, reportes, etc.]"
---
```

# Tabla NOMBRE_TABLA — [descripción corta]

La tabla NOMBRE_TABLA pertenece al módulo de **MODULO** del departamento de
**DEPARTAMENTO**. Almacena [qué almacena]. Cada fila representa [qué representa
una fila]. Es consultada por [procesos o sistemas que la usan]. Las tablas
relacionadas principales son TABLA_A (por COL_FK_A) y TABLA_B (por COL_FK_B).

## Estructura de columnas

| Columna         | Tipo          | Nulo | Descripción                              |
|-----------------|---------------|------|------------------------------------------|
| COL_PK          | DECIMAL(10)   | No   | Identificador único                      |
| COL_FK_A        | DECIMAL(8)    | No   | FK a TABLA_A (sin constraint en DB)      |
| COL_ESTADO      | CHAR(1)       | No   | Estado del registro; valores controlados |
| COL_TIPO        | CHAR(2)       | Sí   | Tipo; valores por catálogo externo       |
| COL_FECHA       | DATE          | No   | Fecha de creación                        |
| COL_MONTO       | DECIMAL(15,2) | Sí   | Monto calculado; nulo si abierto         |
| COL_CAMPO_LIBRE | VARCHAR(50)   | Sí   | Aparenta libre; ver sección Campo        |

## Columnas clave y su significado de negocio

- **COL_PK**: identificador único; nunca se reutiliza ni se recicla
- **COL_ESTADO**: controla el ciclo de vida; valores válidos documentados en sección Regla
- **COL_TIPO**: parece campo libre pero sus valores vienen de TABLA_CATALOGO; ver sección Campo
- **COL_MONTO**: permanece nulo mientras COL_ESTADO = 'A'; no confiar en él para registros abiertos
- **COL_CAMPO_LIBRE**: nombre engañoso; en la práctica solo acepta valores de un catálogo no documentado en la DB

## Campo: COL_TIPO — valores por catálogo sin integridad referencial

En la tabla NOMBRE_TABLA, el campo COL_TIPO aparenta ser un campo de texto libre
(VARCHAR sin CHECK constraint y sin FK declarada), pero en la práctica sus valores
provienen exclusivamente de la tabla TABLA_CATALOGO, columna COD_TIPO. No existe
ninguna restricción en la base de datos que lo imponga; la integridad la mantiene
únicamente la aplicación.

Los valores válidos conocidos son:

| Valor | Significado                | Tabla origen            |
|-------|----------------------------|-------------------------|
| 'AA'  | [descripción del valor AA] | TABLA_CATALOGO.COD_TIPO |
| 'BB'  | [descripción del valor BB] | TABLA_CATALOGO.COD_TIPO |
| 'CC'  | [descripción del valor CC] | TABLA_CATALOGO.COD_TIPO |

> **Advertencia**: insertar un valor en COL_TIPO que no exista en TABLA_CATALOGO
> no genera error en la base de datos, pero rompe los reportes que hacen JOIN con
> TABLA_CATALOGO para obtener la descripción. Siempre validar contra TABLA_CATALOGO
> antes de insertar.

Para obtener la descripción legible de COL_TIPO hay que unir con TABLA_CATALOGO:

```sql
SELECT
    t.COL_PK,
    t.COL_TIPO,
    c.DESCRIPCION AS TIPO_DESCRIPCION
FROM SCHEMA.NOMBRE_TABLA t
LEFT JOIN SCHEMA.TABLA_CATALOGO c ON c.COD_TIPO = t.COL_TIPO
WHERE t.COL_ESTADO != 'C';
```

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Campo: COL_CAMPO_LIBRE — valores controlados sin catálogo formal

En la tabla NOMBRE_TABLA, el campo COL_CAMPO_LIBRE es VARCHAR sin ninguna
restricción en la base de datos. No existe tabla de catálogo asociada. Sin
embargo, el proceso [NOMBRE_PROCESO] solo escribe valores específicos en este
campo. Los valores observados en producción y su significado son:

- `'VALOR_1'`: [qué significa y cuándo se asigna]
- `'VALOR_2'`: [qué significa y cuándo se asigna]
- `NULL`: [qué significa que sea nulo — ¿registro sin procesar? ¿no aplica?]

> **Advertencia**: cualquier valor distinto a los listados indica un registro
> generado fuera del proceso normal o un error de carga. Filtrar por valores
> no documentados produce resultados incorrectos silenciosamente.

Estado: candidata · Procedencia: [patrón observado en datos de producción] · Confianza: media

## Antipatrón: [nombre descriptivo del antipatrón]

En la tabla NOMBRE_TABLA, [descripción del antipatrón: qué columna o estructura
está mal diseñada, por qué existe (deuda técnica, migración histórica, decisión
de diseño antigua), y cuál es el impacto práctico al consultar la tabla].

Ejemplo: la columna COL_X almacena múltiples valores separados por coma
('VAL1,VAL2,VAL3') en lugar de una tabla de detalle. Para filtrar por un valor
específico hay que usar LIKE, lo que impide usar índices y degrada el rendimiento.

```sql
-- MAL: no usar en producción con tablas grandes
SELECT * FROM SCHEMA.NOMBRE_TABLA
WHERE COL_X LIKE '%VAL1%';
```

```sql
-- CORRECTO: patrón recomendado para este antipatrón
SELECT * FROM SCHEMA.NOMBRE_TABLA
WHERE ',' || COL_X || ',' LIKE '%,VAL1,%';
```

> **Advertencia**: no normalizar esta columna fue una decisión histórica de
> [año/contexto]. No intentar refactorizarla sin coordinar con [equipo responsable]
> porque [N] procesos dependen del formato actual.

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Antipatrón: columna con significado cambiante según COL_ESTADO

En la tabla NOMBRE_TABLA, la columna COL_Y tiene un significado distinto
dependiendo del valor de COL_ESTADO. Cuando COL_ESTADO = 'A', COL_Y almacena
[significado A]. Cuando COL_ESTADO = 'F', COL_Y almacena [significado F].
Este patrón es polimorfismo de columna y hace que las consultas genéricas sobre
COL_Y sean incorrectas si no filtran primero por COL_ESTADO.

> **Advertencia**: nunca agregar o promediar COL_Y sin filtrar por COL_ESTADO
> primero. Los valores tienen unidades o semánticas distintas según el estado.

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Regla: [nombre corto y descriptivo de la regla]

En la tabla NOMBRE_TABLA, [descripción completa en prosa. Explica qué campo está
involucrado, qué valor o condición aplica, y cuál es el impacto si no se respeta.
Menciona nombres de columnas y valores concretos. Este chunk debe ser autónomo:
quien lo lea sin el resto del documento debe entender la regla completa].

> **Advertencia**: [consecuencia crítica si se ignora].

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Regla: valores válidos del campo COL_ESTADO

En la tabla NOMBRE_TABLA, el campo COL_ESTADO solo admite en la práctica los
valores 'A' (abierto), 'F' (facturado), 'C' (cancelado) y 'D' (despachado),
aunque la base de datos no impone esta restricción mediante un CHECK constraint.
Cualquier otro valor en COL_ESTADO indica un registro corrupto o generado por
un proceso defectuoso. Siempre filtrar por valores conocidos en consultas de
análisis para evitar incluir registros inválidos.

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Regla: [validación que la DB no impone]

En la tabla NOMBRE_TABLA, la columna COL_Z debería ser siempre mayor que cero,
pero la base de datos no tiene un CHECK constraint que lo garantice. El proceso
[NOMBRE_PROCESO] valida esto en la capa de aplicación. Registros con COL_Z <= 0
son errores de carga y deben excluirse de cualquier análisis. Se estima que
[N%] de los registros históricos tienen este problema por una migración de [año].

Estado: confirmada · Procedencia: [fuente] · Confianza: alta

## Flujo: [nombre del flujo o ciclo de vida]

En la tabla NOMBRE_TABLA, [descripción del ciclo de vida. Explica los estados
posibles, las transiciones entre ellos, qué columnas cambian en cada transición,
y qué otras tablas participan en el flujo. Por ejemplo: el registro nace en
ESTADO 'A', pasa a 'D' cuando logística confirma, y a 'F' cuando se factura,
momento en que se calcula COL_MONTO. Solo puede cancelarse ('C') desde 'A' o 'D'].

Estado: confirmada · Procedencia: [fuente]

## Consulta: [nombre descriptivo — qué resuelve esta consulta]

Para [objetivo] en la tabla NOMBRE_TABLA, usar el siguiente patrón. Excluir los
registros con COL_TIPO = 'XX' (registros internos) y COL_ESTADO = 'C' (cancelados).
No incluir COL_MONTO en registros con COL_ESTADO = 'A' porque estará nulo por diseño.

```sql
SELECT
    t.COL_PK,
    t.COL_ESTADO,
    t.COL_TIPO,
    t.COL_MONTO
FROM SCHEMA.NOMBRE_TABLA t
WHERE t.COL_TIPO != 'XX'
  AND t.COL_ESTADO NOT IN ('C')
  AND t.COL_FECHA >= :fecha_inicio
ORDER BY t.COL_FECHA DESC;
```

## Consulta: verificar valores no documentados en COL_CAMPO_LIBRE

Para auditar si existen valores inesperados en COL_CAMPO_LIBRE de la tabla
NOMBRE_TABLA que no estén en la lista de valores conocidos, usar esta consulta
de diagnóstico. Valores distintos a los documentados indican registros generados
fuera del proceso normal.

```sql
SELECT
    COL_CAMPO_LIBRE,
    COUNT(*) AS CANTIDAD
FROM SCHEMA.NOMBRE_TABLA
WHERE COL_CAMPO_LIBRE NOT IN ('VALOR_1', 'VALOR_2')
   OR COL_CAMPO_LIBRE IS NULL
GROUP BY COL_CAMPO_LIBRE
ORDER BY CANTIDAD DESC;
```

## Relación: NOMBRE_TABLA con TABLA_RELACIONADA

La tabla NOMBRE_TABLA se une con TABLA_RELACIONADA por la columna COL_FK.
[Explica cuándo y por qué se hace esta unión, qué información aporta cada tabla,
y si hay condiciones especiales: nulos, cardinalidad, filtros obligatorios.
Nota: la FK no está declarada en la base de datos; la integridad la mantiene
la aplicación].

```sql
SELECT
    t.COL_PK,
    t.COL_ESTADO,
    r.COL_NOMBRE
FROM SCHEMA.NOMBRE_TABLA t
JOIN SCHEMA.TABLA_RELACIONADA r ON r.COL_PK = t.COL_FK
WHERE t.COL_ESTADO != 'C'
  AND t.COL_TIPO != 'XX';
```

## Glosario: [término de negocio]

[Término] (en la tabla NOMBRE_TABLA, campo COL_X = 'Y'): [definición completa
en prosa. Explica qué significa este valor en el contexto del negocio, por qué
existe, y cómo distinguirlo de otros valores similares].

## Glosario: [otro término]

[Término] (en la tabla NOMBRE_TABLA): [definición completa en prosa].

---

Esta vez la plantilla está renderizada directamente (no dentro de un bloque de código exterior) para que puedas verificar que todos los bloques SQL abren y cierran correctamente, y luego copiar el raw text.
