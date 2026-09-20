# 📘 Práctica de Tokenización de Palabras (PLN con NLTK)

> **Revisión técnica con enfoque de mentoría.**
> Este documento registra, de forma honesta y verificable, los errores cometidos en la primera versión de la práctica y las áreas de mejora identificadas. Sirve como recordatorio y evaluacion personal y como material de aprendizaje para quien lea este repositorio.

**Última revisión:** 20 de septiembre de 2026
**Entorno:** Python 3.13 · Jupyter Notebook en VS Code · NLTK
**Archivo revisado:** `PracticaTokenizacion.ipynb`

---

## 🎯 Objetivo de la práctica

Procesar un texto en español aplicando un flujo básico de Procesamiento de Lenguaje Natural (PLN):

1. **Tokenizar** el texto (dividirlo en palabras).
2. **Limpiar** los tokens (quitar *stopwords* y signos de puntuación).
3. **Contar frecuencias** y mostrar las palabras más comunes y el tamaño del vocabulario.

---

## 📋 Resumen de hallazgos

| # | Error | Tipo | Consecuencia | Gravedad |
|---|-------|------|--------------|----------|
| 1 | Bloque `if __name__ == "__main__":` dentro de la función, después del `return` | Estructura / indentación | El código de prueba **nunca se ejecutaba**; la celda no mostraba salida | 🔴 Alta |
| 2 | `return none` (minúscula) | `NameError` | Falla si ocurre un error de recursos de NLTK | 🔴 Alta |
| 3 | `stop_word` en lugar de `stop_words` | `NameError` | La función se detiene al limpiar los tokens | 🔴 Alta |
| 4 | `resultados['frecuencia']` en lugar de `'frecuencias'` | `KeyError` | Falla al imprimir el vocabulario final | 🔴 Alta |
| 5 | `most_common(1200)` con clave llamada `top_10` | Lógica / coherencia | Resultado no coincide con lo que promete el código | 🟠 Media |
| 6 | `nltk.download()` sin argumentos dentro de un mensaje de error | Diseño | Abre una ventana gráfica y no aporta al mensaje | 🟠 Media |
| 7 | `import nltk` duplicado, `import string` sin uso, descargas mezcladas con imports | Estilo (PEP 8) | Código menos limpio y más difícil de mantener | 🟡 Baja |

> 💡 **Lección clave:** el error #1 *ocultaba* a los errores #3 y #4. Como el bloque de prueba nunca corría, esos fallos estaban latentes y no se veían. **Un error de estructura puede esconder otros errores.**

---

## 🔍 Detalle de cada error

### 1. Bloque principal dentro de la función (error de indentación)

**Qué pasó:** el bloque `if __name__ == "__main__":` quedó indentado *dentro* de `procesar_texto_pln`, justo después del `return`. En Python, una función termina su ejecución al llegar a `return`, por lo que las líneas que quedan después no se ejecutan. Además, el bloque estaba definido dentro de la función, no a nivel del archivo.

```python
# ❌ Incorrecto
def procesar_texto_pln(texto, idioma='spanish'):
    ...
    return {...}

    if __name__ == "__main__":      # <- indentado dentro de la función
        resultados = procesar_texto_pln(texto_ejemplo)
```

```python
# ✅ Correcto
def procesar_texto_pln(texto, idioma='spanish'):
    ...
    return {...}

# Fuera de la función, al nivel del archivo (o en otra celda del notebook)
texto_ejemplo = "..."
resultados = procesar_texto_pln(texto_ejemplo)
```

**Nota:** en un notebook, `__name__` vale `'__main__'` porque el código se ejecuta en el entorno de nivel superior, por lo que la condición `if __name__ == "__main__":` no es necesaria. Se usa sobre todo en archivos `.py` que también pueden importarse como módulo.

**Fuentes:** [Python — `__main__` (Top-level code environment)](https://docs.python.org/3/library/__main__.html) · [Python — Simple statements (`return`)](https://docs.python.org/3/reference/simple_stmts.html)

---

### 2. `none` en minúscula

**Qué pasó:** en Python el valor nulo se escribe `None`, con mayúscula inicial. `none` es un nombre no definido. Este error solo se activa si la tokenización falla (rama `except`), por eso pudo pasar desapercibido.

```python
# ❌ Incorrecto
return none

# ✅ Correcto
return None
```

**Fuentes:** [Python — Built-in Constants (`None`)](https://docs.python.org/3/library/constants.html) · [Python — Built-in Exceptions (`NameError`)](https://docs.python.org/3/library/exceptions.html)

---

### 3. Variable mal escrita: `stop_word` vs `stop_words`

**Qué pasó:** la variable se definió como `stop_words` (plural) pero se usó como `stop_word` (singular). Python distingue cada nombre exactamente, y lanza `NameError` cuando no encuentra un nombre local o global.

```python
# ❌ Incorrecto
stop_words = set(stopwords.words(idioma))
tokens_limpios = [t for t in tokens if t.isalnum() and t not in stop_word]

# ✅ Correcto
stop_words = set(stopwords.words(idioma))
tokens_limpios = [t for t in tokens if t.isalnum() and t not in stop_words]
```

**Fuente:** [Python — Built-in Exceptions (`NameError`)](https://docs.python.org/3/library/exceptions.html)

---

### 4. Clave incorrecta del diccionario: `'frecuencia'` vs `'frecuencias'`

**Qué pasó:** la función retorna un diccionario con la clave `"frecuencias"`, pero se consultó `'frecuencia'`. Acceder a una clave inexistente lanza `KeyError`.

```python
# ❌ Incorrecto
len(resultados['frecuencia'])

# ✅ Correcto
len(resultados['frecuencias'])
```

**Fuente:** [Python — Built-in Exceptions (`KeyError`)](https://docs.python.org/3/library/exceptions.html)

---

### 5. Nombre y valor que no coinciden: `top_10` con `most_common(1200)`

**Qué pasó:** la clave del resultado se llama `top_10` y el mensaje dice "Top 10", pero se solicitaban 1200 palabras. `Counter.most_common(n)` devuelve los `n` elementos más frecuentes.

```python
# ❌ Incorrecto
palabras_comunes = frecuencia_palabras.most_common(1200)

# ✅ Correcto (y mejor aún: parametrizarlo)
def procesar_texto_pln(texto, idioma='spanish', top_n=10):
    ...
    palabras_comunes = frecuencia_palabras.most_common(top_n)
```

**Fuente:** [Python — `collections.Counter.most_common`](https://docs.python.org/3/library/collections.html#collections.Counter.most_common)

---

### 6. `nltk.download()` sin argumentos dentro del mensaje de error

**Qué pasó:** llamar a `nltk.download()` sin indicar un paquete abre el descargador gráfico de NLTK. Colocarlo dentro de un `print` ejecuta la descarga como efecto secundario y mezcla un mensaje informativo con una acción.

```python
# ❌ Incorrecto
print('❌ Error: ... Ejecuta: ', nltk.download(), '.')

# ✅ Correcto
print("❌ Error: recursos de NLTK no encontrados. Ejecuta nltk.download('punkt_tab').")
```

La documentación de NLTK indica que los datos se instalan con el descargador y que se pueden bajar paquetes individuales. Además, `word_tokenize` requiere que los modelos Punkt estén instalados.

**Fuentes:** [NLTK — Installing NLTK Data](https://www.nltk.org/data.html) · [NLTK — `word_tokenize`](https://www.nltk.org/api/nltk.tokenize.word_tokenize.html) · [NLTK — paquete `tokenize`](https://www.nltk.org/api/nltk.tokenize)

---

### 7. Orden y limpieza de imports

**Qué pasó:** `import nltk` aparecía dos veces, `import string` no se usaba y las descargas de recursos estaban intercaladas entre los imports. PEP 8 recomienda imports al inicio del archivo, uno por línea y agrupados.

```python
# ✅ Correcto
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from collections import Counter

nltk.download('punkt')
nltk.download('punkt_tab')
nltk.download('stopwords')
```

**Fuente:** [PEP 8 — Style Guide for Python Code (sección *Imports*)](https://peps.python.org/pep-0008/)

---

## 🧭 Áreas de mejora (plan de estudio)

| Área | Debilidad observada | Cómo trabajarla |
|------|---------------------|-----------------|
| **Estructura del código** | Confundir el nivel de indentación de un bloque | Repasar cómo Python delimita bloques y qué ocurre después de un `return` |
| **Lectura de errores (tracebacks)** | Los errores latentes no se detectaron a tiempo | Ejecutar siempre el flujo completo y leer el mensaje de error de abajo hacia arriba |
| **Consistencia de nombres** | Variables y claves escritas de forma distinta (`stop_word`, `frecuencia`) | Usar el autocompletado del editor y un linter que detecte nombres no definidos |
| **Coherencia lógica** | Nombre (`top_10`) y valor (`1200`) contradictorios | Usar parámetros con nombre y evitar "números mágicos" |
| **Gestión de recursos externos** | Manejo poco claro de las descargas de NLTK | Leer la documentación oficial y descargar paquetes específicos |
| **Estilo y buenas prácticas** | Imports desordenados y sin uso | Seguir PEP 8 y revisar el código antes de subirlo |

---

## ✅ Checklist antes de subir una práctica

- [ ] Reinicié el kernel y ejecuté **todas** las celdas en orden (*Restart* + *Run All*).
- [ ] El notebook muestra las salidas esperadas.
- [ ] No hay variables ni claves con nombres distintos a los definidos.
- [ ] Los nombres coinciden con el comportamiento (`top_10` devuelve 10).
- [ ] No hay imports duplicados ni sin uso.
- [ ] Los recursos externos (NLTK) se descargan por nombre, sin ventanas emergentes.
- [ ] El código de prueba está fuera de la función y se ejecuta.

---

## ▶️ Cómo ejecutar el notebook en VS Code

1. Instalar las extensiones **Python** y **Jupyter** de Microsoft.
2. Abrir el archivo `.ipynb` y elegir el intérprete con **Select Kernel** (arriba a la derecha).
3. Instalar la librería si hace falta: `pip install nltk`.
4. Ejecutar las celdas en orden con `Shift + Enter` o usar **Run All**.

---

## 📚 Fuentes consultadas

| Tema | Fuente oficial |
|------|----------------|
| Entorno de nivel superior y `__name__` | <https://docs.python.org/3/library/__main__.html> |
| Sentencia `return` | <https://docs.python.org/3/reference/simple_stmts.html> |
| Excepciones (`NameError`, `KeyError`) | <https://docs.python.org/3/library/exceptions.html> |
| Constante `None` | <https://docs.python.org/3/library/constants.html> |
| `Counter.most_common` | <https://docs.python.org/3/library/collections.html#collections.Counter.most_common> |
| Guía de estilo PEP 8 | <https://peps.python.org/pep-0008/> |
| NLTK — instalación de datos | <https://www.nltk.org/data.html> |
| NLTK — `word_tokenize` | <https://www.nltk.org/api/nltk.tokenize.word_tokenize.html> |
| NLTK — módulo `tokenize` | <https://www.nltk.org/api/nltk.tokenize> |

---

## 🤝 Nota sobre la autoría de esta revisión

Esta revisión fue elaborada con apoyo de una herramienta de inteligencia artificial (Claude, de Anthropic) actuando como mentor técnico, a partir del análisis del notebook original. Las referencias enlazadas son documentación oficial y pueden consultarse para verificar cada afirmación.

*Errar es parte del proceso: documentarlo es lo que convierte el error en aprendizaje.* ✨
