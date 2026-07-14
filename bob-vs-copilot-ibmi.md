# IBM Bob vs GitHub Copilot para desarrollo IBM i

> Análisis comparativo realista — Julio 2026
> Contexto: Bob 2.0 y el Premium Package for i salieron el 24 de junio de 2026.

---

## 1. Contexto general

| Aspecto | IBM Bob | GitHub Copilot |
|---|---|---|
| Naturaleza | Fork de VS Code, IDE standalone | Extensión dentro de VS Code |
| Marketplace | Open VSX (sin acceso al Marketplace de Microsoft) | Marketplace completo de Microsoft |
| GA | 24 de marzo de 2026 (Premium Package for i: 24 de junio de 2026) | Años de iteración pública |
| Modelo de cobro | Bobcoins (consumo) + suscripción base | Suscripción flat (~$10–39/mes) |

**Dependencias IBM i de Bob** (se instalan automáticamente desde Open VSX):
Code for IBM i, Db2 for IBM i, RPGLE, CLLE, IBM i Testing, IBMi Languages.

**Requisitos IBM i:** IBM i 7.4+, SSH activo (`STRTCPSVR *SSHD`, programa licenciado 5733-SC1).

---

## 2. Ventajas reales de Bob sobre Copilot para IBM i

### Conectividad nativa al sistema
- Lee/escribe fuentes en **QSYS e IFS** directamente.
- Ejecuta **CL, SQL y PASE** contra el IBM i conectado, sin mover el código fuera del sistema.
- Copilot solo ve el workspace local o Git; no tiene noción de miembros fuente, bibliotecas ni compilación nativa a menos que se le construya (p. ej. vía un framework propio).

### Conocimiento de dominio empaquetado
- **~40 skills y workflows** curados para IBM i:
  - Conversión fixed-format → free-format
  - RPG II/III → ILE RPG
  - Extracción de reglas de negocio
  - Reemplazo de acceso record-level (RLA) por SQL
  - DDS → DDL
  - Generación de unit tests, compilación, display files
- **RAG con documentación IBM i curada** que se remonta hasta el System/36 — reduce alucinaciones en RPG legado, el punto débil clásico de los LLM genéricos.
- Copilot entrena sobre GitHub, donde el RPG es marginal; su conocimiento de RPG III/fixed-format es notoriamente flojo.

### Prompting optimizado por el equipo de IBM i
- IBM afinó los prompts para máxima precisión en RPG al menor costo de Bobcoins.
- Es el conocimiento del equipo IBM i (Tim Rowe, Steve Will) destilado — algo que Copilot nunca tendrá de fábrica.

### Modos especializados (Premium Package)
- **IBM i Developer Mode:** persona enfocada en RPG, COBOL, DDS, SQL y CL.
- **Database Mode:** especializado en Db2 for i.

### Guardrails para entornos regulados
- Aprobación explícita antes de modificar fuentes.
- Diseño orientado a industrias reguladas (FedRAMP, HIPAA, PCI) — relevante en banca.

### Arquitectura agéntica nativa (Bob 2.0)
- Subagentes con contexto limpio para tareas paralelas.
- Tool calling paralelo.
- Orquestación multi-modelo: IBM Granite, Anthropic Claude, Mistral, Llama — selección automática por tarea.

---

## 3. Aspectos malos y mejorables de Bob

### El modelo Bobcoins es el problema #1
- Reporte de campo: un usuario gastó los **40 Bobcoins completos sin terminar una app Python básica** y cuestionó el valor de $20/mes si solo alcanza para "un proyecto al mes".
- Para trabajo agéntico diario, Pro no alcanza; el tier realista es **Pro+**, y con Premium Package quedas en **~$100/mes**.
- Es **opaco**: no controlas tokens, controlas una abstracción que IBM define y puede recalibrar.

### Sigue alucinando en RPG
- Reporte de campo del Premium Package: en la primera hora, Bob sugirió un **parámetro de comando que no existe** — el error clásico que el paquete supuestamente mitiga.
- El RAG ayuda, pero no elimina el problema.

### Producto inmaduro
- GA en marzo de 2026; Premium Package for i con apenas semanas en el mercado.
- Bob 2.0 apenas agregó tool calling paralelo (antes las herramientas corrían secuencialmente) — capacidades que Copilot/Claude Code resolvieron hace tiempo.

### Fricción por ser fork
- Sin acceso al Marketplace de Microsoft (solo Open VSX); extensiones propietarias de Microsoft no disponibles.
- Dependencia del ritmo de releases de IBM.
- Comunicación confusa: Bob 2.0 y el Premium Package se lanzaron el mismo día por canales de anuncio separados.

### Vendor lock-in doble
- IDE de IBM + moneda de IBM + skills en formato cerrado de IBM.
- Los artefactos (modos, skills, workflows personalizados) **no son portables** a otros ecosistemas.

---

## 4. Comparación estructurada

La comparación honesta no es "Bob vs Copilot" sino:

> **Bob vs Copilot + el framework que un equipo esté dispuesto a construir.**

| Dimensión | Bob + Premium i | Copilot + framework propio (ej. FW7) |
|---|---|---|
| Conocimiento IBM i | Nativo, curado por IBM | Lo que tú le inyectes |
| Acceso a QSYS/compilación | Out-of-the-box | Vía Code for IBM i + hooks propios |
| Costo mensual real | $100+ (Pro+ con Premium Package) | $10–39 flat |
| Control de costos | Bobcoins opacos, overage $0.50/u | Total (routing por tiers, validación en hooks gratis) |
| Personalización | Modos/skills de IBM, formato cerrado | Total: agents, hooks, MCP, routing |
| Ecosistema | Open VSX, roadmap de IBM | Marketplace completo, iteración mensual |
| Madurez agéntica | Subagentes desde jun 2026 | Años de evolución + arquitectura probada |
| Portabilidad | Nula (lock-in IBM) | Alta (convenciones abiertas, MCP) |

### Precios de referencia (USD, indicativos)

| Plan | Bobcoins/mes | Precio base | + Soporte | + Premium Package i | Total aprox. |
|---|---|---|---|---|---|
| Trial | 40 (30 días) | Gratis | — | — | $0 |
| Pro | 40 | $20 | $3 | $20–40 | $60–80 |
| Pro+ | 160 | $60 | $9 | $20–40 | ~$100 |
| Ultra | 500 | $200 | $30 | $20–40 | ~$240 |
| Enterprise | Pooled | ~$500/RU | $75/año | Custom | Contactar IBM |

- Overage: **~$0.50 por Bobcoin** extra.
- Promo vigente: **primer mes gratis** del Premium Package (Java Modernization e IBM i) hasta el 31 de diciembre de 2026.

### Veredicto por perfil

- **Shop IBM i sin capacidad de ingeniería de agentes:** Bob gana claramente. Da en un día lo que tomaría meses construir.
- **Equipo con framework agéntico propio:** la ecuación cambia radicalmente — Bob vende una versión menos personalizable de lo que ya tienes.

---

## 5. ¿Cuál destaca para trabajo diario con personalización y transparencia?

**Para un perfil con framework agéntico propio: Copilot + framework (FW7).**

### Razones concretas
1. **Ya está construido lo que Bob vende:** orquestador/subagentes, skills con progressive disclosure, hooks deterministas, routing por tiers, RAG propio vía MCP.
2. **Principio arquitectónico superior:** "validaciones deterministas en hooks, razonamiento en agentes" — gastar Bobcoins en validaciones que un script hace gratis es un anti-patrón de costos.
3. **Transparencia:** con Copilot + MCP controlas cada pieza; con Bob operas dentro de una caja con moneda propia.
4. **Costo:** ~$100/mes de Bob vs $10–39 flat, para capacidades en gran parte redundantes.

### Donde Bob sí aporta valor único
- Conectividad QSYS nativa "llave en mano".
- RAG de documentación IBM i histórica (difícil de replicar).

### Alternativa abierta a ese valor único
- **Code for IBM i** (dependencia del propio Bob) ya da la conexión al sistema.
- **ARCAD MCP Server:** 70+ herramientas MCP que exponen contexto IBM i validado (impact analysis, cross-references, contexto de código) a **cualquier agente** — Claude, Copilot o Bob por igual.
- Patrón recomendado: *contexto IBM i como MCP + agente de tu elección* — más alineado con personalización y transparencia que casarse con el IDE de IBM.

### Recomendación final
Usar el trial gratis de Bob como **fuente de inteligencia competitiva**:
1. Estudiar sus 40 skills y workflows.
2. Extraer los patrones de prompting que IBM afinó para RPG.
3. Portar lo valioso al framework propio.

> Bob es un excelente producto para la mayoría del mercado IBM i. Para un pionero de IA agéntica con framework propio, es más valioso **como referencia que como herramienta**.

---

*Fuentes: bob.ibm.com (docs y pricing), IT Jungle, TechChannel, PromptedDev, Nick Litten, DAI Source, Fresh/Brewed — junio/julio 2026.*
