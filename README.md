# VpC1-TP3
Trabajo Práctico para la materia Visión por Computadora 1 de la Carrera de Especialización en Inteligencia Artificial (CEIA)

## Punto 1: Detección única con Template Matching

### Objetivo

El objetivo del pipeline es detectar la mejor coincidencia única de un patrón o *template* dentro de un conjunto de imágenes, utilizando la técnica de Template Matching de OpenCV, y visualizar el resultado sobre cada imagen.

### Transformaciones aplicadas

#### Redimensionamiento del template
Antes de aplicar el algoritmo de coincidencia, se redimensiona el template a un tamaño específico definido por la entrada `params`. Esto permite adaptar el patrón a las proporciones del objeto a detectar en cada imagen, mejorando la precisión del matching.

#### Inversión del template (opcional)
Si `params[2] == 1`, se invierte el template aplicando `255 - template`. Esto es útil para detectar versiones del objeto en negativo o con contraste invertido (por ejemplo, blanco sobre negro en lugar de negro sobre blanco).

#### Selección del canal rojo
Se realiza la búsqueda de coincidencia únicamente sobre el canal rojo (`img[:, :, 2]`). Esta decisión se basa en que dicho canal puede ofrecer mayor contraste o definición del objeto buscado, según el contexto de las imágenes.

### Preprocesamiento y heurística

Para cada imagen del conjunto:

- Se carga y convierte a RGB para visualización.
- Se obtienen parámetros personalizados (ancho, alto, inversión) desde un diccionario `dict_template_size`.
- Se aplica la función `matching`, que:
  - Calcula el mapa de similitud (`res`).
  - Dibuja el rectángulo de detección sobre la imagen.
  - Muestra el score máximo (`max_val`) como indicador de calidad de coincidencia.

### Métrica utilizada para detección

Se utiliza `cv.TM_CCOEFF_NORMED`. Las razones de su elección:

- Invariante a cambios de brillo, ya que elimina la media local del template y de la región evaluada.
- Normalizada, lo que facilita comparar scores entre imágenes.
- Adecuada para detecciones precisas cuando el patrón tiene buena visibilidad.

---

## Punto 2: Detección múltiple con Template Matching

### Objetivo

El objetivo de esta función es detectar múltiples apariciones de un patrón dentro de una misma imagen. En lugar de buscar una única coincidencia, se identifican todas las ubicaciones que superan cierto umbral de similitud.

### Transformaciones aplicadas

- Redimensionamiento del template: Se ajusta a un tamaño personalizado mediante `params`, permitiendo adaptarse a diferentes escalas del objeto.
- Inversión del template (opcional): Si se activa, se invierte para detectar versiones negativas del patrón.
- Selección del canal rojo: Se trabaja exclusivamente sobre el canal rojo por su mejor contraste en este problema.

### Métrica utilizada para detección

Se usa `cv.TM_CCOEFF_NORMED`, que:

- Evalúa la correlación entre la media local del template y la región de la imagen.
- Devuelve una matriz con valores entre -1 (anticorrelación perfecta) y 1 (correlación perfecta).
- Es robusta ante variaciones de iluminación.

### Criterio de detección

- Se calcula el valor máximo (`max_val`) en el mapa de similitud.
- Se define un umbral relativo como `threshold = max_val * 0.83`, valor empírico que balancea entre sensibilidad y precisión:
  - Umbral bajo: más coincidencias, pero riesgo de falsos positivos.
  - Umbral alto: menos falsos positivos, pero se pueden perder detecciones válidas.
- Se consideran válidas todas las ubicaciones donde `res >= threshold`.

---

## Punto 3: Heurísticas avanzadas, fallback y limitaciones

### Estrategia combinada

- Se utiliza SIFT para detecciones únicas con mayor robustez geométrica.
- Si el score de SIFT está por debajo de cierto umbral o falla la detección, se aplica un fallback a detección múltiple con template matching.
- Esta heurística permite cubrir ambos escenarios: patrones bien definidos (SIFT) y repetitivos o borrosos (template matching).

### Limitaciones observadas

- El score de template matching no siempre es representativo de una buena detección:
  - En casos con muchas detecciones incorrectas, el score puede seguir siendo alto.
- Para mitigar esto, se define un límite máximo de detecciones por imagen como guardarraíl ante este comportamiento errático.

### Sobre la escala del template

- Cuando el template se hace demasiado pequeño (en iteraciones de resize), comienza a correlacionar con fondos blancos u otras regiones irrelevantes, generando falsos positivos.
- Por esta razón, se limita el escalamiento inferior del template a un mínimo razonable.

### Pirámide de escalas vs. resize manual

- Se evaluó el uso de piramidado con `cv.pyrDown`, pero su escalamiento discreto (por mitades) no permitía alcanzar tamaños óptimos del template.
- Se optó por un resizing manual continuo que da mejor control sobre los tamaños a evaluar y mejora los resultados.