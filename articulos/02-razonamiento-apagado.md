titulo: Apagar el razonamiento cuesta 12 aciertos, no te hace más rápido, y nadie te avisa
fecha: 2026-11-17
resumen: El mismo modelo, el mismo banco, la misma máquina. Con razonamiento 45/48. Sin razonamiento 33/48. Y encima más lento.

Hay una costumbre muy extendida: si un modelo razona y tarda, se le apaga el razonamiento para que vaya más rápido.

**Lo medí. Sale peor en las dos cosas.**

## El experimento

Un modelo denso de 27B, cuantizado a 4 bits, servido por `llama.cpp` con la misma ventana de contexto y el mismo banco de pruebas. **La única diferencia entre las dos filas que importan es una bandera.** Con ella, el modelo piensa antes de contestar; sin ella, contesta directo.

El banco son 48 casos, repartidos en tres niveles de dificultad y varias familias: cuentas, extracción de datos, instrucciones de formato, abstención («si no lo sabes, dilo»), y tres casos de razonamiento largo.

| cómo se sirvió | aciertos | vacías | ¿pensó? | velocidad |
|---|---|---|---|---|
| **con razonamiento** | **45 / 48** | 0 | 48/48 | 5,9 tok/s |
| **sin razonamiento** | **33 / 48** | 0 | 0/48 | 4,3 tok/s |

**Doce aciertos.** De cada 48 preguntas, doce pasan de bien a mal por una bandera.

## Y aquí está la parte que no esperaba

**Quitando el razonamiento, el modelo fue MÁS LENTO.** 5,9 contra 4,3 tokens por segundo. El doble de malo: **contesta peor y tarda más**.

No es un artefacto de dos carreras sueltas: es el mismo servidor, el mismo modelo y el mismo banco, con las velocidades salidas del propio contador de tokens de cada respuesta.

**Si alguien te dice «apaga el razonamiento y así va rápido», pídele los números.** En esta máquina, apagarlo no compró velocidad y costó doce aciertos.

## Por grupos: dónde duele

Con el razonamiento apagado no se pierde por igual en todo:

| familia | con razonamiento |
|---|---|
| cuentas | **14 / 14** |
| formato | **8 / 8** |
| instrucción | **8 / 8** |
| razonamiento largo | **5 / 5** |
| código | 3 / 3 |
| lengua | 3 / 4 |
| abstención | 1 / 3 |

La caída se concentra en los dos niveles altos, y sobre todo en la **abstención**: cuando no se le deja pensar, el modelo deja de reconocer que no sabe y **contesta con una cifra inventada**. Eso es peor que un fallo: un fallo se ve, una cifra inventada se cuela.

## La segunda sorpresa: la plantilla vale 9 aciertos

El mismo modelo, **el mismo modo sin razonamiento**, servido por dos servidores distintos:

| servidor | aciertos sin razonar |
|---|---|
| uno, con su plantilla de chat | 42 / 48 |
| otro, con la plantilla del fichero GGUF | **33 / 48** |

**Nueve aciertos de diferencia por cómo se formatea la conversación.** No por el modelo, no por las tarjetas: por la plantilla. Y esto es todavía más silencioso que el razonamiento, porque la plantilla no se ve en ningún sitio mientras el modelo contesta.

## Lo que no sé

- **No he aislado** cuánto de la diferencia viene de cada parte de la plantilla. Sé que cambia el resultado; no sé qué línea lo cambia.
- **El banco es mío**, 48 casos elegidos para las cosas que yo necesito de un modelo local: no es un benchmark académico y no pretende serlo. Los casos están escritos y se pueden leer.
- **Una sola máquina.** Dos tarjetas de 16 GB, 62 GB de RAM, Linux. Con otros modelos y otros tamaños puede ser distinto. Por eso el compromiso es dejar el número y el comando.

## Qué hago con esto

1. **No apago el razonamiento por defecto.** Si necesito velocidad, la busco en otra parte: menos contexto, un modelo más pequeño, menos tokens de salida.
2. **Cuando comparo dos servidores, mido primero la plantilla.** Si no la controlo, no estoy comparando modelos: estoy comparando andamios.
3. **Y compruebo la abstención aparte.** Es lo primero que se cae al apagar el razonamiento, y es lo que menos se nota.

---

**Nota de método.** Las dos filas de la primera tabla salen del mismo binario, el mismo día, con el mismo banco y el mismo fichero de resultados, uno al lado del otro. La velocidad es la del contador de tokens de cada respuesta, sumada sobre los 48 casos. La máquina: dos tarjetas de 16 GB, 62 GB de RAM, Linux. Si repites esto y te sale otra cosa, el número manda: el mío no es sagrado.
