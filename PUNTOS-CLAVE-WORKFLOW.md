# Puntos clave a cubrir en tu workflow

> Escribe libre. Esto no es un formulario — es la lista de lo que el auditor necesita encontrar en tu texto.
> Lo que no cubras, te lo preguntará. Lo que sí, te ahorra una tanda de preguntas.

---

## 1. Para qué existe y cuándo termina

Una frase de propósito y una condición de cierre **observable**.

Mal: "termina cuando todo está correcto".
Bien: "termina cuando el archivo quedó cargado en el sistema y la contraparte confirmó recepción".

---

## 2. Qué lo dispara

Un correo, un archivo que aparece, una hora fija, alguien que te lo pide. Di también si eso es detectable por una máquina o si depende de que una persona se dé cuenta.

---

## 3. Los pasos, en lenguaje literal

Describe lo que **realmente haces**, no lo que dice el manual.

Mal: "se consulta la información del cliente".
Bien: "entro al portal, filtro por fecha, exporto el CSV y lo abro en Excel".

Un paso nuevo empieza cuando cambias de sistema, cambias de persona, o tomas una decisión.

---

## 4. Dónde el proceso se bifurca

Los puntos donde el resultado cambia según lo que encuentres. Búscalos por la frase "depende de": si al describir un paso dijiste *depende del caso*, *si viene con X entonces*, *a veces hay que* — eso es una bifurcación y hay que decir cuáles son las ramas.

Esto es lo que separa un script de un agente. Es el punto más valioso de toda la lista.

---

## 5. Qué sabes tú que no está escrito

Por cada decisión, pregúntate: ¿podría darle ahora mismo a alguien la ruta del documento donde está esa regla?

Si no puedes, dilo. No es un defecto tuyo — es dónde el agente va a inventar si nadie se lo advierte. Marca esos puntos aunque la regla te parezca obvia.

---

## 6. Qué pasos no se pueden deshacer

Señala los que envían un correo a un cliente, escriben en producción, aprueban algo o notifican a un tercero. Ante la duda, márcalo como irreversible.

Si además toca datos de clientes o mueve dinero, dilo explícitamente.

---

## 7. Cómo verificas que cada paso salió bien

Por cada paso, cómo sabes que **ese paso** quedó correcto antes de seguir al siguiente. No el workflow completo, cada paso.

Si de alguno no sabes qué responder, dilo. Un paso irreversible sin forma de verificarlo es la señal más importante que le puedes dar al auditor.

---

## 8. Con qué sistemas hablas y si tienes acceso real

Lista los sistemas que tocas y, por cada uno: ¿tiene API, CLI o algo programático? ¿tienes credencial hoy, o habría que pedirla?

Esta es la sección que más proyectos mata. Si el sistema central no tiene forma de acceso automatizado, eso cambia todo el veredicto — mejor saberlo al principio.

---

## 9. Tres ejecuciones reales

De memoria basta, no necesitas archivos:

- La última vez que lo hiciste (camino normal)
- La última vez que algo salió mal
- El caso raro que siempre te da problemas

Por cada una: con qué entraste y qué salió. Son la única forma de medir después si la automatización realmente cumple.

---

## Lo que NO necesitas escribir

Volumen, tiempo ahorrado, ROI, métricas objetivo. Sirven para justificar el proyecto ante tu jefe — no para decidir la arquitectura. Van en otro documento, después.

## Una advertencia

No pulas el texto. Las dudas, los "aquí siempre me trabo" y los "depende" son señal, no ruido. Si los editas para que se lea limpio, borras justo lo que el auditor está buscando.
