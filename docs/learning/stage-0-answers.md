# Etapa 0 – Mis respuestas y correcciones

> Registro de mis respuestas a las preguntas de comprensión de la Etapa 0 y las
> correcciones que recibí. Complementa `stage-0-why-evals-first.md`.
> Iniciado: 2026-11-22.

---

## P2 – ¿Para qué sirven las 8 preguntas trampa?

**Pregunta completa:** ¿Para qué sirven las 8 preguntas trampa? ¿Qué error del
sistema detectan que las otras 42 no ven?

### Mi respuesta

> No sabía que eran 8 preguntas trampa, solo que había algunas. La idea es descubrir
> si el sistema realmente responde correctamente, porque puede ser que yo haga las
> pruebas con preguntas sencillas y fáciles, pero en el mundo real los usuarios pueden
> preguntar cualquier cosa: temas que el sistema no sabe o preguntas extrañas. En esas
> situaciones el sistema tiene que detectar que no conoce la respuesta y decir que no
> sabe, en lugar de inventar. Para eso sirven las preguntas trampa: para asegurarte de
> que, si no sabe algo, te dice que no tiene información en lugar de inventar o meter
> ruido.

### Corrección

**Bien:** la idea central es correcta. El sistema debe decir "no tengo fuente" en
lugar de inventar. (Las notas originales decían 5 porque venían del plan v3; en el
v5 son 8.)

**Faltaba responder:** *¿qué error detectan que las otras 42 no ven?*

En las otras 42, **la respuesta siempre existe** en las leyes cargadas. Un sistema que
**nunca se abstiene** y siempre responde algo no pierde puntos en ellas, porque
responder era lo correcto. Podría sacar 42/42 y nunca sabrías que inventa. Solo una
pregunta sin fuente puede revelar ese defecto. Las trampas no son "preguntas
difíciles", son **preguntas donde lo correcto es abstenerse**.

**Matiz:** lo de "preguntas sencillas vs. lo que pregunta un usuario real" es más bien
el tema de la P5 (el sesgo al escribir las preguntas). Está relacionado, pero es otro
problema.

---

## P2b – ¿Cuál es mejor pregunta trampa?

**Pregunta:** solo cargamos CTS y gratificaciones. ¿Cuál es mejor pregunta trampa?

- a) *"¿Cuál es la capital de Francia?"*
- b) *"¿Cuántos días de vacaciones me tocan al año?"*

### Mi respuesta

> La de la capital de Francia la puede responder cualquier LLM, así que la va a
> responder bien; no tiene nada de trampa, ni siquiera necesita información del
> sistema. La pregunta trampa sería la de los días de vacaciones, porque no se
> relaciona con CTS ni gratificaciones, pero es una pregunta sobre leyes relacionadas
> con el trabajo, y esa información no está cargada en el sistema.

### Corrección

**Correcto: b).** Y por la razón correcta: es del dominio laboral, pero su respuesta no
está en las normas cargadas.

**Corrección sobre la a):** en este proyecto el sistema solo puede responder con los
artículos recuperados. Si responde "París", **rompe la regla**, aunque el dato sea
cierto. Lo correcto también sería abstenerse. La a) es una trampa *demasiado fácil*:
se nota de inmediato que está fuera del tema, así que casi no prueba nada.

**Por qué b) es buena trampa:**

1. **Engaña a la búsqueda:** trae artículos de CTS o gratificaciones que "se parecen"
   (trabajador, remuneración, meses), y Claude recibe textos cercanos al tema.
2. **Engaña al modelo:** Claude probablemente sabe por su entrenamiento cuántos días
   de vacaciones hay en Perú, y puede responder de memoria sin ninguna fuente cargada.
   Es justo el error que queremos atrapar: una respuesta que suena correcta, pero que
   no está respaldada por un artículo recuperado.

**Idea clave:** una buena pregunta trampa está **cerca del tema** pero **fuera de las
fuentes**. Mientras más se parece a una pregunta válida, más útil es.

---

## P3 – ¿7/10 vs 8/10 basta para elegir B?

**Pregunta completa:** En la Etapa 2, el modelo de embeddings A acierta 7 de 10 y el
B acierta 8 de 10. ¿Basta para elegir B? ¿Por qué el plan dice que hay que llegar a
50 preguntas antes de comparar versiones?

### Mi respuesta

> No es suficiente elegir B porque son muy pocas preguntas y eso puede meter mucho
> ruido: con pocas preguntas, cada una tiene un porcentaje mayor. Si el modelo LLM
> acierta una pregunta por mera suerte, parecería que es mejor que el otro, pero no es
> así. Es mejor tener muchas preguntas, así cada una tiene un porcentaje menor. Con 50
> preguntas, cada una añade un 2%, y el resultado es más fiable. A más preguntas, más
> fiabilidad de los resultados al comparar dos modelos.

### Corrección

**Bien:** la respuesta es correcta. No basta, porque con 10 preguntas cada una vale
10% y una sola diferencia puede ser suerte (ruido). Con 50, cada una vale 2%.

**Precisión 1: en la Etapa 2 no hay LLM.** Lo que se compara es el **modelo de
embeddings**, es decir, la búsqueda. El recall@5 solo mira si el artículo correcto
salió entre los 5 primeros. Claude todavía no participa. Las respuestas de Claude se
evalúan en la Etapa 4.

**Precisión 2: más preguntas = más fiable la *medición*, no el *modelo*.** Agregar
preguntas no mejora el modelo. Mejora la regla con la que lo mides.

**Precisión 3: 50 preguntas no vuelven confiable cualquier diferencia.** Aun con 50,
ganar por 1 o 2 preguntas (2-4%) sigue pudiendo ser suerte. Una ventaja de varias
preguntas (8-10) ya es difícil que sea casualidad. Regla práctica: mientras más
pequeña es la diferencia, más preguntas hacen falta para creerla.

---

## P4 – ¿Falló la búsqueda o falló Claude?

**Pregunta completa:** Si Claude responde mal, ¿cómo sabes si la culpa es de la
búsqueda o de Claude? ¿Qué dato necesitas tener anotado de antemano?

### Mi respuesta

> Por lo que leí en otra documentación, en los evals no basta con la pregunta y la
> respuesta: también hay que tener los chunks relevantes asociados a cada pregunta.
> Si la búsqueda devuelve chunks que no están entre los relevantes, el error está en
> la búsqueda. Si sí devuelve los chunks relevantes, el problema puede estar en la
> generación de la respuesta. Lo que hay que tener a mano son los chunks relevantes
> asociados a cada pregunta y también los chunks que devuelve el sistema.

### Corrección

**Bien:** es correcto, y es el mismo razonamiento de la tabla A/B/C de la Parte 2.

**Precisión 1: en este proyecto el "chunk relevante" es el artículo esperado.** Como
en la Etapa 1 partimos cada norma por "Artículo N.", un chunk = un artículo. Por eso
en `evals/questions.yaml` el campo se llama `expected_articles`: es tu lista de
chunks relevantes.

**Precisión 2: solo uno de los dos datos se anota de antemano.**
- **De antemano (a mano, en la Etapa 0):** los artículos esperados.
- **En cada corrida (lo guarda el script):** los 5 artículos que devolvió la búsqueda.

La comparación entre ambos es justamente el recall@5.

**Precisión 3: dos casos especiales.**
- **Preguntas multi-artículo:** si se necesitan los artículos 3 y 5 y la búsqueda solo
  trae el 3, la búsqueda falló *a medias*. Claude no puede dar una respuesta completa
  con la mitad de la información.
- **Preguntas trampa:** no tienen artículo esperado. Ahí no se mide el recall; se mide
  si el sistema **se abstuvo** (por eso llevan un "criterio de abstención").

---

## P5 – Escribir las preguntas después de ver que funciona

**Pregunta completa:** Con el ejemplo de "¿cuándo me pagan la CTS?": si escribieras
las preguntas después de ver que el sistema funciona con "¿cuándo se deposita la
CTS?", ¿qué pregunta probablemente escribirías y qué problema quedaría escondido?

### Mi respuesta

> Probablemente haría una pregunta como "¿en qué fecha pagan la CTS?". El problema que
> se esconde es que estoy haciendo preguntas que sé que el sistema va a responder
> correctamente. No son preguntas que haría un usuario normal; están sesgadas, y eso
> hace que el sistema responda bien, pero si le hiciera otra pregunta diferente,
> posiblemente fallaría. *(La respuesta quedó cortada al final.)*

### Corrección

**Bien:** la idea general es correcta. Las preguntas quedan sesgadas hacia lo que ya
funciona, la nota sale alta y la falla real queda escondida.

**Error en el ejemplo:** "¿en qué fecha **pagan** la CTS?" es justo la pregunta
*difícil*, la que usa la palabra del usuario. Si ya viste que funciona con
"deposita", lo más probable es que escribas algo como **"¿en qué meses se deposita la
CTS?"**: otra vez con la palabra de la ley.

**El problema concreto que queda escondido:** el **desajuste de vocabulario**. El
usuario dice "me pagan"; la ley dice "depósitos semestrales". Si todas tus preguntas
usan las palabras de la ley, la búsqueda siempre encuentra coincidencias y el
recall@5 sale alto. Nunca descubres que la búsqueda falla cuando el usuario habla
con sus propias palabras, que es justo lo que las etapas 2 y 3 (embeddings, búsqueda
híbrida) deben resolver.

**Lección práctica para `questions.yaml`:** escribir las preguntas cotidianas con
palabras de usuario ("me pagan", "me botaron", "entré en marzo"), no copiando frases
del artículo.

---

## P6 – ¿Por qué separar 15 held-out?

**Pregunta completa:** ¿Por qué separamos 15 de las 50 preguntas como held-out? ¿Qué
pasa si las uso para ajustar el prompt o la búsqueda?

### Mi respuesta

> Se separan esas 15 preguntas para probar, cuando el sistema ya está desarrollado,
> que no quedó afinado a las 35 preguntas con las que se ha entrenado. Puede ocurrir
> que con preguntas diferentes, como esas 15, responda mal. No hay que usarlas para
> ajustar el prompt, porque caes en lo mismo: afinas el sistema ya no a 35 sino a 50
> preguntas. Entonces tendrías que hacer 15 preguntas nuevas para verificar si
> realmente responde bien. *(La respuesta quedó cortada al final.)*

### Corrección

**Bien:** correcto. Y la última idea es muy buena: si las held-out se contaminan, la
única salida es escribir preguntas nuevas, porque las viejas ya no sirven como examen
sorpresa.

**Precisión 1: no "entrenamos" nada.** Claude y el modelo de embeddings vienen ya
entrenados; nunca cambiamos sus pesos. Lo que **ajustamos** es lo que está alrededor:
el prompt, el chunking, la búsqueda (semántica, híbrida, reranker) y la elección del
modelo. El sobreajuste igual puede pasar, porque cada ajuste lo eliges mirando qué
preguntas de desarrollo fallan.

**Precisión 2: cómo se leen los resultados.** Se corren solo dos veces (al cerrar la
Etapa 4 y antes de desplegar), y lo que importa es la **diferencia** con desarrollo:
- Desarrollo 85%, held-out 80% → el sistema generaliza bien.
- Desarrollo 90%, held-out 60% → ajustaste de más a las 35.

**Precisión 3: también hay que mirarlas poco.** No es solo "no ajustar con ellas".
Si las corres seguido y ves cuáles fallan, ya estás tomando decisiones con esa
información, aunque no quieras. Por eso solo se corren esas dos veces.

---

## P7 – ¿Por qué etiquetar por artículo y no por número de chunk?

**Vocabulario:** *etiquetar* es anotar de antemano, a mano, la respuesta correcta de
cada pregunta. En `evals/questions.yaml`, la etiqueta es el campo
`expected_articles`.

**Pregunta completa:** En la Etapa 3 pruebas dos formas de cortar el texto: (A) un
chunk por artículo y (B) chunks de 200 palabras. Si en el YAML hubieras escrito
`"chunk-347"` como respuesta correcta de la grat-002, ¿sería el mismo texto en A y en
B? ¿Y qué pasa si escribiste `"ley-27735:art-5"`?

### Mi respuesta

> Si la etiqueta se hace con el código del artículo, en el caso A va a coincidir
> porque es el mismo artículo. Y en el caso B el artículo va a contener al chunk de
> diferentes palabras.

### Corrección

**Bien:** con la etiqueta por artículo, A coincide directo, y en B los chunks son
trozos que salen de los artículos. Si cada chunk guarda de qué artículo viene, basta
con que **cualquier trozo** del art. 5 salga entre los 5 primeros. La misma
etiqueta sirve para A y para B.

**Faltaba la otra mitad: ¿qué pasa con `"chunk-347"`?**
- En A, el chunk 347 es, por ejemplo, el artículo 5 de la Ley 27735.
- En B, el texto se corta distinto, así que el chunk 347 es **otro texto**: quizá
  un pedazo del TUO de CTS.

La etiqueta deja de apuntar a la respuesta correcta. Para comparar A y B tendrías
que **volver a etiquetar las 50 preguntas** cada vez que cambias el chunking. Y si
re-etiquetas, ya no comparas con la misma regla.

**Idea clave:** la etiqueta debe depender del **contenido** (la ley), no de **cómo
lo partiste** (el chunking). Así el chunking se puede cambiar sin tocar los evals.

**Precisión:** en B, un chunk de 200 palabras no siempre queda dentro de un solo
artículo. Puede llevarse el final del art. 5 y el inicio del 6. Por eso, si algún
día se parten artículos largos, conviene cortar *dentro* de cada artículo y nunca
cruzar de uno a otro.

---

## Material de apoyo para las preguntas pendientes

Explicaciones que no estaban en `stage-0-why-evals-first.md`.

### Para P3: con pocas preguntas, la diferencia puede ser suerte

Dos alumnos dan un examen de 10 preguntas: uno saca 7 y el otro 8. ¿El segundo sabe
más? Tal vez, o tal vez le tocó justo una que conocía. Con 10 preguntas, cada una vale
el 10%, así que **un solo acierto de suerte cambia el resultado**. Eso es *ruido*.

Con 50 preguntas, cada una vale el 2%. Si un modelo le gana al otro por 8 o 10
preguntas, ya es difícil que sea suerte. Por eso se puede empezar con 10, pero **no se
comparan versiones hasta tener las 50**.

### Para P6: las 15 preguntas held-out

Si un alumno practica muchas veces con *el mismo examen* y lo va corrigiendo hasta
sacar 20, eso no prueba que domine el curso: puede haberse aprendido esas preguntas de
memoria.

En las etapas 2 a 4 se ajustan el prompt, la búsqueda y el chunking mirando qué
preguntas fallan. Después de muchos ajustes, el sistema queda "a la medida" de esas 35
preguntas, aunque no sea a propósito. Eso es *sobreajuste* (overfitting).

Las 15 held-out son un **examen sorpresa guardado en un cajón**:
- No se miran mientras se ajusta.
- Se corren solo dos veces: al cerrar la Etapa 4 y antes de desplegar.
- Si en desarrollo sacas 90% y en held-out 60%, ajustaste demasiado.
- Si las usas para ajustar, dejan de ser sorpresa y ya no te dicen nada.

---

## Preguntas pendientes

- [x] P2 – Preguntas trampa
- [x] P3 – ¿7/10 vs 8/10 basta para elegir? ¿Por qué llegar a 50?
- [x] P4 – ¿Cómo sé si falló la búsqueda o falló Claude? ¿Qué dato necesito anotado?
- [x] P5 – Si escribo las preguntas después de ver que funciona "¿cuándo se deposita
      la CTS?", ¿qué pregunta escribiría y qué problema quedaría escondido?
- [x] P6 – ¿Por qué separar 15 held-out? ¿Qué pasa si las uso para ajustar?
- [x] P7 – ¿Por qué etiquetar por artículo y no por número de chunk?
