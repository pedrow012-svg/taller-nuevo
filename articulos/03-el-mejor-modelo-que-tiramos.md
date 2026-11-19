titulo: El modelo que dimos por malo era el mejor. Lo medí con la bandera equivocada
fecha: 2026-11-17
resumen: Le pusimos 36/48 y lo descartamos. Con el razonamiento encendido y otro motor da 45/48 y va al doble de velocidad. Tres veces el mismo error en un día.

Este artículo es una equivocación contada entera, porque las equivocaciones se cuentan mejor que los aciertos.

## El caso

Un modelo grande, en cuantización agresiva de 2 bits (66 GB). Lo probamos y sacó **36 de 48** en nuestro banco. El veredicto que escribí fue: *«solo resucita al que ya perdió»*. **Descartado.**

Ese veredicto tenía dos cosas mal, y ninguna era el modelo.

## Error 1: le apagué el razonamiento

Los modelos que razonan gastan tokens pensando antes de contestar. Se apagan con una bandera para ahorrar tiempo, y es una costumbre muy extendida.

**La medición se hizo con esa bandera puesta.**

El mismo banco, el mismo día, con el razonamiento encendido:

| | aciertos |
|---|---|
| razonamiento **apagado** | 36 / 48 |
| razonamiento **encendido** | **45 / 48** |

**Nueve aciertos por una bandera.** Y no solo eso: al encenderlo, la velocidad **subió** (de 2,7 a 47,4 tokens por segundo), porque el otro motor movía los expertos del modelo entre memoria y tarjeta de una forma mucho peor.

## Error 2: cambié de motor y de bandera a la vez

Y aquí está el error de método, que es el que quiero dejar escrito:

**Cambié dos cosas a la vez y le eché la culpa a una tercera.**

- Cambié de servidor (por necesidades de despliegue).
- Se quedó el razonamiento apagado (por costumbre).
- **Y el veredicto se lo llevó el modelo**, que era lo único que no había cambiado.

Es el error clásico de medir: **dos variables movidas, una conclusión**. Y encima la conclusión sonaba razonable — «es un cuantizado a 2 bits, normal que sea malo» — que es la clase de explicación que uno acepta sin comprobar.

## La tabla completa, con la misma vara

Mismo banco de 48 casos. Misma máquina: dos tarjetas de 16 GB, 62 GB de RAM, Linux.

| motor | aciertos | velocidad | ventana |
|---|---|---|---|
| **el mismo modelo, con razonamiento y buen motor** | **45 / 48** | **47,4 tok/s** | 131.072 |
| un modelo denso de 27B, con razonamiento | 45 / 48 | 5,9 tok/s | 8.192 |
| **el motor que teníamos en producción** | 42 / 48 | 43,6 tok/s | 147.456 |
| el modelo de 27B sin razonamiento | 33 / 48 | 4,3 tok/s | 8.192 |
| el modelo de 2 bits, con la bandera equivocada | 34 / 48 | 2,7 tok/s | 131.072 |

**El que iba a la basura es el que gana.**

## Lo que esto me enseñó, y me costó seis horas

1. **Antes de culpar al modelo, comprueba las banderas.** El razonamiento, la plantilla de chat y el tamaño de contexto cambian el resultado más que el modelo en sí. En este mismo día, esos tres factores me costaron 12, 9 y 9 aciertos respectivamente.
2. **Una variable cada vez.** Si cambias de servidor y de bandera, no sabes cuál de las dos te hundió el número. Y lo que aprendas no sirve para la próxima.
3. **La explicación plausible es la trampa.** «Es un cuantizado a 2 bits» explica el resultado perfectamente… y era falso. **Una explicación que encaja no es una explicación que se ha comprobado.**
4. **Un número mal medido cuesta más que no tener número**, porque el mal medido se cree. Con el 36/48 en la mano estuve a punto de tirar el mejor motor de la casa.

## Lo que sigue en pie

**El banco no estaba mal.** Los 48 casos midieron lo que tenían que medir, con su fichero de resultados y su fecha. **El que estaba mal era yo, eligiendo qué le pasaba al modelo.**

Y por eso el banco sirve: **cuando la bandera cambió, el número cambió, y el fichero de resultados lo demuestra.** Si me hubiera fiado de la impresión, hoy seguiría diciendo que ese modelo es malo.

---

**Nota de método.** Las dos filas del mismo modelo salen del mismo banco, el mismo día, con la misma máquina y con los ficheros de resultados guardados. La diferencia entre ellas es la bandera de razonamiento y el servidor que lo sirve. El banco son 48 casos escritos por nosotros, con los tres niveles y las familias a la vista, y no es un benchmark académico: es lo que necesitamos de un modelo local. Si lo repites y te sale otra cosa, el número manda.
