# Etapa 0 – Por qué las preguntas de evaluación van primero

> Notas de estudio personales (en español). Conversación del 2026-11-20.
> Estado: Etapa 0 sin empezar. Hay preguntas pendientes al final.

---

## Parte 1 – Por qué escribir las preguntas antes del código

### El problema: un RAG puede verse bien y estar mal

En este proyecto, "funciona" significa algo muy concreto: **para la pregunta X, el
sistema trae el artículo Y y lo cita bien**. Que la respuesta suene bien no basta,
porque eso es justo lo que hacen bien los modelos de lenguaje, aunque se equivoquen.

Supón que ya armaste todo y preguntas *"¿cuándo se deposita la CTS?"*. El sistema
responde algo fluido y cita "Artículo 22". ¿Está bien? Sin un artículo esperado ya
anotado no lo puedes saber. Tendrías que abrir la ley y verificarlo a mano, cada vez,
con cada pregunta.

### Razón 1: las preguntas son tu forma de medir

En las etapas 2 y 3 vas a tomar decisiones con números:
- ¿El modelo de embeddings A o el B?
- ¿Búsqueda semántica sola o híbrida?
- ¿Vale la pena el reranker?

Para responderlas necesitas un **recall@5**: de 30 preguntas, ¿en cuántas aparece el
artículo correcto entre los 5 primeros resultados? Por ejemplo, A acierta 21/30 y B
acierta 26/30, así que eliges B. **Sin las preguntas etiquetadas ese número no existe**
y eliges a ojo. Las preguntas son la regla con la que mides.

### Razón 2: si las escribes al final, te engañas sin querer

Este es el motivo principal para hacerlas **antes**. Si primero construyes el sistema y
después escribes las preguntas, inconscientemente vas a escribir las que ya viste que
funcionan. Es como redactar el examen después de saber qué estudió el alumno: siempre
sale 20.

Escritas antes, reflejan lo que **un usuario real preguntaría**, no lo que tu sistema
ya sabe responder. Por eso incluyen cosas incómodas:
- Lenguaje cotidiano: *"entré en marzo, ¿cuánto me toca de gratificación?"*.
- Preguntas trampa, donde lo correcto es decir "no tengo fuente".

### Razón 3: te obligan a entender el dominio antes de programar

Para etiquetar *"¿cuándo se deposita la CTS?" → Artículo N de la Ley de CTS*, tienes
que abrir la ley y encontrarlo. Y en ese proceso descubres cosas que cambian el código:
- Cómo están escritos los artículos ("Artículo 1.", "Art. 1°"…). Eso define cómo se
  parte el texto en la Etapa 1.
- Que algunas respuestas están repartidas en **dos artículos**.
- Que hay artículos modificados por normas posteriores.

Si lo descubres después de programar, rehaces el código.

### Razón 4: detectar si algo empeoró

Desde la Etapa 4, cada cambio (en el prompt, en el chunking, en el modelo) se compara
contra un puntaje base. En la Etapa 8, el despliegue a AWS **se bloquea solo** si el
puntaje baja. Todo eso depende de este archivo de preguntas.

### Resumen

| Sin preguntas primero | Con preguntas primero |
|---|---|
| "Parece que funciona" | "Acierta 26 de 30" |
| Eliges modelos a ojo | Eliges con números |
| Preguntas sesgadas hacia lo que ya funciona | Preguntas que reflejan al usuario real |
| Descubres el formato de la ley tarde | Lo descubres antes de programar |
| No sabes si un cambio rompió algo | Lo sabes en cada cambio |

A esto se le llama a veces *eval-driven development*, y es parecido a escribir tests
antes del código. La diferencia es que aquí el "test" mide qué tan buena es la búsqueda
y la respuesta, no si una función devuelve un valor exacto.

---

## Parte 2 – Respuesta a mi primer intento (búsqueda vs. respuesta)

### Mi idea: "pueden salir preguntas fuera del alcance de las leyes cargadas"

Es una buena observación, aunque funciona al revés: que haya preguntas fuera del
alcance **no es un problema, es algo que queremos**. Por eso el plan incluye 5
preguntas trampa a propósito. Un usuario real va a preguntar sobre despidos aunque solo
hayamos cargado CTS y gratificaciones, y el sistema tiene que saber decir "no tengo
fuente". Así que esa no es la razón para escribirlas antes.

### "Si el sistema tiene las leyes, debería haber respuesta". No necesariamente.

El sistema **no "sabe" las leyes**. No las lee completas cada vez que alguien pregunta.
Funciona en dos pasos separados:

```
Pregunta del usuario
      │
      ▼
① BÚSQUEDA  ── busca en la base de datos los 5 artículos que parecen más relacionados
      │
      ▼
② RESPUESTA ── Claude recibe SOLO esos 5 artículos y responde con ellos
```

Esto es lo que significa **RAG**: *Retrieval-Augmented Generation*, es decir, generar
una respuesta ayudado por lo que se recuperó.

**¿Por qué no darle a Claude todas las leyes y ya?** Porque cada palabra que le envías
cuesta dinero y tiempo. Además, cuando el texto es muy largo, el modelo se distrae y
cita peor. Con dos leyes todavía alcanzaría, pero con 50 normas ya no.

**Analogía: un bibliotecario y un abogado.**
- El bibliotecario (la búsqueda) va a los estantes y trae 5 artículos.
- El abogado (Claude) **solo puede leer lo que le trajeron**. No puede ir al estante.

Si el bibliotecario trae los artículos equivocados, el abogado más brillante del mundo
igual responde mal, o dice "no tengo fuente" aunque la respuesta sí estaba en la
biblioteca.

### Ejemplo concreto de cómo falla la búsqueda

El usuario pregunta: *"¿cuándo me **pagan** la CTS?"*

La ley no dice "pagan". Dice algo como *"los **depósitos** deberán efectuarse
**semestralmente** en los meses de mayo y noviembre"*.

Si la búsqueda solo compara palabras, no encuentra coincidencia entre "pagan" y
"depósitos", y trae otros artículos. Claude recibe 5 artículos que no hablan de la
fecha y responde mal o se abstiene. **La respuesta estaba en la base de datos y aun así
no salió.** Por eso existen las etapas 2 y 3: hacer que la búsqueda entienda que "me
pagan" y "depósito" significan algo parecido.

### Por qué se mide la búsqueda y no solo la respuesta

Cotejamos **las dos cosas por separado**, porque cada una puede fallar por su lado:

| Caso | ¿Búsqueda trajo el artículo correcto? | ¿Respuesta correcta? | ¿Dónde está el error? |
|---|---|---|---|
| A | Sí | Sí | Ninguno |
| B | No | No | En la **búsqueda**: el abogado nunca vio el artículo |
| C | Sí | No | En la **respuesta**: lo tenía delante y lo ignoró o citó mal |

Si solo miras la respuesta, en los casos B y C ves lo mismo ("respondió mal") y **no
sabes qué arreglar**. En el caso B tocar el prompt de Claude no sirve de nada. En el
caso C cambiar el modelo de embeddings tampoco.

Por eso cada pregunta del archivo lleva **el artículo esperado**. Así podemos preguntar
primero: ¿la búsqueda lo trajo entre los 5 primeros? Eso es el recall@5.

### Volviendo a la pregunta 1

La razón principal para escribirlas **antes** es el sesgo. Si ya construiste el sistema
y viste qué preguntas funcionan, sin querer vas a escribir preguntas parecidas a esas,
con las mismas palabras que usa la ley. El sistema saca buena nota y tú crees que
funciona. Luego llega un usuario real que dice "me pagan" en vez de "depósito" y falla.

---

## Preguntas pendientes (responder con mis palabras)

De la primera ronda:
1. ~~Si escribiera las 30 preguntas después de construir el sistema, ¿qué problema
   concreto aparecería?~~ (intentada; ver Parte 2)
2. ¿Por qué hacen falta las 5 preguntas trampa? ¿Qué error del sistema detectan que las
   otras 25 no detectan?
3. Si en la Etapa 2 el modelo A acierta 7 de 10 y el B acierta 8 de 10, ¿basta para
   elegir B? ¿Por qué el plan dice que hay que llegar a 30?

De la segunda ronda:
4. Si Claude responde mal, ¿cómo sabes si la culpa es de la búsqueda o de Claude? ¿Qué
   dato necesitas tener anotado de antemano?
5. Con el ejemplo de "¿cuándo me pagan la CTS?": si escribieras las preguntas
   **después** de ver que el sistema funciona con "¿cuándo se deposita la CTS?", ¿qué
   pregunta probablemente escribirías y qué problema quedaría escondido?
