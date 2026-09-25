# Mejores prácticas para crear prompts con Claude

> Resumen en español de la documentación oficial de Anthropic.
>
> Fuente: [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
>
> Este documento es un resumen práctico, no una traducción íntegra de la página original. La documentación puede cambiar a medida que se publiquen nuevos modelos.

## 1. Principios generales

### Sé claro y directo

Claude responde mejor a instrucciones explícitas. Define con precisión:

- El resultado que esperas.
- El formato de salida.
- Las restricciones que debe respetar.
- El orden de los pasos, cuando sea importante.

Una buena regla es mostrar el prompt a una persona que no conozca el contexto. Si no puede seguirlo sin confundirse, probablemente Claude tampoco.

Explica también el motivo de una regla cuando sea útil. El contexto ayuda al modelo a generalizar mejor que una prohibición aislada.

### Usa ejemplos

Los ejemplos son una de las formas más fiables de controlar el formato, el tono y la estructura de la respuesta. Para que sean efectivos:

- Deben parecerse al caso real.
- Deben cubrir casos límite y variar lo suficiente.
- Deben separarse claramente de las instrucciones, por ejemplo con etiquetas `<example>` y `<examples>`.

Como punto de partida, utiliza entre tres y cinco ejemplos bien seleccionados.

### Estructura el prompt con XML

Las etiquetas XML ayudan a distinguir instrucciones, contexto, ejemplos y datos variables. Usa nombres descriptivos y consistentes, por ejemplo:

```xml
<instructions>
  Analiza los documentos y resume los riesgos principales.
</instructions>

<context>
  El informe está dirigido al equipo directivo.
</context>

<input>
  {{DOCUMENTO}}
</input>
```

Cuando trabajes con varios documentos, anida las etiquetas para conservar la jerarquía:

```xml
<documents>
  <document index="1">
    <source>informe-anual.pdf</source>
    <document_content>{{INFORME_ANUAL}}</document_content>
  </document>
</documents>
```

### Define un rol

Una breve descripción en el mensaje de sistema puede orientar el conocimiento, el tono y el comportamiento de Claude:

```text
Eres un asistente de programación especializado en Python.
```

### Trabaja con contexto largo

Para entradas grandes:

1. Coloca los documentos y datos extensos al principio del prompt.
2. Coloca la consulta y las instrucciones después del contexto.
3. Usa etiquetas XML para separar documentos, fuentes y metadatos.
4. Pide primero que extraiga citas o fragmentos relevantes y después que realice el análisis.

Este último paso ayuda a que la respuesta se apoye en la información pertinente y no en suposiciones.

## 2. Salida y formato

### Controla la verbosidad

Los modelos actuales suelen responder de forma más directa y concisa. Si necesitas visibilidad después de usar herramientas, solicítala explícitamente:

```text
Después de completar una tarea que use herramientas, proporciona un resumen breve del trabajo realizado.
```

La longitud predeterminada puede variar entre modelos. Si necesitas una respuesta corta o extensa, dilo directamente.

### Describe lo que sí debe hacer

Las instrucciones positivas suelen ser más eficaces que las prohibiciones. En lugar de decir «no uses Markdown», describe el resultado deseado, por ejemplo: «escribe en párrafos fluidos y utiliza Markdown solo para títulos y código».

También puedes utilizar etiquetas XML para señalar el formato esperado y hacer que el estilo del prompt se parezca al estilo de salida que quieres obtener.

### Matemáticas y LaTeX

Los modelos recientes pueden usar LaTeX por defecto para expresiones matemáticas. Si necesitas texto plano, indícalo expresamente y especifica cómo representar divisiones, multiplicaciones y exponentes.

### Creación de documentos

Para presentaciones, animaciones o documentos visuales, especifica el tema, la jerarquía visual, el nivel de acabado y las interacciones o animaciones que quieras incluir.

## 3. Sustituir respuestas precompletadas

En modelos recientes, las respuestas precompletadas —un mensaje parcial del asistente que Claude debía continuar— dejaron de estar disponibles en ciertos modelos. Las alternativas dependen del objetivo:

- Para JSON, YAML o clasificaciones: usa salidas estructuradas, esquemas o herramientas con valores enumerados.
- Para eliminar preámbulos: pide directamente que responda sin frases introductorias.
- Para evitar rechazos inadecuados: formula una instrucción clara en el mensaje del usuario.
- Para continuar una respuesta interrumpida: indica que la respuesta anterior terminó en un punto concreto y proporciona el texto disponible.
- Para mantener contexto en conversaciones largas: usa mensajes de usuario, herramientas o mecanismos de compactación administrada.

## 4. Uso de herramientas

### Indica cuándo debe actuar

Si quieres que Claude modifique algo, dilo como una acción directa. «Sugiere cambios» puede producir recomendaciones; «modifica esta función» deja claro que debe implementar el cambio.

Para que actúe de forma proactiva, puedes establecer una regla de acción predeterminada. Si prefieres control estricto, indica que no debe editar archivos ni ejecutar cambios sin una solicitud explícita.

### Llamadas paralelas

Cuando varias llamadas de herramientas son independientes, conviene ejecutarlas en paralelo para reducir la latencia. No las paralelices si una llamada depende del resultado de otra; tampoco inventes parámetros para llamadas futuras.

## 5. Razonamiento y pensamiento

### Evita el exceso de exploración

Algunos modelos recientes investigan más y utilizan más herramientas, especialmente con niveles altos de esfuerzo. Para controlar este comportamiento:

- Cambia reglas generales por condiciones concretas.
- Elimina instrucciones como «si tienes dudas, usa una herramienta» cuando provoquen llamadas innecesarias.
- Reduce el nivel de `effort` si el modelo dedica demasiado tiempo a explorar.
- Pide que elija un enfoque y lo siga, salvo que aparezca información que lo contradiga.

### Usa pensamiento adaptativo

Para tareas complejas, con varios pasos o uso de herramientas, el pensamiento adaptativo permite que Claude decida cuándo y cuánto razonar. En los modelos que lo admiten, se configura con `thinking: {type: "adaptive"}` y se controla la profundidad mediante `effort`.

Una instrucción útil es pedirle que evalúe los resultados de las herramientas, planifique el siguiente paso y luego actúe. Si el pensamiento añade latencia sin mejorar la respuesta, indica que solo debe emplearse cuando aporte un beneficio significativo.

### Verificación

Para código, matemáticas y tareas con criterios comprobables, pide una revisión final contra una lista de pruebas. Sin embargo, algunos modelos ya se autocorrigen eficazmente; añadir demasiadas instrucciones de verificación puede producir trabajo redundante y más latencia.

## 6. Sistemas agénticos

### Tareas largas y varias ventanas de contexto

Para tareas que duran mucho tiempo:

1. Usa la primera ventana para preparar la estructura, las pruebas y las herramientas.
2. Mantén una lista de tareas y un registro de progreso.
3. Guarda el estado en formatos estructurados, como JSON, y las notas generales en texto libre.
4. Utiliza Git como historial y punto de recuperación.
5. Prioriza avances incrementales y verificables.
6. Proporciona herramientas para comprobar la interfaz, el navegador o las integraciones.

Cuando el contexto se renueve, pide a Claude que revise los archivos de estado, las pruebas y los últimos cambios antes de continuar.

### Equilibra autonomía y seguridad

Permite acciones locales y reversibles, pero exige confirmación antes de operaciones destructivas, difíciles de revertir o visibles para terceros, como borrar archivos, hacer `git push --force`, modificar infraestructura o publicar mensajes.

### Investigación

Para investigaciones complejas:

- Define qué significa que la respuesta sea correcta.
- Pide verificar los datos en varias fuentes.
- Mantén hipótesis alternativas y niveles de confianza.
- Registra las conclusiones y critica periódicamente el enfoque.

### Subagentes

Los subagentes son apropiados para trabajos paralelos, aislados o independientes. Para tareas simples, operaciones secuenciales o ediciones de un solo archivo, suele ser mejor trabajar directamente. Vigila la creación excesiva de subagentes cuando una búsqueda o una lectura sencilla sea suficiente.

### Encadenamiento de prompts

Divide una tarea en varias llamadas cuando necesites inspeccionar resultados intermedios, registrar evaluaciones o forzar un flujo específico. Un patrón habitual es:

```text
borrador → revisión contra criterios → versión refinada
```

### Evita la sobreingeniería

Pide soluciones centradas en el alcance real. No añadas abstracciones, configuraciones, validaciones, comentarios o archivos auxiliares que no sean necesarios. Las pruebas deben verificar la solución, no definirla mediante valores codificados o casos especiales.

Para reducir alucinaciones en tareas de código, exige que Claude lea los archivos relevantes antes de responder y que no especule sobre código que no haya inspeccionado.

## 7. Consejos específicos

### Visión

Para analizar imágenes complejas, varias imágenes o interfaces, puede ser útil proporcionar una herramienta de recorte o una capacidad equivalente de ampliación. Esto permite examinar regiones relevantes con mayor precisión.

### Diseño frontend

Si quieres diseños distintivos, solicita explícitamente decisiones creativas sobre:

- Tipografía.
- Paleta y tema.
- Animaciones y microinteracciones.
- Fondos, profundidad y atmósfera.

Evita resultados genéricos indicando el contexto visual, la personalidad de la interfaz y las elecciones que no quieres repetir, como paletas o layouts demasiado comunes.

## 8. Migración entre generaciones de modelos

Al migrar prompts antiguos:

1. Describe con exactitud el comportamiento deseado.
2. Añade modificadores de calidad cuando necesites más detalle o profundidad.
3. Solicita de forma explícita las funciones interactivas o animaciones.
4. Cambia la configuración de pensamiento según el modelo; los modelos nuevos suelen usar pensamiento adaptativo en lugar de un presupuesto manual.
5. Sustituye las respuestas precompletadas por salidas estructuradas o instrucciones directas.
6. Reduce instrucciones agresivas diseñadas para modelos anteriores, porque pueden provocar demasiadas llamadas a herramientas.
7. En modelos que devuelven bloques de pensamiento, conserva el historial exactamente como lo entrega la API y evita modificar mensajes anteriores.

## Lista de comprobación

- [ ] ¿El resultado esperado está descrito claramente?
- [ ] ¿El formato y las restricciones son explícitos?
- [ ] ¿El contexto está separado de las instrucciones?
- [ ] ¿Hay ejemplos relevantes y variados?
- [ ] ¿Las entradas largas están organizadas con etiquetas XML?
- [ ] ¿Se pidió actuar o solo sugerir?
- [ ] ¿Las herramientas independientes pueden ejecutarse en paralelo?
- [ ] ¿El nivel de pensamiento es proporcional a la complejidad?
- [ ] ¿Existen pruebas, criterios de éxito o mecanismos de verificación?
- [ ] ¿Las acciones destructivas requieren confirmación?
- [ ] ¿El prompt sigue siendo adecuado para el modelo específico utilizado?

