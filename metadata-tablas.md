---
title: Tabla FACTURC — Cabecera de facturas
description: Campos, relaciones y reglas RN-070 a RN-073 de FACTURC
tags: [tabla, facturacion, FACTURC, diccionario-datos]
version: "1.0"
---

# Tabla FACTURC — Cabecera de facturas

## Propósito de la tabla FACTURC

La tabla FACTURC almacena la cabecera de cada factura del módulo de facturación. La escriben los
programas FAC001R (emisión) y FAC009R (anulación); la leen los reportes contables. Reside en la
biblioteca PRODDAT y crece ~50.000 registros al mes.

## Claves de la tabla FACTURC

La clave primaria es `IDFACTURA` (numérico secuencial). Existe un índice por
`FACTURC.IDCLIENTE` + `FACTURC.FECHAEMI` para las consultas del portal por cliente y fecha.

## Campos de la tabla FACTURC

| Campo | Tipo | Nulo | Default | Descripción de negocio | Valores / Dominio |
| --- | --- | --- | --- | --- | --- |
| IDFACTURA | DECIMAL(10) | No | — | Identificador único de la factura | secuencial |
| IDCLIENTE | CHAR(10) | No | — | Cliente al que se emite | FK a CLIENTC |
| ESTADO | CHAR(1) | No | 'P' | Estado de la factura | 'P'=Pendiente, 'L'=Liquidada, 'A'=Anulada |
| TOTAL | DECIMAL(11,2) | No | 0 | Monto total con impuestos | > 0 |

Si la tabla tiene más de ~25 campos, partir en grupos con un `### Campos de identificación de
FACTURC`, `### Campos de montos de FACTURC`, cada uno con su propia tabla.

## Campos con particularidades en FACTURC

### Campo FACTURC.ESTADO — el valor 'A' no se borra físicamente

Cuando una factura se anula, `ESTADO` pasa a 'A' pero el registro permanece para auditoría.
Ningún proceso debe filtrar asumiendo que 'A' no existe. Este tipo de conocimiento (que no está
en el catálogo del sistema) es lo más valioso de documentar.

## Relaciones de FACTURC con otras tablas

### FACTURC → CLIENTC (N:1)

`FACTURC.IDCLIENTE` referencia a `CLIENTC.IDCLIENTE`. Es FK física con ON DELETE RESTRICT: no se
puede borrar un cliente con facturas.

### FACTURC → ORDENC (N:1, relación IMPLÍCITA, sin FK)

`FACTURC.IDORDEN` corresponde a `ORDENC.IDORDEN`, pero **no existe FK declarada**: la integridad
se mantiene solo por convención del programa FAC001R. Consecuencias prácticas:
- Pueden existir facturas con `IDORDEN` que ya no está en ORDENC (registros huérfanos).
- El join es seguro para leer, pero NO asumir que toda factura tiene orden: verificar con
  `LEFT JOIN` y contemplar nulos.
- Al insertar de prueba, el motor NO valida la existencia: la validación es responsabilidad del
  código, no de la base.

## Advertencias y deuda técnica de FACTURC

Sección para tablas viejas o críticas con malas prácticas. Documentar aquí lo que rompe las
suposiciones normales, porque es lo que causa bugs y lo que un agente necesita saber antes de
tocar la tabla:

- **Sin integridad referencial declarada**: las relaciones con ORDENC y PRODUCTC son solo
  convención de programa. Esperar datos huérfanos; nunca asumir que un join encuentra match.
- **Clave primaria no confiable históricamente**: registros anteriores a 2018 pueden tener
  `IDFACTURA` duplicado por una carga masiva mal hecha. Filtrar por `FECHAEMI >= '2018-01-01'`
  si la unicidad importa.
- **Campo reutilizado**: `OBSERV` se usó para guardar el número de nota de crédito antes de que
  existiera `IDNOTACRED`. En registros viejos el dato vive en `OBSERV`, no en `IDNOTACRED`.
- **Tipos sobredimensionados**: `IDCLIENTE` es CHAR(10) pero solo se usan 6 posiciones; los 4
  restantes están en blanco. Comparar con TRIM para evitar fallos de match.
- **Sin borrado físico**: nada se borra; los registros muertos se marcan con `ESTADO='A'`. Toda
  query de negocio debe excluir 'A' explícitamente.

## Reglas de la tabla FACTURC

### RN-073 — ESTADO solo admite 'P', 'L' o 'A'

El campo `FACTURC.ESTADO` solo acepta los valores 'P', 'L' o 'A'. Se valida con el constraint
`CHK_FACTURC_ESTADO`. Cualquier otro valor es rechazado en la inserción.

## Metadata estructurada de FACTURC (opcional)

Incluir este bloque SOLO si un agente va a parsear la metadata para generar DDL, INSERTs o
validadores. Es complemento de la prosa, no sustituto. Mantenerlo corto y consistente con las
tablas de arriba.

```json
{
  "tabla": "FACTURC",
  "esquema": "PRODDAT",
  "pk": ["IDFACTURA"],
  "pk_confiable": false,
  "campos": [
    { "nombre": "ESTADO", "tipo": "CHAR(1)", "nulo": false, "dominio": {"P":"Pendiente","L":"Liquidada","A":"Anulada"} }
  ],
  "relaciones": [
    { "destino": "CLIENTC", "tipo": "N:1", "join": "FACTURC.IDCLIENTE = CLIENTC.IDCLIENTE", "fk_declarada": true, "on_delete": "RESTRICT" },
    { "destino": "ORDENC", "tipo": "N:1", "join": "FACTURC.IDORDEN = ORDENC.IDORDEN", "fk_declarada": false, "integridad": "convencion_programa", "nota": "Esperar huérfanos; usar LEFT JOIN" }
  ],
  "reglas": [
    { "id": "RN-073", "campos": ["ESTADO"], "regla": "ESTADO IN ('P','L','A')" }
  ],
  "advertencias": [
    "Sin integridad referencial declarada con ORDENC/PRODUCTC: convención de programa",
    "IDFACTURA puede estar duplicado antes de 2018",
    "OBSERV reutilizado como nota de crédito en registros viejos",
    "IDCLIENTE relleno con blancos: comparar con TRIM",
    "Sin borrado físico: excluir ESTADO='A' en queries de negocio"
  ]
}
```

<!--
GUÍA DE LLENADO (borrar al terminar):
- Un archivo por tabla. El nombre de la tabla aparece en TODOS los headings.
- En prosa, usar notación TABLA.CAMPO; valores codificados literales entre comillas ('P').
- No documentes solo lo que el catálogo (DSPFFD / information_schema) ya sabe: el valor está
  en la descripción de negocio, el dominio, las particularidades y las reglas.
- TABLAS VIEJAS / CRÍTICAS CON MALAS PRÁCTICAS:
  * Documentar las relaciones que existen DE HECHO aunque no haya FK declarada, marcándolas
    como implícitas (fk_declarada: false). Es lo que el catálogo no te dice.
  * Usar la sección "Advertencias y deuda técnica": huérfanos, claves no confiables, campos
    reutilizados, tipos sobredimensionados, sin borrado físico, encoding raro. Esto evita bugs.
  * Si una relación no es fiable, dilo explícitamente y recomienda LEFT JOIN + manejo de nulos.
- El bloque JSON es opcional y solo si vas a generar código a partir de la metadata.
-->
