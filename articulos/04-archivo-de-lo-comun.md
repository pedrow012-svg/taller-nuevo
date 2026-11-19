titulo: El archivo de lo común existe. Y el modelo distingue mucho peor un eccema que un tumor
fecha: 2026-11-17
resumen: 3.061 casos con diagnóstico de dermatólogo, de enfermedades comunes. Medidos con el mismo motor de nuestro visor: 42 % de acierto al primer parecido, frente al 77 % en lesiones tumorales.

Tenemos un visor que, cuando le entras una foto de una lesión de piel, te enseña los casos
más parecidos de un archivo y **no diagnostica nada**: compara parecido de imagen y deja el
juicio a quien mira. Funciona porque detrás hay un archivo con biopsia: **2.106 fotos de seis
tipos de lesión** (melanoma, basalioma, queratosis actínica, nevus, queratosis seborreica y
carcinoma escamoso), licencia `CC BY 4.0`.

El agujero era evidente para cualquiera que lo use de verdad: **son seis cosas, y todas son
tumor, precáncer o nevus.** No hay un solo eccema. No hay un hongo. No hay rosácea. Si un
médico de cabecera le entra una dermatitis, el visor le devuelve un nombre de cáncer con un
número bajo al lado. Y el aviso de prudencia que le pusimos no arregla el agujero: lo señala.

Así que la pregunta era si existe un archivo de **lo común** —y con licencia que permita
usarlo—, y, sobre todo, **si la técnica aguanta cuando las enfermedades se parecen entre sí**.

## El archivo: SCIN

**Skin Condition Image Network**, de Google Research (2024). Fotos de móvil donadas por
usuarios de Estados Unidos con consentimiento informado, y etiqueta de dermatólogo para cada
caso. Este es el recuento, hecho por nosotros sobre su propia tabla de etiquetas:

| | |
|---|---|
| casos en total | **5.033** |
| **con diagnóstico de dermatólogo** | **3.061** |
| sin diagnóstico (campo vacío) | 1.972 |
| diagnósticos distintos | **370** |

Y de las enfermedades comunes, cuántos casos las mencionan (y cuántos las tienen como
diagnóstico principal):

| enfermedad | casos | principal |
|---|---|---|
| eccema / piel atópica | 1.056 | 488 |
| dermatitis de contacto alérgica | 873 | 270 |
| picaduras | 403 | 185 |
| urticaria | 332 | 214 |
| psoriasis | 308 | 109 |
| foliculitis | 268 | 142 |
| dermatitis irritativa | 249 | 39 |
| **hongos (tiña)** | 206 | 93 |
| exantema por fármacos | 148 | 58 |
| herpes zóster | 142 | 68 |
| impétigo | 121 | 69 |
| **acné** | 98 | 61 |
| pitiriasis rosada | 80 | 44 |
| escabiosis | 63 | 21 |
| molusco contagioso | 63 | 18 |
| tiña versicolor | 53 | 37 |
| **rosácea** | 52 | 29 |

**La licencia, en plata:** no prohíbe el uso comercial, permite producir y compartir material
transformado —y un vector lo es—, exige decir de dónde viene y prohíbe reidentificar a las
personas. **No es `CC BY`**: es una licencia propia de tres páginas, y antes de vender nada
con ella hay que leerla entera y con abogado. Aquí se dice y no se interpreta de más.

## Cómo se midió

Con **el mismo motor que usa el visor** (un codificador de piel de `HAI-DEF`, `448×448`, en
CPU, sin tarjeta gráfica) y la misma idea: el vector de cada foto, coseno entre todos, y
mirar quién sale primero.

Se eligieron **9 enfermedades comunes** y, de cada una, **49 o 50 casos con etiqueta principal
clara** (peso ≥ 0,5 en la etiqueta combinada de los dermatólogos, para no medir casos
ambiguos). Total: **424 fotos**, una por caso.

```sh
python3 medidas/mide_acierto_scin.py <carpeta_de_fotos> lista.tsv
```

`lista.tsv` lleva tres columnas: `case_id`, diagnóstico y ruta de la imagen. El medidor no
entrena nada, no escribe en el archivo del producto y deja los vectores en caché para poder
repetirlo sin volver a calcularlos.

## Los números

| qué se mide | resultado |
|---|---|
| **el 1º parecido es de la misma enfermedad** | **179 / 424 = 42,2 %** |
| **alguno de los 5 lo es** | **334 / 424 = 78,8 %** |
| azar (decir siempre la más común) | 11,4 % |

Y por enfermedad, que es donde está la información:

| enfermedad | acierto al 1º |
|---|---|
| **rosácea** | **72 %** |
| urticaria | 64 % |
| acné | 52 % |
| hongos (tiña) | 51 % |
| psoriasis | 40 % |
| eccema | 34 % |
| foliculitis | 30 % |
| picaduras | 28 % |
| **dermatitis de contacto alérgica** | **24 %** |

**El contraste con lo que ya teníamos es la noticia:** en las lesiones tumorales del archivo
con biopsia, el primer parecido es de la misma enfermedad el **76,8 %** de las veces y alguno
de los cinco el **97,4 %** (medido sobre las 2.106 fotos, azar 36,8 %).

**⇒ La técnica se transfiere, pero con la mitad de puntería.** Y el orden de la tabla no es
capricho: cuanto más se parecen las enfermedades entre sí a simple vista, peor discrimina el
parecido de imagen.

## Lo incómodo, dicho antes de que lo pregunte nadie

**Lo que peor distingue es la dermatitis de contacto alérgica: 24 %.** Es la clase más común
del archivo (873 casos) y es, a la vez, el pan de cada día de una médica del trabajo.

Hay dos razones, y ninguna es una excusa:

1. **En SCIN no hay biopsia.** La etiqueta es «lo que dice un dermatólogo mirando la foto»,
   no un resultado de laboratorio. Eso hace el suelo más blando que en el archivo de tumores.
2. **El eccema, la dermatitis irritativa y la alérgica se parecen entre sí a propósito**:
   son la misma familia de inflamación con distinta causa. Un comparador de imagen no ve la
   causa; ve la piel.

Una especialista de la mutua lo dijo con más gracia y con más razón: **las lesiones que ella
ve en su trabajo son fáciles de diagnosticar porque las causa un agente identificado.** Es
decir: **justo esa clase, la que peor mide nuestro motor, es la que menos necesita un
comparador.** El dato malo y la necesidad real apuntan al mismo sitio.

## Qué significa esto

- **El archivo de lo común existe y se puede usar.** Eso ya no es una hipótesis.
- **El número honesto es otro según de qué se hable.** No es lo mismo decir «parecidos de
  lesiones con biopsia» (76,8 %) que «parecidos de enfermedades comunes» (42,2 %). Mezclarlos
  en una frase sería mentir con dos verdades.
- **Y el producto tendría que hablar distinto según el grupo**, con su aviso delante. Un
  número bajo al lado de un eccema no es un diagnóstico peor: es un archivo que no está
  hecho para eso todavía.

## La medida completa: 3.060 casos y 211 enfermedades en juego

El piloto dejó una pregunta abierta: **¿cuánto baja el acierto cuando el archivo no tiene nueve
enfermedades, sino todas las que tiene?** Ya está medido, y es la cifra que importa para una
herramienta de verdad: **3.060 fotos, una por caso con diagnóstico, 211 enfermedades distintas.**

| qué se mide | piloto (9 enfermedades) | **completo (211 enfermedades)** |
|---|---|---|
| **el 1º parecido es de la misma enfermedad** | 42,2 % | **19,8 %** |
| **alguno de los 5 lo es** | 78,8 % | **47,8 %** |
| azar (decir siempre la más común) | 11,4 % | 5,1 % |

**La bajada no es un fallo de la medida: es la pregunta.** Con nueve enfermedades entre las que
elegir, acertar es más fácil. Con **211** —casi todas parecidas de dos en dos, y muchas con menos
de cinco casos— el primer puesto se vuelve mucho más duro. **El número honesto para una
herramienta real es el segundo: uno de cada cinco.**

Las enfermedades frecuentes, con su acierto al primer parecido en el corpus completo:

| enfermedad | casos | acierto al 1º |
|---|---|---|
| **rosácea** | 29 | **44,8 %** |
| urticaria | 214 | 38,3 % |
| acné | 61 | 36,1 % |
| tiña versicolor | 37 | 35,1 % |
| eccema | 488 | 29,1 % |
| herpes simple | 55 | 29,1 % |
| exantema por fármacos | 58 | 25,9 % |
| hongos (tiña) | 92 | 25,0 % |
| psoriasis | 109 | 22,0 % |
| herpes zóster | 68 | 22,1 % |
| dermatitis de contacto alérgica | 270 | 21,1 % |
| pitiriasis rosada | 44 | 20,5 % |
| foliculitis | 142 | 18,3 % |
| impétigo | 69 | 17,4 % |
| picaduras | 185 | 16,8 % |

**Y aquí está el dato que cierra una discusión de esta casa, con la boca llena: la familia de la
dermatitis de contacto es, para este comparador, prácticamente invisible.**

| | casos | acierto al 1º |
|---|---|---|
| dermatitis irritativa | 39 | **0 %** |
| dermatitis de contacto (sin especificar) | 36 | **2,8 %** |
| «acute dermatitis, NOS» | 46 | **2,2 %** |

**Cero de treinta y nueve.** No es que acierte poco: es que **el parecido de imagen no distingue
una dermatitis irritativa de todo lo demás.** Y no es una desgracia, es una lección: **una médica
del trabajo dijo que las lesiones que ella ve son fáciles de diagnosticar porque las causa un
agente identificado.** Es decir, **la clase que este motor no ve es justo la que un profesional
resuelve con la historia del paciente y no con una foto.**

**⇒ El comparador no sustituye a esa mirada: sirve para las enfermedades en las que la imagen
lleva la información —urticaria, rosácea, acné, hongos—, y no para las que se resuelven
preguntando.** Saber cuál es cuál es exactamente lo que hace útil la herramienta.

## Las fronteras de esta medida

1. **424 fotos, 9 enfermedades, una sola foto por caso.** Es un piloto, no un censo.
2. **Un motor y una configuración.** No se ha probado a afinar ni a cambiar de codificador.
3. **Fotos de móvil donadas en Estados Unidos**, con los fototipos que allí hay. Fuera de ahí
   vale menos, igual que en el archivo de tumores.
4. **La etiqueta es de dermatólogo mirando la foto**, no una biopsia.
5. **La medida completa se hizo el mismo día** y está arriba: **3.060 fotos, 211 enfermedades,
   19,8 % al primer parecido.** Lo que sigue faltando es **un archivo con biopsia para lo común**
   —en SCIN la etiqueta es de dermatólogo mirando la foto— y **probar otro codificador**, porque
   todo esto está medido con uno solo.

## Los ficheros

- Medidor: `~/Orfeo_Core/medidas/mide_acierto_scin.py`
- Muestra medida y vectores en caché: `scin_piloto/` (425 fotos, 511 MB)
- **Corpus completo medido: `scin_completo/` (3.061 fotos, una por caso, 3,7 GB)**; la salida, en
  `medida_completa.txt`
- Etiquetas y casos: `scin_labels.csv`, `scin_cases.csv` (descargados del cubo público de SCIN)
- Comando: `python3 medidas/mide_acierto_scin.py scin_piloto lista.tsv`
- Fecha de la medida: **17 de noviembre de 2026**
