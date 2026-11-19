# El Taller — escritos de la casa

**Medidas, pruebas y lo que se rompe.** Lo que se publica aquí ha pasado por un banco, tiene su fichero de
resultados y se puede repetir. Lo que no se ha podido medir, se dice.

## Cómo se construye

```sh
python3 construye.py
```

Escribe en `sitio/`: un índice y un HTML por artículo. **Sin dependencias salvo `markdown` de Python**, sin
JavaScript, sin fuentes externas. Si mañana esto se cae, se levanta con un comando.

## Cómo se escribe un artículo

Un fichero `.md` en `articulos/`, con una cabecera mínima:

```
titulo: Lo que has medido, en una frase
fecha: 2026-11-17
resumen: Una línea para el índice.
```

Y luego el texto. **Regla de la casa:** ningún número sin su fichero de resultados, su fecha y su comando.

## Publicar

Construido en local. **Publicar es decisión de Pedro**: aquí solo se forja.
