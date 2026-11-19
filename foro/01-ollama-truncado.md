# Ollama trunca tu documento en silencio, y el modelo te dice que lo ha leído entero

**Le pasé un documento de 300.000 caracteres y otro de 600.000. En los dos casos leyó exactamente 65.538 tokens. Y contestó como si lo hubiera leído todo.**

## Cómo me di cuenta

Le pedí a un modelo que encontrara un dato que yo había puesto **a propósito en la última línea** de un documento largo. Contestó con seguridad, y el dato no estaba en su respuesta. Volví a preguntarle por algo del principio: acertó. Por algo del final: inventó.

Entonces conté los tokens que había leído de verdad.

## El número

| lo que le mandé | lo que leyó |
|---|---|
| 300.000 caracteres | **65.538 tokens** |
| 600.000 caracteres | **65.538 tokens** |

**El mismo número exacto las dos veces.** No es una coincidencia: es un techo. **Ollama corta en 65.536 tokens** —el resto son los tokens de plantilla— **y no dice nada.** Ni un aviso, ni un error, ni una advertencia en el log.

## Por qué esto es peor que un error

Un error se ve. Esto no.

**El modelo no sabe que le han cortado el documento.** Él ve un texto que empieza bien y acaba en mitad de una frase, y hace lo que sabe: contestar. Y como los modelos están entrenados para ser útiles, **rellenan el hueco con lo que probablemente diría el documento** — que es exactamente lo que parece una respuesta correcta.

**El fallo no está en el modelo. Está en que nadie te avisa de que el modelo no ha leído lo que crees que ha leído.**

## Cómo comprobarlo en tu máquina (dos minutos)

1. Coge un documento largo, de más de 100.000 tokens.
2. Pon una frase rara al final, del tipo *«la palabra secreta es MARMOLINA»*.
3. Pregúntale por ella.
4. Mira cuántos tokens dice el `usage` que ha leído.

Si te sale un número redondo y siempre el mismo, ya lo tienes.

## Lo que no sé

- **No sé si pasa en todas las versiones** de ollama. Lo medí en la mía, con un modelo concreto, y no lo he probado con otras.
- **No sé si es intencionado** (un límite de seguridad) o un descuido. La documentación no lo dice donde yo lo busqué.
- **No he mirado si el servidor de OpenAI lo hace igual.** Solo puedo hablar de lo que medí.

## La pregunta

**¿A alguien más le ha pasado?** Me interesa sobre todo si os ha pasado con documentos largos y sin daros cuenta, porque **eso significaría que hay gente tomando decisiones con respuestas construidas sobre un documento que el modelo nunca leyó entero.**

---

*Los números completos, con el fichero de resultados y el comando exacto para repetirlo, están en [El Taller](https://taller.dantehef.dev).*
