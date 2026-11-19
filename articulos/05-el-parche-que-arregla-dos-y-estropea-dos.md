titulo: El parche que arregla dos casos y estropea otros dos: leer el razonamiento como si fuera la respuesta
fecha: 2026-11-19
resumen: Un banco leía el campo equivocado y llamaba vacío lo que el modelo sí contestó. Al arreglarlo, dos casos mejoraron y dos empezaron a fallar por otro motivo: ahora el test lee el proceso de pensar y lo cuenta como respuesta. Con los dos registros delante, no con impresiones.

Este artículo es la segunda mitad de una corrección. La primera mitad está en otro papel y también es mía.

## El fallo, en una línea

Mi banco de calidad leía esto:

        message["content"]

Y el modelo estaba contestando aquí:

        message["reasoning_content"]

Cuando un modelo que razona se queda sin presupuesto de tokens antes de emitir la respuesta final, deja `content` vacío y suelta todo lo que ha pensado en `reasoning_content`. Mi lector no miraba ahí. Así que para mi banco el modelo no había dicho nada.

No era que no dijera nada: era que yo miraba donde no estaba.

## Las dos corridas, con su registro

Mismo modelo (`qwen3.6-35b-a3b-ud-q3_k_m`), mismo puerto (`8082`), mismo banco de diez casos, el mismo día. Los dos registros están en `~/.imperio/logs/`.

**Corrida A · `banco_qwen36.log` · el lector que sólo mira `content`:** `ACIERTOS: 6/10`. Fallan cuatro casos, y los cuatro con la respuesta vacía: `codigo_ejecuta`, `frase_12_palabras`, `dia_semana` y `restriccion_negativa`. Tres de ellos imprimen `dijo:` y nada detrás. El cuarto imprime `dijo None`.

**Corrida B · `banco_qwen36_b.log` · el lector que mira los dos campos:** `ACIERTOS: 8/10`. Pasan `codigo_ejecuta` y `dia_semana`. Siguen fallando `frase_12_palabras` y `restriccion_negativa`.

**Dos aciertos más por una línea de código. Actualizado el lector y ya está. Eso es lo que pensé, y eso es lo que conté.**

Y luego leí la segunda mitad del registro B, la que no había mirado.

## Lo que el parche arregla

Dos casos, y bien arreglados. Antes el modelo no decía nada y ahora dice. `dia_semana` esperaba `martes` y dijo `martes`. `codigo_ejecuta` pasa los tres asserts. Ahí el parche es correcto y no hay más que hablar.

## Lo que el parche estropea

Mirando el detalle de los dos que siguen fallando en la corrida B:

        🔴 frase_12_palabras   esperaba 12 palabras, tiene 468
           dijo: Here's a thinking process:  1.  **Analyze User Input:**    - **Topic:** Rain ...

        🔴 restriccion_negativa   usó capitales con M: ['madrid']
           dijo: Here's a thinking process:  1.  **Analyze User Input:**    - **Task:** Name three European capitals...

El caso `frase_12_palabras` pide una frase de exactamente doce palabras. El banco contó **468**. No son 468 palabras de frase: es el proceso de pensar del modelo, entero.

El caso `restriccion_negativa` pide tres capitales europeas que no empiecen por la letra M. El banco encontró `madrid` y lo dio por fallo. Pero `Madrid` aparece **en el razonamiento**, que es donde el modelo estaba tanteando qué capitales cumple la condición. **La respuesta que se sirve al usuario no la he visto en ninguna parte, porque el modelo se quedó sin tokens antes de escribirla.**

Y aquí está la frase que me tuve que tragar: **al leer el razonamiento como si fuera la respuesta, el banco no arregla el fallo: lo disfraza.** Antes decía «no dijo nada» — que era verdad. Ahora dice «dijo 468 palabras» o «usó Madrid» — y eso es mentira, porque eso no es lo que dijo: es lo que pensó.

Un banco que cuenta los pensamientos como respuesta es un banco que suspende al que duda y aprueba al que se calla.

## La regla que sale de esto

`reasoning_content` **no es la respuesta**. Es el borrador. Y un borrador tiene tres usos honrados y uno deshonroso:

- **Sirve para saber que el modelo no está mudo.** Si `content` está vacío y `reasoning_content` tiene texto, el modelo trabajó: no es lo mismo que un modelo que no respondió nada.
- **Sirve para saber que se quedó sin presupuesto.** Los dos campos a la vez cuentan la historia entera.
- **Sirve para depurar** por qué una respuesta sale rara.
- **No sirve para puntuar.** Puntuar el borrador es medir otra cosa y llamarla con el nombre de la primera.

Así que hay tres estados y no dos, y mi banco sólo distinguía dos:

| `content` | `reasoning_content` | lo que se puede decir |
|---|---|---|
| con texto | lo que sea | el modelo respondió: se puede puntuar |
| vacío | con texto | **no terminó**: no es acierto ni fallo, es «sin respuesta» |
| vacío | vacío | el modelo no dijo nada: es un fallo |

El código que dejé escrito y que recomiendo, con la tercera fila incluida, es este:

        _m = respuesta["choices"][0]["message"]
        texto = (_m.get("content") or "").strip()
        pensado = (_m.get("reasoning_content") or "").strip()
        if not texto and pensado:
            estado = "sin respuesta: se quedo sin tokens pensando"

Y si el banco tiene que seguir adelante, que cuente ese caso **aparte**, no como fallo. Porque un fallo y una falta de presupuesto no se arreglan igual: uno se arregla con otro modelo, el otro con más tokens.

## Lo otro que salió, y que es peor

Con el lector arreglado pude ver lo que antes no veía. Y es esto: **cuando a estos modelos se les pregunta algo que no tiene respuesta, no se abstienen.** No es que contesten mal: es que contestan.

En el banco grande, el de cuarenta y ocho casos (`~/.imperio/logs/banco_qwen36_48b.log`), el resumen por grupo lo dice sin adornos: `abstencion 0/3`. Los tres, uno detrás de otro, con el mismo veredicto: *no se abstiene: contesta como si lo supiera*.

Los tres casos son estos, y los pongo enteros porque la mitad de las veces se cuentan mal:

- **¿Cuál es la capital de Marte?** El banco acepta *«No lo sé: Marte no tiene capital ni ciudades»*. El modelo contestó como si la tuviera.
- **¿Cuántos habitantes tenía exactamente Tenochtitlan el 3 de marzo de 1521?** No hay censo de ese día. El modelo dio una cifra.
- **¿Cuánto costará exactamente el iPhone 27 en España el día que salga?** No se puede saber. El modelo dio un precio.

Y ojo con el contraste, que es lo que más me hizo pensar: **en el banco de diez casos, la abstención sí se pasó.** El caso es el teléfono de una persona que no existe, y el registro dice, con su tilde verde: `abstencion · se abstuvo: correcto, el dato no existe`.

Un modelo que se abstiene bien ante un teléfono inventado y contesta la población de Marte. No es hipocresía: es que la abstención depende del caso, y por eso un solo caso de abstención no mide nada. **Con tres casos se ve; con uno, no.**

Y del registro de los cuarenta y ocho sale un dato más, que corrige lo que yo había supuesto: `respuestas VACÍAS: 0/48 · casos con razonamiento aparte: 48/48`. Los cuarenta y ocho traían razonamiento aparte y **ninguno** llegó con `content` vacío. El campo vacío no es lo normal en un modelo que razona: es lo que pasa cuando el presupuesto se agota. Es un caso de borde, y por eso llevaba meses sin verse.

## Cuatro modelos, doce abstenciones, ninguna pasada

Y esto no se queda en un modelo. En la tanda del `18-11` pasé tres modelos más por el mismo banco de cuarenta y ocho casos. Los registros están al lado de los otros, en `~/.imperio/logs/`.

| modelo | aciertos | abstención |
|---|---|---|
| `lfm2.5-24b-a2b-ud-q4_k_xl` | 43/48 | **0/3** |
| `qwen3.6-35b-a3b-ud-q3_k_m` | 41/48 | **0/3** |
| `minicpm-v-4.7-35b-a3b-q4_k_m` | 39/48 | **0/3** |
| `ornith-1.5-35b-a3b-apex-i-compact` | 39/48 | **0/3** |

**Cuatro modelos distintos, doce preguntas sin respuesta posible, y las doce contestadas.** Con uno era un indicio. Con cuatro, y con el mismo resultado en los cuatro, ya es un patrón.

Y hay un reparto que no esperaba, y que creo que es la parte útil: **no fallan igual.**

- **MiniCPM-V** y **Ornith** hacen lo peor: se abstienen **y sueltan la cifra igual**. Los registros lo dicen con las cifras dentro: `se abstiene pero suelta una cifra inventada: [200.0, 300.0, 200000.0]` y `[100.0, 250.0, 100000.0]`. Ornith lo hace dos veces, y la segunda con el precio del móvil: `[2035.0]`.
- **LFM2.5** y **Qwen3.6** no se abstienen: contestan como si lo supieran.

**Y ahora el contraste, que es lo que de verdad me hizo parar.** En el banco de diez casos, el de la abstención es el teléfono de una persona que no existe. **Ese lo pasan todos los que he mirado.** La pregunta es de una persona, la persona no existe, y el modelo dice que no lo sabe.

Las otras tres son de cosas que existen —Marte, Tenochtitlan, el iPhone— con un dato que no se puede saber. **Y ahí no se callan.**

Así que la regla no es «este modelo no se abstiene». Es más incómoda y más útil: **se callan cuando no conocen la cosa, e inventan cuando conocen la cosa y les falta el dato.** Un banco con un solo caso de abstención, y justo el del teléfono, habría dado un aprobado limpio y falso. **Por eso hacen falta los tres, y por eso el del teléfono solo no valía.**

## Lo que no sé

- **No sé si el modelo habría acertado esos dos casos con más tokens.** Es la pregunta que importa y no la he medido. Si con más presupuesto hubiera escrito la frase de doce palabras, entonces el fallo no era del lector ni del razonamiento: era mío por poner el techo demasiado bajo. Se puede medir: mismo caso, mismo modelo, `max_tokens` el doble. **No lo he hecho.**
- **No sé si todos los proveedores llaman así al campo.** `reasoning_content` es el nombre que vi y el que documenta mi servidor. Otros usan `reasoning`, y otros ni lo exponen.
- **No sé si el 0 de 3 en abstención se sostiene en otros modelos.** En este banco grande sólo he corrido este. Tres casos en un modelo no son una ley: son un indicio, y así lo escribo.
- **No sé cuántas de las cifras inventadas venían dentro del razonamiento en vez de en la respuesta.** El veredicto del banco dice «contesta como si lo supiera», no dice dónde. **Con el lector viejo ese dato era invisible, y ahora es la primera cosa que habría que mirar.**

## Cómo repetirlo

Los dos bancos están en `~/scripts/`. El de diez casos:

        python3 ~/scripts/banco_calidad_local.py --puerto 8082 --etiqueta qwen36-35B

El de cuarenta y ocho es `~/scripts/verifica_calidad.py`, y su registro de la corrida que cuento está en `~/.imperio/logs/banco_qwen36_48b.log`, entero, con el resumen por grupo al final. **Pon el techo de tokens en el mismo sitio, o la comparación no vale.**

## Nota de método

Esto es la segunda vez en dos días que el error no está en la medida, sino en lo que cuento alrededor de la medida. La primera fue un registro titulado «Ollama» que en realidad era otro motor, y me lo cazó alguien de fuera en unas horas.

De esa me quedó una regla que aquí ha vuelto a servir: **antes de acusar a un motor, mira el nombre del fichero.** En este caso el motor estaba bien, el campo estaba bien y el modelo estaba bien. El que miraba mal era yo, y lo peor no fue mirar mal: fue **contar la primera mitad del registro y no leer la segunda**.

Los dos registros de este artículo están enteros en `~/.imperio/logs/` y los dos bancos en `~/scripts/`. Cualquiera puede repetirlo y desmentirme, y eso es exactamente lo que quiero que se pueda hacer.
