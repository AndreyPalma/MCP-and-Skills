Prompt

Analiza el video de Midudev: https://youtu.be/2aN_-m1uU4k?si=k8AwS2L_QCLq9tyV

Y crea una infografía limpia, estilo boceto a mano, que explique los temas de MCP y Skills como el orador lo dice y ejemplifica para su uso con la IA como desarrollador. Usa colores pastel suaves como verde menta, azul claro y lavanda. La estética debe ser amigable, informal y parecerse a un garabato en una pizarra. Incluye iconos sencillos de líneas y texto escrito a mano.


Infografía:
​🎨 Concepto Visual de la Infografía
​Fondo: Color crema suave (estilo papel de libreta).
​Estilo: Trazos de lápiz imperfectos, flechas dibujadas a mano y garabatos decorativos.
​Colores: * Verde menta para acciones y herramientas (MCP).
​Azul claro para teoría y guías (Skills).
​Lavanda para el "cerebro" o núcleo (LLM).
​
​📝 Estructura de la Infografía
​1. El Núcleo: El LLM (El Cerebro) 🧠
​Color: Lavanda.
​Descripción: El modelo (GPT, Claude, Gemini) es reactivo. Solo predice el siguiente token.
​Icono: Un cerebro simple con líneas de conexión.
​Dato Clave: "Si la basura entra, la basura sale" (Importancia del contexto). [10:04]
​2. MCP: Model Context Protocol (Las Manos) 🛠️
​Color: Verde menta.
​¿Qué es?: Herramientas externas que conectan la IA con el mundo real.
​Ejemplos del video:
​Controlar Chrome: Para ver una web en tiempo real. [01:12:25]
​Ticket Taylor: Para consultar ventas de entradas de un evento mediante una API. [01:14:15]
​Google Search / SQL: Conexión a bases de datos o buscadores.
​Icono: Una llave inglesa o una mano alcanzando un enchufe.
​Resumen: Le da funcionalidad y acceso a datos de terceros que el modelo no conoce.
​3. Agent Skills (El Conocimiento/Experiencia) 💡
​Color: Azul claro.
​¿Qué es?: Módulos de instrucciones y "know-how" que la IA carga solo cuando los necesita.
​Ejemplo del video:
​Frontend Design Skill: Una guía que le dice a la IA: "No uses estéticas genéricas, sigue estos principios de composición". [01:17:36]
​Diferencia clave: A diferencia de un prompt gigante, las Skills se activan bajo demanda para no saturar el contexto. [01:23:16]
​Icono: Un libro abierto con una bombilla encima.
​Resumen: Es el conocimiento especializado sobre cómo hacer algo bien.
​4. La gran diferencia: Funcionalidad vs. Conocimiento ⚖️
​Dibujo Central: Una balanza o un diagrama comparativo.
​MCP (Verde): "Poder hacer" (Conectarse a Slack, leer mi DB).
​Skills (Azul): "Saber cómo" (Seguir guías de arquitectura, estilos de código limpios).
​🚀 Uso práctico para el Desarrollador
​En herramientas como Claude Code o Visual Studio Code (Copilot), Midudev explica que:
​Planeas: Usas un modelo caro (como Claude 3.5 Sonnet) para diseñar el plan. [54:13]
​Ejecutas: Usas MCPs para que la IA edite archivos o ejecute comandos de terminal (ej. npm install). [01:05:23]
​Refinas: Instalas Skills desde repositorios (como skills.sh) para que el código resultante siga los estándares de tu equipo. [01:16:45]


Skills y MCP


Explicación Individual de 'Skills'
​En el contexto de la IA para desarrolladores, una 'Skill' (habilidad) representa un módulo de conocimiento o experiencia estructurado que se puede "enseñar" o proporcionar al modelo de IA. Piensa en ello como una guía de estilo, un conjunto de principios arquitectónicos o una biblioteca de mejores prácticas para una tarea específica.
​Como se muestra en la imagen anterior, el modelo de IA central (lavanda) puede acceder a diferentes módulos de 'Skills' (azul claro) según sea necesario. Estos módulos contienen instrucciones y conocimiento especializado. Por ejemplo:
​Skill de Diseño Frontend: Incluye principios de composición, guías de estilo y reglas para evitar estéticas genéricas, como el ejemplo que vimos de "Frontend Design Skill".
​Skill de Arquitectura Limpia: Proporciona directrices sobre cómo estructurar el código de forma modular y mantenible.
​Skill de Pruebas Unitarias: Contiene las mejores prácticas para escribir pruebas efectivas y con buena cobertura.
​La clave es que estas habilidades son conocimiento especializado que la IA "carga bajo demanda". No es un prompt gigante que siempre está presente, sino experiencia que el modelo invoca cuando la tarea lo requiere, lo que ayuda a no saturar el contexto de la conversación.
​Explicación Individual de 'MCPs' (Model Context Protocol)
​Un 'MCP' (Model Context Protocol) es el mecanismo que permite a un modelo de IA interactuar con el mundo real, actuando como sus "manos" o "conectores". De forma nativa, un modelo de IA es como un cerebro brillante en una sala aislada: puede procesar información, pero no puede realizar acciones externas ni acceder a datos en tiempo real. Los MCPs son los puentes que rompen ese aislamiento.
​En la imagen anterior, puedes ver cómo el modelo de IA central (lavanda) extiende conectores o "brazos" (verde menta) hacia herramientas y sistemas externos. Cada conector es un MCP que proporciona una funcionalidad específica o acceso a datos de terceros:
​MCP para Chrome: Permite al modelo navegar por la web en tiempo real, extraer información de páginas o interactuar con elementos de una interfaz.
​MCP para APIs (como Ticket Taylor): Le da al modelo la capacidad de consultar y manipular datos de servicios externos, como las ventas de eventos en el ejemplo que vimos.
​MCP para Bases de Datos (como SQL/Google Search): Proporciona acceso a repositorios de información estructurada o a motores de búsqueda para obtener datos externos.
​A diferencia de las 'Skills', que son conocimiento especializado, los MCPs son funcionalidad pura y acceso a datos dinámicos. Permiten al desarrollador configurar a la IA para que "pueda hacer" cosas en el mundo real, no solo "saber cómo" hacerlas.
