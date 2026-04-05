Prompt

Genera 3 imagenes respectivamente para:

1- Explicacion de las skills
2- Explicacion de los MCPs
3- Infografía conjunta de ambos en el flujo de trabajo de los desarrolladores

Utiliza el estilo y diseño de la infografia con todo y para en caso de generar nuevas imagenes o hacer cambios mantener ese mismo estilo posteriormente.


Anatomía Prompt Claude


"Quiero [TAREA] para que [CRITERIOS DE ÉXITO]. Primero, les estes archivos completamente antes de responder:
[nombrede_archivo.md] — [lo que contiene] [nombrede_archivo.md] — [lo que contiene] [nombrede_archivo.md] — [lo que contiene]
Aquí tienes una referencia de lo que quiero lograr:
[Sube el archivo de referencia como markdown o pégalo aquí] Esto es lo que hace que esta referencia funcione:
[Pega tu plano de ingeniería inversa, los patrones, el tono, la estructura y las reglas que extrajiste de la referencia. Formatea cada una coma una regla que comience con "Siempre" o "Nunca".]
Esto es lo que necesito para mi versión:
RESUMEN DE ÉXITO Tipo de salida + longitud: [Contrato, memo, informe, propuesta, landing page, post?] Reacción del destinatario: [¿Qué deben pensar/sentir/hacer después de leer?] NO suena como: [Qué evitar: IA genérica, demasiado casual, formal, cargado de jerga?] Éxito significa: [¿Firman? ¿Aprueban? ¿Responden? ¿Toman acción?]
Mi archivo de contexto contiene mis estándares, restricciones, puntos conflictivos y audiencia. Léelo completamente antes de comenzar. Si estás a punto de rompar una de mis reglas, detente y dímelo.
NO empieces a ejecutar todavía. En su lugar, hazme preguntas aclaratorias (usa la herramienta 'AskUserQuestion') para que podamos refinar el enfoque juntos paso a paso.
Antes de escribir nada, enumera las 3 reglas de mi archivo de contexto que más importan para esta tarea.
Luego, dame tu plan de ejecución (máximo 5 pasos). Solo comienza a trabajar una vez que nos hayamos alineado."


How to make slides with Claude:

1. Task

☑ Define what you want & what success looks like:
"I want to build a pptx presentation so that [SUCCESS CRITERIA]."

No "make me a deck about X." 
↳ That's how you get 10 slides of nothing.

2. Context Files

☑ Upload your expertise and rules as files:
"First, read these files completely before responding: [filename.md] — [what it contains]."

Stop re-explaining your brand, your audience, your standards every single time. Put it in files.

3. Reference

☑ Show what you're aiming for. Upload an example.
"Here is a reference to what I want to achieve: [filename]."

No short prompts and hoping AI figures it out.

4. Research

☑ Claude searches before touching a single slide:
"Search the web using at least 5 varied searches (trends, data, expert opinions, case studies, counterarguments)."

It saves a structured research brief, organized by theme, with source URLs and key data points pulled out.

I like to add:
"Prioritize 2025–2026 sources. Flag anything where sources conflict or data is thin."

5. Slide Brief

☑ Turn research into a slide-by-slide outline:
"Each slide gets: a title, 3 key points, and any specific data/stats from the research that must appear on that slide."

No full paragraphs. Gamma will write the final text. Just give it enough structure and data to work with.

6. Rules

☑ Don't jump into making slides (yet):
"Do not generate slides yet." "Do not write full paragraphs."

A step-by-step approach takes longer, but makes (much) better slides. If you want to be faster, just go straight to Gamma.

7. Generate

☑ Pass the outline to Gamma as a presentation:
"Pass gamma-outline.md to Gamma as a presentation using textMode 'generate'. Use theme: [your brand theme name]."

Claude connects to Gamma for you. Like magic.

8. Theme

☑ Make sure to use your Gamma theme:
"Use theme: [your brand theme name]."

If you don't know how, I made a guide for teams at how-to-gamma.ai (scroll to the bottom).

After generating, review card by card:
→ Would I say this out loud?
→ Does this card earn its place?
→ Is the data right?

9. Alignment

☑ The prompt starts by asking YOU questions.
"Start by using AskUserQuestion to make sure you have enough context from me before researching."

Necesito ayuda con la revisión y mejor reestructuración de los agentes creados adjuntos que son para trabajar bajo Spec Driven Development (SDD) utilizando de orquestador el sdd-orquestrator y de subagente los demás, sin embargo tengo un dilema al intentar empezar a trabajar con un proyecto y quiero que de forma eficiente los subagentes puedan buscar skills que necesiten dentro del proyecto o globalmente, ya que actualmente tengo el inconveniente de que no funciona de fomra optima eso, buscan mal las rutas o no leen skills tipo buenas practicas de codigo, frontend-design etc que son valiosas que los subagentes las lean y apliquen, no el orquestador principal para cuidar la ventana de contexto principal, ayudame a mejorar este flujo de trabajo para que funcione de forma mas eficiente para poder probar el flujo completo de forma correcta (Las rutas indicadas entre los agentes si existen y son correctas, sin embargo al usarlos no funcionan correctamente)

