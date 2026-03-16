# Skills y MCPs para Desarrolladores de IA

---

## 1. ¿Qué es un Skill?

En el contexto de la IA para desarrolladores, un **Skill** (habilidad) representa un módulo de conocimiento o experiencia estructurado que se puede "enseñar" o proporcionar al modelo de IA. Piensa en ello como una guía de estilo, un conjunto de principios arquitectónicos o una biblioteca de mejores prácticas para una tarea específica.

El modelo de IA central puede acceder a diferentes módulos de Skills según sea necesario. Estos módulos contienen instrucciones y conocimiento especializado. Por ejemplo:

- **Skill de Diseño Frontend:** Incluye principios de composición, guías de estilo y reglas para evitar estéticas genéricas.
- **Skill de Arquitectura Limpia:** Proporciona directrices sobre cómo estructurar el código de forma modular y mantenible.
- **Skill de Pruebas Unitarias:** Contiene las mejores prácticas para escribir pruebas efectivas y con buena cobertura.

La clave es que estas habilidades son **conocimiento especializado que la IA carga bajo demanda**. No es un prompt gigante que siempre está presente, sino experiencia que el modelo invoca cuando la tarea lo requiere, lo que ayuda a no saturar el contexto de la conversación.

---

## 2. ¿Qué es un MCP? (Model Context Protocol)

Un **MCP** (Model Context Protocol) es el mecanismo que permite a un modelo de IA interactuar con el mundo real, actuando como sus "manos" o "conectores". De forma nativa, un modelo de IA es como un cerebro brillante en una sala aislada: puede procesar información, pero no puede realizar acciones externas ni acceder a datos en tiempo real. Los MCPs son los puentes que rompen ese aislamiento.

El modelo de IA central extiende conectores hacia herramientas y sistemas externos. Cada conector es un MCP que proporciona una funcionalidad específica o acceso a datos de terceros:

- **MCP para Chrome:** Permite al modelo navegar por la web en tiempo real, extraer información de páginas o interactuar con elementos de una interfaz.
- **MCP para APIs (como Ticket Taylor):** Le da al modelo la capacidad de consultar y manipular datos de servicios externos.
- **MCP para Bases de Datos (SQL / Google Search):** Proporciona acceso a repositorios de información estructurada o a motores de búsqueda para obtener datos externos.

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
  - `"Diseño Frontend"` → guías de composición y estilo
  - `"Arquitectura Limpia"` → estructura de código modular
  - `"Pruebas Unitarias"` → mejores prácticas de testing
- Nota al pie en letra pequeña manuscrita: `"Se activan bajo demanda · No saturan el contexto"`

---

### Imagen 2 — MCPs: Las Herramientas

**Concepto central:** Un cerebro esquemático (trazo negro) en el centro extendiendo "brazos" o conectores (líneas rojas) hacia herramientas y sistemas externos representados como bloques.

**Elementos visuales:**
- Logo BAC en esquina superior derecha.
- Icono principal: Llave inglesa o enchufe (líneas negras, detalle en rojo).
- Título con tipografía manuscrita: `"MCPs = Poder hacer algo en el mundo real"`
- Tres bloques conectados con flechas dibujadas a mano:
  - `"Chrome"` → navegar y extraer información de la web
  - `"API / Ticket Taylor"` → consultar datos de servicios externos
  - `"SQL / Google Search"` → acceder a bases de datos y buscadores
- Nota al pie en letra pequeña manuscrita: `"Funcionalidad pura · Datos dinámicos en tiempo real"`

---

### Diferencia Clave (opcional: tercera imagen de cierre)

Representa una balanza o diagrama comparativo de dos columnas con estilo pizarra:

| MCP (rojo / trazo activo)  | Skills (gris / trazo pasivo)         |
|----------------------------|--------------------------------------|
| Funcionalidad              | Conocimiento                         |
| "Poder hacer"              | "Saber cómo"                         |
| Herramientas               | Seguir guías de arquitectura         |                   
| Conectarse a terceros      | Aplicar estilos de código limpios    |
| Leer una base de datos     |                                      |                                                             


- Incluir logo BAC en esquina superior derecha.
- Separador central: línea vertical dibujada a mano en negro.
