# Taller: Crear skills usando `skill-creator`

> Objetivo: aprender a usar `skill-creator` para que **él** construya tus skills. Tu trabajo no es escribir el `SKILL.md` a mano, sino darle instrucciones claras y detalladas de lo que quieres, dejar que lo genere, probarlo e iterar.

---

## La idea principal

`skill-creator` es un asistente que crea skills por ti. Tú no programas la skill: tú **describes muy bien qué quieres que haga**, y él se encarga de escribirla, probarla y mejorarla.

Por eso, la habilidad más importante en este taller no es técnica. Es **saber explicar con claridad** lo que necesitas. Mientras mejores y más detalladas sean tus instrucciones, mejor saldrá la skill desde el primer intento.

---

## ¿Qué es una skill?

Una skill es un conjunto de instrucciones reutilizables que el asistente de VS Code (GitHub Copilot) consulta cuando aparece cierta tarea. Una vez creada, el asistente la usa de forma automática cada vez que detecta que aplica, sin que tengas que explicarle todo de nuevo.

`skill-creator` se encarga de darle a esa skill la estructura correcta. Tú solo necesitas tener claro **qué hace** y **cuándo debe usarse**.

---

## El flujo completo

Crear una skill con `skill-creator` sigue siempre el mismo camino:

1. **Tú explicas** qué quieres, con el mayor detalle posible.
2. **`skill-creator` te hace preguntas** para llenar los huecos.
3. **`skill-creator` escribe** la skill.
4. **`skill-creator` prueba** la skill: genera los casos, los ejecuta y arma un reporte.
5. **Tú revisas** el reporte y das feedback.
6. **Se mejora y se repite** hasta que quede bien.

No es de una sola pasada. Es un ida y vuelta donde tú diriges y `skill-creator` ejecuta.

---

## Parte 1 — Cómo dar buenas instrucciones (lo más importante)

Cuando empieces, no digas solo "créame una skill para documentar código". Eso es demasiado vago y `skill-creator` tendrá que adivinar. En su lugar, dale una instrucción **clara y detallada**. Una buena instrucción responde a estas cuatro cosas:

1. **Qué debe hacer la skill.** La tarea concreta, paso a paso si aplica.
2. **Cuándo debe activarse.** Qué tipo de petición o situación debe dispararla.
3. **Qué entra y qué sale.** Qué información o archivos recibe, y cómo debe verse el resultado.
4. **Qué reglas o detalles importan.** Casos especiales, formato exacto, cosas que NO debe hacer.

### Ejemplo de instrucción débil

> "Hazme una skill para generar reportes."

`skill-creator` no sabe qué reporte, con qué datos, ni cuándo usarlo. Tendrá que preguntarte todo.

### Ejemplo de instrucción clara y detallada

> "Quiero una skill que genere diagramas de flujo en formato Mermaid a partir de la descripción de un proceso. Recibe un texto que describe los pasos de un proceso de negocio. La salida es un bloque de código Mermaid con un `flowchart TD`, siguiendo siempre esta estructura: el inicio se representa con un nodo redondeado `([Inicio])`, cada paso de acción con un rectángulo `[ ]`, cada decisión con un rombo `{ }` cuyas ramas se etiquetan 'Sí' y 'No', y el final con `([Fin])`. Los nodos se nombran con identificadores cortos (A, B, C...) y el texto va entre los corchetes. Debe activarse cuando alguien pida un diagrama, un flujo, un flowchart o visualizar los pasos de un proceso, aunque no mencione la palabra 'Mermaid'. Si el proceso descrito no tiene un punto final claro, debe asumir un nodo `([Fin])` único al cerrar todas las ramas; no debe dejar nodos sueltos sin conexión."

Fíjate en el detalle: no solo dice "haz un diagrama", sino **qué formato exacto** (Mermaid), **qué estructura debe seguir** (cada tipo de nodo con su forma) y **qué hacer en casos límite**. Con esta instrucción, `skill-creator` puede arrancar casi sin preguntar nada. **Ese es el objetivo del taller: aprender a escribir instrucciones de este tipo, donde la estructura de salida queda totalmente definida.**

---

## Parte 2 — El campo que decide si la skill se usa

De todo lo que genera `skill-creator`, hay una parte crítica: la **descripción** de la skill. Es lo que hace que el asistente la use o la ignore.

Si la descripción es vaga, la skill puede existir pero nunca activarse. Por eso, en tu instrucción, sé muy específico sobre **cuándo debe dispararse** y menciona varias formas en que alguien podría pedirlo.

- Poco claro: "para documentar código".
- Claro: "úsala siempre que el usuario quiera documentar, explicar, resumir o mapear las funciones de un programa o módulo, aunque no use la palabra 'documentación'."

Conviene que esa parte sea un poco insistente, porque por defecto las skills tienden a **no** activarse cuando deberían.

---

## Parte 3 — Probar la skill (todo lo hace `skill-creator`)

Aquí está una de las partes más potentes: **no tienes que probar la skill a mano**. `skill-creator` puede montar y ejecutar todo el proceso de prueba por ti. Solo tienes que pedírselo. Estas son las posibilidades que ofrece:

### 1. Genera el set de casos de prueba (los "evals")

`skill-creator` crea por ti un archivo de casos de prueba (lo llama *evals*): una lista de peticiones realistas con las que se ejercita la skill. Tú puedes proponerle ejemplos, y él los completa y organiza. Conviene que esos ejemplos sean concretos, como los escribiría alguien de verdad:

> "Tengo la descripción de un proceso de aprobación de facturas y necesito el diagrama de flujo en Mermaid."

### 2. Ejecuta las pruebas

`skill-creator` corre la skill sobre cada caso y guarda los resultados de forma ordenada. Es decir, no solo escribe la skill: la **pone a funcionar** sobre los ejemplos para ver qué produce realmente.

### 3. Genera un reporte para que revises

Esto es clave: `skill-creator` arma un **reporte de revisión** donde puedes ver, caso por caso, la petición y el resultado que produjo la skill. Ahí mismo puedes dejar tu comentario sobre cada uno. Así no tienes que ir rebuscando salidas sueltas: lo tienes todo presentado para revisar de un vistazo.

### 4. Verifica la calidad con afirmaciones (benchmark)

Puedes pedirle que defina **afirmaciones concretas** que la salida debe cumplir (preguntas con respuesta sí/no) y que las verifique automáticamente, generando un **benchmark**: una tabla con cuántas pasaron y cuántas fallaron. Ejemplos de afirmaciones para el caso del diagrama:

- "¿La salida es un bloque Mermaid válido con `flowchart TD`?"
- "¿Cada decisión usa un rombo con ramas 'Sí' y 'No'?"
- "¿Todos los nodos están conectados, sin quedar sueltos?"

Las cosas subjetivas ("¿se ve ordenado?") no se verifican así; esas las juzgas tú directamente en el reporte.

> En resumen, basta con decirle algo como: *"prueba esta skill con estos casos, genera el reporte para revisarlo y dime qué afirmaciones cumple"*, y `skill-creator` se encarga de generar los evals, ejecutarlos, mostrarte el reporte y darte el benchmark.

---

## Parte 4 — Decidir si iterar o no

Después de revisar el reporte y el benchmark, decides. **Conviene mejorar la skill si:**

- El resultado no es lo que pediste.
- Falla alguna de las afirmaciones de calidad del benchmark.
- Solo funciona para ese ejemplo exacto y no para casos parecidos.

Para mejorar, le das a `skill-creator` un feedback claro de qué estuvo mal (puedes dejarlo directamente en el reporte de revisión), y él reescribe la skill. Luego vuelve a ejecutar las pruebas y te genera un nuevo reporte para comparar contra el anterior.

**Cuándo dejar de iterar:**

- El resultado ya hace lo que querías.
- No le encuentras nada que corregir.
- Ya no estás logrando mejoras reales.

Un consejo importante: cuando des feedback, piensa en que la skill se usará muchas veces con peticiones distintas, no solo con tu ejemplo. Pide cambios que sirvan en general, no parches que solo arreglen el caso que probaste.

---

## Parte 5 — Optimizar una skill existente

`skill-creator` no solo crea skills nuevas; también mejora las que ya tienes. Si una skill "no se activa cuando debería" o "se activa de más", normalmente el problema está en su descripción.

Para revisarlo, dale a `skill-creator` ejemplos de peticiones que **sí** deberían activar la skill y otras que **no**, sobre todo casos parecidos que comparten palabras pero buscan algo distinto. Con eso él ajusta la descripción para que dispare en el momento correcto.

---

## Ejercicio del taller

En parejas:

1. **Elijan una tarea repetitiva** que hagan a menudo y quieran convertir en skill.
2. **Escriban la instrucción para `skill-creator`** respondiendo las cuatro preguntas de la Parte 1: qué hace, cuándo se activa, qué entra/sale, qué reglas de estructura importan. Esta es la parte clave del ejercicio.
3. **Dejen que `skill-creator` genere la skill** a partir de esa instrucción.
4. **Pídanle que genere los casos de prueba (evals)**, propónganle 2 ejemplos concretos y déjenlo ejecutar las pruebas.
5. **Revisen el reporte** que produce `skill-creator` y pídanle 2 afirmaciones de calidad para el benchmark; vean cuáles pasan.
6. **Decidan si iterar.** Si algo falló, den feedback claro (en el reporte) y dejen que `skill-creator` la mejore y vuelva a probar una vez.

---

## Lo que hay que recordar

- Tú no escribes la skill: la **describes bien** y `skill-creator` la construye.
- Una buena instrucción responde: qué hace, cuándo se activa, qué entra/sale, qué estructura de salida debe tener.
- La **descripción** de la skill decide si se usa o no; sé específico con el "cuándo".
- `skill-creator` puede **generar los casos de prueba, ejecutarlos, darte un reporte para revisar y un benchmark** de calidad. No pruebes a mano: pídeselo.
- Mejora pensando en el uso general, no solo en tu ejemplo.
- Itera hasta que haga lo que querías; luego, para.
