titulo: Hubo un hilo, y nos corrigieron dos veces
fecha: 2026-11-19
resumen: El artículo anterior cuenta lo que medimos. Este cuenta cómo llegamos a medirlo: un issue que abrimos el 18-11 convencidos de un fallo, un colaborador de Ollama que nos contestó dos veces, y una corrección nuestra que nos hicimos nosotros mismos al día siguiente. Incluye la cifra que retiramos y las dos preguntas que siguen sin respuesta.
tema: foro

El artículo anterior de este taller cuenta **lo que medimos**. Este cuenta **cómo llegamos a medirlo**, porque el camino tiene dos correcciones y las dos son nuestras: una que nos hizo un colaborador de Ollama y otra que nos hicimos nosotros al día siguiente, solos, al repetir la prueba con más cuidado.

Va con nombres, horas y enlaces, porque lo que no se puede comprobar no vale nada.

## Lo que creíamos el 18 de noviembre

Abrimos el issue [`#19765`](https://github.com/ollama/ollama/issues/19765) a las **19:58 del 18-11** *(hora de GitHub, que va en UTC)* con un titular claro: Ollama se comía media ventana de contexto **en silencio**. Teníamos la medida repetida en cuatro combinaciones de versión y configuración, todas con el mismo número, y ninguna con un aviso.

Y nos contestaron. Primero que el modelo que habíamos usado no lo soportaba la versión `0.34.1` — con el error `unknown model architecture: 'flint'` delante. Después, y esto es lo que importa, con dos ejecuciones que nos dejaban en evidencia:

```
{"content": "The secret word is **MARMOLINA**.", "prompt_eval_count": 140658, ...}
{"error": "request (175165 tokens) exceeds the available context size (147456 tokens), try increasing it"}
```

Un prompt de **140.658 tokens** leído entero, con la palabra escondida en la cabecera y encontrada. Y un error explícito al pasarse. Es decir: **en la versión nueva, y por la puerta nativa, no había truncado silencioso.** Se lo agradecimos y cerramos el issue nosotros mismos, **51 minutos después de abrirlo**.

## Lo que medimos el 19 por la mañana

Instalamos la `0.40.16` al lado de la `0.34.1` y medimos las dos con el mismo documento, el mismo modelo y la misma máquina. Y salió esto:

| versión | `num_ctx` | tokens leídos | ¿avisó? |
|---|---|---:|---|
| 0.34.1 | por defecto | 16.386 | no |
| 0.34.1 | 32.768 | 16.386 | no |
| 0.40.16 | por defecto | 16.386 | no |
| 0.40.16 | 32.768 | 16.386 | no |

Cuatro combinaciones, un solo número, y ni un aviso en la puerta `/api/chat`. Doblar el documento a 600.000 caracteres no movió la cifra **ni un token**. En la puerta `/v1/responses` sí había diferencia —la `0.34.1` devolvía `status: completed` con el contenido vacío, y la `0.40.16` daba el error de tamaño—, y eso también lo escribimos.

Y ahí fue donde nos equivocamos de diagnóstico **por segunda vez**, aunque esta vez el error era de los que se ven al repetir: dijimos que **el presupuesto del prompt era media ventana**.

## La corrección que nos hicimos nosotros

Al día siguiente, midiendo de nuevo para la última respuesta del hilo, apareció el campo que lo explicaba todo: `/api/chat` acepta **`truncate: false`**. Con ese campo puesto, el motor **no trunca y dice la verdad**, con el límite en la ventana exacta:

- `num_ctx=4096` y un prompt de **28.737 tokens** → error explícito: `request (28737 tokens) exceeds the available context size (4096 tokens)`.
- `num_ctx=32768` con el mismo texto → **entra entero**, `prompt_eval_count = 28.737`.

⇒ **La media ventana no era el techo: era lo que deja el truncado.** Corta a la mitad y deja la otra mitad sin usar. Y por eso, si usas `/api/chat` con los valores por defecto, la forma de que te avise existe: pon `truncate: false` y tendrás un error en vez de un silencio.

La respuesta correcta la dejamos en el hilo el 19-11 a las **16:00**, [`#issuecomment-7014488289`](https://github.com/ollama/ollama/issues/19765#issuecomment-7014488289): son seis comentarios en total, el issue sigue cerrado y ninguno de los datos que publicamos antes se ha retirado salvo la cifra que había que retirar.

## Y las dos preguntas que siguen abiertas

Porque no todo está resuelto, y decirlo es parte del trabajo:

1. **¿De qué depende la reserva de la mitad?** Lo intentamos moviendo `num_predict` y el modelo abortó repitiendo antes de darnos un número. **No medido.**
2. **¿De dónde salen los dos tokens de más?** `2.050` sobre `2.048`, `4.098` sobre `4.096`. Probablemente la plantilla del modelo. **No comprobado.**

Si alguien lo mide, que lo cuente: el número ya tiene dónde vivir.

## Qué se saca en limpio

- **Si usas `/api/chat` y tus respuestas salen raras con documentos largos, no es tu modelo.** Pon `truncate: false` y mira el error.
- **Un issue abierto con prisa se cierra con prisa.** Nos pasó: lo cerramos nosotros en 51 minutos, y la respuesta buena llegó al día siguiente.
- **Y la corrección más incómoda es la que te haces tú**, con el mismo banco y el mismo texto, sin que nadie te lo pida. La cifra retirada era nuestra, y está retirada aquí y en el artículo anterior.
