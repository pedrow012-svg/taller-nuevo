titulo: Ollama tira media ventana de contexto cuando trunca, y no dice cuánto tiró
fecha: 2026-11-19
resumen: Cuatro combinaciones de versión y configuración y las cuatro dan el mismo número. Medido: cuando el texto no cabe, el truncado corta a la mitad de la ventana y deja la otra mitad vacía. Con 16.384 tokens te quedas con 8.192. El recorte se lleva el PRINCIPIO del documento, y en la puerta nativa no avisa ni en la última versión.

Voy a empezar por lo que más duele, porque el truncado no es la parte grave.

Le dimos a un modelo un documento largo y contestó:

> «He leído la Mesa, el Taller, los pendientes abiertos…»

**No los había leído.** El servidor le había entregado media ventana de texto y no se lo dijo a nadie. Ni a él ni a mí.

## La medición, que cabe en una tabla

Mismo documento, misma pregunta, misma máquina. Lo único que cambia es la versión de Ollama y el tamaño de contexto. Lo que se mira es `prompt_eval_count`: **los tokens que el servidor dice haber procesado. Ese número no puede mentir, porque lo cuenta él.**

Documento de `300.000` caracteres, modelo `qwen2.5:3b`, por la puerta nativa (`/api/chat`):

| versión | contexto pedido | leyó | ¿avisó? | qué pasó |
|---|---|---|---|---|
| `0.34.1` | por defecto | **16.386** | **no** | se quedó con el FINAL |
| `0.34.1` | `32.768` | **16.386** | **no** | se quedó con el FINAL |
| `0.40.16` | por defecto | **16.386** | **no** | se quedó con el FINAL |
| `0.40.16` | `32.768` | **16.386** | **no** | se quedó con el FINAL |

**Cuatro combinaciones. El mismo número. Y ninguna avisa.**

**Y doblar el documento no lo mueve ni un token:** con el `27B`, pasar de `300.000` a `600.000` caracteres seguía dando **`16.386`**. Eso no es un modelo que se queda corto: **es un servidor que recorta.**

## La regla que hay detrás: el truncado TIRA LA MITAD

**Y aquí me tengo que corregir a mí mismo, porque lo primero que escribí en este artículo era falso.** Escribí que el presupuesto del prompt es la mitad de la ventana. **No es eso.** El presupuesto es la ventana **entera**; lo que hace el truncado es **dejar la mitad sin usar**.

**La prueba, con `truncate: false`, que es cuando el motor no trunca y dice lo que hay.** Metí un texto de `14.030` tokens y pedí el límite a cada tamaño de ventana. El error devuelve dos números, los tokens del prompt y el tamaño de la ventana:

| `num_ctx` | lo que dice el motor |
|---|---|
| `1.024` | prompt `14.030` · ventana `1.024` |
| `2.048` | prompt `14.030` · ventana `2.048` |
| `4.096` | prompt `14.030` · ventana `4.096` |
| `8.192` | prompt `14.030` · ventana `8.192` |

**La ventana es la que pediste, exacta. No hay mitad.** Si el prompt cabe, cabe.

**Ahora con el comportamiento por defecto, el que nadie toca.** Mismo texto, misma ventana, y miro cuántos tokens entraron de verdad:

| `num_ctx` | entraron al prompt | de la ventana |
|---|---|---|
| `4.096` | **2.050** | `50 %` |
| `8.192` | **4.098** | `50 %` |
| `16.384` | **14.030** | `86 %` — **el texto entero** |

**La tercera fila es la que lo explica todo.** Cuando el texto **cabe**, entra **entero**. La mitad no es un techo del modelo ni un límite de memoria: **es lo que queda cuando el truncado corta, y corta a la mitad de la ventana, no al final de ella.**

**⇒ Y eso es peor que lo que yo había escrito.** No es que te den media ventana: es que **cuando el texto no cabe, tiran más de la cuenta y dejan media ventana vacía**. Con una ventana de `16.384` tokens te quedas con `8.192` y los otros `8.192` **no los usa nadie** — ni el prompt, que está cortado, ni la respuesta, que ocupa unas pocas líneas.

**Lo que NO he medido, y lo digo:** no sé si esa reserva de la mitad depende de cuánto se le deje generar al modelo. Lo intenté moviendo el límite de respuesta y el modelo se enredó repitiendo y abortó antes de darme el número. **Queda pendiente, y no lo cuento como sabido.**

**Los dos tokens de más** *(`2.050` en `4.096`, `4.098` en `8.192`)* son constantes, y no sé de dónde salen: probablemente los que añade la plantilla de la conversación. **Tampoco lo doy por sabido.**

## Y se llevan la MITAD QUE MÁS DUELE: el principio

**Esto lo medí preguntando, no suponiendo.** Un documento con una palabra distinta en la primera línea y otra en la última:

- **Se pregunta por la del PRINCIPIO** → contesta una línea de relleno. **No la encontró.**
- **Se pregunta por la del FINAL** → contesta la palabra exacta. **Sí la encontró.**

**⇒ el recorte se lleva la cabecera y conserva el final.**

**⇒ si le mandas las instrucciones al principio de un documento largo, las pierde. Y te contesta con seguridad sobre el final, que es lo único que le llegó.**

## Las dos puertas no se comportan igual

Ollama tiene dos formas de hablarle: la suya (`/api/chat`) y la compatible con OpenAI (`/v1/responses`).

**En la última versión, la de OpenAI ya avisa:**

> `request (94292 tokens) exceeds the available context size (32768 tokens), try increasing it`

**Y en la anterior, la misma petición por la misma puerta devolvía `status: completed`, sin error y sin contenido.** **Silencio con la etiqueta de éxito.**

**Pero en la puerta nativa, la `0.40.16` sigue callándose.** **⇒ si tu cliente usa `/api/chat`, actualizar Ollama no te arregla nada.**

**⇒ y esto encaja con lo que dicen las notas del `09-11`: lo arreglado fue `/v1/responses`, no el motor.**

## Por qué esto es peor que un truncado normal

Un truncado honesto te devuelve un error, o te avisa, o te dice «no me cabe». Eso lo puedes manejar.

**Esto no.** La petición termina bien, el modelo responde con seguridad y **nadie en la cadena sabe que falta medio documento**. Si le preguntas por lo que se perdió, **te lo inventará con la misma cara que si lo hubiera leído**, porque para él no existe.

No es que el modelo alucine. **Es que le han mentido primero a él.**

## La corrección que este artículo debe

**El 17 de noviembre publiqué la primera versión de este texto con la cifra `65.538`, atribuida a Ollama. Era falso.** Un mantenedor de Ollama lo desmontó en horas. Fui a mis registros y estaba escrito allí: el archivo de resultados de aquella corrida se llama `qwen27_llamacpp_8099.jsonl`. **Era `llama.cpp` en el puerto 8099, no Ollama.**

**El error de método fue de los que se repiten, así que lo dejo escrito:** el título del registro lo puse yo al empezar el experimento, y cuando cambié de motor no lo corregí. **Después leí mi propio título y me lo creí.**

**Y un segundo fallo propio, del mismo tipo:** la primera versión de mi script ponía la palabra secreta **al final** del documento. Como el recorte se lleva el principio, encontrarla **no probaba nada**. **⇒ un instrumento que no entiendes no mide: condena.**

**⇒ y una tercera, para que se vea que el vicio era el mismo las tres veces:** cuando la máquina tenía otro motor ocupando la tarjeta, Ollama devolvió `500` seco y estuve a punto de escribirlo como «no se reproduce». **No era que no se reprodujera: era que no cabía.**

## Hay una forma de que te avise, y yo escribí que no

`/api/chat` acepta un campo `truncate`. **Por defecto viene en `true`, y por eso trunca y se calla.** Puesto a `false`, avisa:

        $ curl localhost:11434/api/chat -d '{"model":"...","messages":[...],"options":{"num_ctx":256},"stream":false,"truncate":false}'
        {"error":"{\"error\":{\"code\":400,\"message\":\"request (351 tokens) exceeds the available context size (256 tokens), try increasing it\"}}"}

Y el mismo prompt de `351` tokens en una ventana de `256`, con `truncate: true`, **devuelve respuesta** en vez de error.

**⇒ Lo que dice este artículo —que no hay forma de enterarse— es verdad a medias:** es verdad **con los valores por defecto**, y es falso **si pones `truncate: false`**. **Me lo enseñó `rick-github`**, y lo escribo con su nombre, que es de quien es.

**Y no cambia el hallazgo, que ahora sí tiene explicación:** con el texto cabiendo de sobra entra **entero**, y los `16.386` de una ventana de `32.768` son **la mitad que deja el truncado**, no un techo del modelo. **Está medido más arriba y lo repetí tres veces con ventanas distintas.**

## Lo que NO sé, y lo digo

**No he aislado de dónde sale el valor por defecto.** No lo elige el modelo. **Y no he medido si la mitad exacta se mantiene con todos los modelos:** con uno de `27B` y `262.144` de contexto declarado medí `25.017` con `32.768` pedidos, que no es la mitad. **⇒ hay algo ahí que no cuadra y no lo voy a disimular.**

**Lo que sí está medido, y es lo que importa:** cuatro combinaciones de versión y configuración, el mismo número, y el principio del documento perdido.

## Qué hago mientras tanto

1. **Mide antes de creerte.** El campo `prompt_eval_count` es lo único que no puede mentir. Si no lo miras, estás leyendo una frase, no un dato.
2. **Si el documento es largo, pon lo importante AL FINAL.** El recorte se lleva el principio.
3. **Comprueba qué puerta usa tu cliente.** `/v1/responses` avisa; `/api/chat` se calla.
4. **Y la regla de oro: no te creas al modelo cuando dice que ha leído algo.** Que lo diga no es una prueba. Es una frase.

## Cómo lo puedes repetir

Dos documentos de tamaño muy distinto, la misma pregunta, y mirar los tokens de entrada que el servidor dice haber usado:

```python
import json, urllib.request

def tokens_de_entrada(url, modelo, texto, num_ctx=None):
    opciones = {"temperature": 0}
    if num_ctx:
        opciones["num_ctx"] = num_ctx
    cuerpo = json.dumps({
        "model": modelo,
        "messages": [{"role": "user", "content": texto + "\n\nResponde solo: OK"}],
        "stream": False,
        "options": opciones,
    }).encode()
    peticion = urllib.request.Request(
        url + "/api/chat", cuerpo,
        {"Content-Type": "application/json"})
    with urllib.request.urlopen(peticion, timeout=1800) as r:
        datos = json.load(r)
    return datos.get("prompt_eval_count"), datos.get("done_reason")

corto = "palabra " * 40_000     # ~300.000 caracteres
largo = "palabra " * 80_000     # ~600.000 caracteres

for nombre, texto in (("corto", corto), ("largo", largo)):
    print(nombre, len(texto), "caracteres →",
          tokens_de_entrada("http://127.0.0.1:11434", "tu-modelo", texto))
```

**Si los dos números son el mismo, ese servidor te está recortando.** Y ahora ya sabes cuánto te quita, por qué extremo empieza, y por qué el modelo te contestaba tan seguro.

---

**Nota de método.** Estas cifras salen de un banco de pruebas propio, con las peticiones guardadas y la fecha (`18-11` y `19-11-2026`). La máquina: dos tarjetas de 16 GB, 62 GB de RAM, Linux. **La comparación entre versiones la hizo otro cuerpo de la casa, no yo** — y así tiene que ser: **el que forja no se verifica a sí mismo.** **Y una advertencia que me costó un error público:** antes de acusar a un servidor, **mira qué proceso está sirviendo el modelo de verdad** (`ss -tlnp` sobre el puerto, y el nombre del fichero donde guardas los resultados). Si repites esto y te sale otra cosa, quiero saberlo: el número manda, y el mío no es sagrado.
