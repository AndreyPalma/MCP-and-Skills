# Skills y MCPs para Desarrolladores de IA

---

## 1. ¿Qué es un Skill?

En el contexto de la IA para desarrolladores, un **Skill** (habilidad) representa un módulo de conocimiento o experiencia estructurado que se puede "enseñar" o proporcionar al modelo de IA. Piensa en ello como una guía de estilo, un conjunto de principios arquitectónicos o una biblioteca de mejores prácticas para una tarea específica.

El modelo de IA central puede acceder a diferentes módulos de Skills según sea necesario. Estos módulos contienen instrucciones y conocimiento especializado. Por ejemplo:

- **`rpg-best-practices`:** Define estándares de codificación RPG IV / ILE RPG, reglas importantes, estructuración de procedimientos, evitar código legacy con indicadores numéricos etc.
- **`rpg-documentation`:** Guía al modelo para generar comentarios de cabecera en programas RPG, documentar parámetros de `*ENTRY PLIST` y mantener trazabilidad de cambios en ambiente AS400.
- **`rpg-troubleshooting`:** Contiene patrones de diagnóstico de errores comunes en IBM i: mensajes de escape (`*ESCAPE`), volcados de trabajo (`DMPJOB`), análisis de `JOBLOG` y técnicas de depuración con `STRDBG`.

La clave es que estas habilidades son **conocimiento especializado que la IA carga bajo demanda**. No es un prompt gigante que siempre está presente, sino experiencia que el modelo invoca cuando la tarea lo requiere, lo que ayuda a no saturar el contexto de la conversación.

---

## 2. ¿Qué es un MCP? (Model Context Protocol)

Un **MCP** (Model Context Protocol) es el mecanismo que permite a un modelo de IA interactuar con el mundo real, actuando como sus "manos" o "conectores". De forma nativa, un modelo de IA es como un cerebro brillante en una sala aislada: puede procesar información, pero no puede realizar acciones externas ni acceder a datos en tiempo real. Los MCPs son los puentes que rompen ese aislamiento.

El modelo de IA central extiende conectores hacia herramientas y sistemas externos. Cada conector es un MCP que proporciona una funcionalidad específica o acceso a datos de terceros:

- **MCP para IBM i / DB2 for i:** Permite al modelo ejecutar consultas SQL directamente sobre tablas del sistema (`QSYS2`, `SYSTABLES`, `SYSCOLUMNS`), consultar datos de producción y analizar estructuras de ficheros físicos y lógicos.
- **MCP para Job Queue / Scheduler AS400:** Le da al modelo la capacidad de consultar el estado de trabajos en batch (`WRKJOBQ`), revisar la cola de salida (`WRKSPLF`) y disparar o monitorear procesos calendarizados en el sistema.
- **MCP para TOBi Project Builder:** Automatiza la generación de la estructura de carpetas y archivos de un proyecto TOBi al recibir únicamente el número de pase, eliminando el proceso manual de creación y reduciendo errores de configuración inicial.

A diferencia de los Skills, que son conocimiento especializado, los MCPs son **funcionalidad pura y acceso a datos dinámicos**. Permiten al desarrollador configurar a la IA para que "pueda hacer" cosas en el mundo real, no solo "saber cómo" hacerlas.

---

## 3. Infografía para Presentación (Canvas)

Genera una infografía visual con el siguiente concepto y estructura. El objetivo es crear **dos imágenes separadas** para una presentación: una para Skills y otra para MCPs.

### Estilo Visual General (identidad BAC)
- **Fondo:** Blanco puro o gris muy claro (estilo pizarra limpia).
- **Estética:** Garabatos en pizarra — trazos a mano imperfectos, flechas dibujadas con marcador y texto con apariencia de letra manuscrita.
- **Paleta de colores:**
  - Rojo BAC (`#C8102E`) → elementos de acción, títulos principales y bordes de énfasis.
  - Negro (`#1A1A1A`) → texto y líneas de trazo principales.
  - Gris claro (`#E5E5E5`) → fondos de módulos, sombras suaves y separadores.
  - Blanco (`#FFFFFF`) → fondo base y espacios negativos.
  - Rojo pastel suave (`#F5C6CB`) → rellenos de tarjetas o áreas de soporte sin saturar.
- **Logo:** Incluir el logo de BAC (`images/logo-bac.png`) en la esquina superior derecha de cada imagen, a tamaño reducido y sin modificar sus proporciones.
- **Iconos:** Líneas simples (estilo outline), sin rellenos sólidos, con grosor de trazo uniforme.

---

### Imagen 1 — Skills: El Conocimiento

**Concepto central:** Un cerebro esquemático (trazo negro) en el centro, rodeado de módulos tipo tarjeta (borde rojo, fondo gris claro) conectados por líneas dibujadas a mano.

**Elementos visuales:**
- Logo BAC en esquina superior derecha.
- Icono principal: Libro abierto con una bombilla encima (líneas negras, detalle en rojo).
- Título con tipografía manuscrita: `"Skills = Saber cómo hacer algo bien"`
- Tres tarjetas conectadas al cerebro con flechas a mano:
  - `"rpg-best-practices"` → estándares de codificación ILE RPG
  - `"rpg-documentation"` → documentación de programas AS400
  - `"rpg-troubleshooting"` → diagnóstico de errores y JOBLOG
- Nota al pie en letra pequeña manuscrita: `"Se activan bajo demanda · No saturan el contexto"`

---

### Imagen 2 — MCPs: Las Herramientas

**Concepto central:** Un cerebro esquemático (trazo negro) en el centro extendiendo "brazos" o conectores (líneas rojas) hacia herramientas y sistemas externos representados como bloques.

**Elementos visuales:**
- Logo BAC en esquina superior derecha.
- Icono principal: Llave inglesa o enchufe (líneas negras, detalle en rojo).
- Título con tipografía manuscrita: `"MCPs = Poder hacer algo en el mundo real"`
- Tres bloques conectados con flechas dibujadas a mano:
  - `"IBM i / DB2 for i"` → consultar tablas y estructuras del sistema
  - `"Job Queue / Scheduler"` → monitorear y disparar procesos batch
  - `"TOBi Project Builder"` → generar estructura de proyecto automáticamente al indicar número de pase
- Nota al pie en letra pequeña manuscrita: `"Funcionalidad pura · Datos dinámicos en tiempo real"`

---

### Diferencia Clave (opcional: tercera imagen de cierre)

Representa una balanza o diagrama comparativo de dos columnas con estilo pizarra:

| MCP (rojo / trazo activo)       | Skills (gris / trazo pasivo)              |
|---------------------------------|-------------------------------------------|
| Funcionalidad                   | Conocimiento                              |
| "Poder hacer"                   | "Saber cómo"                              |
| Consultar DB2 for i en vivo     | `rpg-best-practices` → código limpio      |
| Monitorear jobs batch AS400     | `rpg-documentation` → cabeceras y trazas  |
| Crear estructura TOBi por nro. de pase | `rpg-troubleshooting` → análisis JOBLOG |


- Incluir logo BAC en esquina superior derecha.
- Separador central: línea vertical dibujada a mano en negro.
