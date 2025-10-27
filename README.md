# Tarea-1-GoodReads "It´s not you, it´s academia"

Este proyecto presenta un análisis y visualización sobre la bibliografía de la autora bestseller Ali Hazelwood, explorando la relación entre el año de publicación de sus obras y las calificaciones (ratings) que han recibido por parte de los lectores en Goodreads. Ali Hazelwood es conocida por sus comedias románticas contemporáneos ambientados en el mundo académico, particularmente en las áreas STEM (Science, Technology, Engineering, and Mathematics).

# Objetivo del Proyecto

El objetivo principal de esta visualización es crear un gráfico informativo y estéticamente atractivo que permita identificar patrones en las publicaciones y cantiad de calificación de los libros de Ali Hazelwood. A través de este análisis visual, podemos responder a la pregunta de si ¿Existe alguna relación entre el año de publicación y la recepción del público?
Acontinuación se explican paso por paso los códigos utilizados.

# Paso 1: Cargar las librerías necesarias

El primer paso en cualquier proyecto de análisis de datos en R es cargar las librerías que proporcionan las funcionalidades necesarias. Para este proyecto, utilizamos varias librerías especializadas:

```r
library(tidyverse)    # Colección de paquetes para ciencia de datos
library(Goodreader)   # Para obtener datos directamente de Goodreads API
library(ggrepel)      # Para crear etiquetas inteligentes que evitan solapamiento
library(ggthemes)     # Proporciona temas adicionales para personalizar gráficos ´´´

**¿Por qué estas librerías?**

- **tidyverse**: Es el ecosistema más completo para manipulación de datos en R. Incluye `dplyr` para transformaciones, `ggplot2` para visualización, `tidyr` para limpieza de datos, y más.

- **Goodreader**: Esta librería especializada nos permite conectarnos directamente a la base de datos de Goodreads para obtener información actualizada sobre libros, autores y calificaciones. Por medio de la búsqueda, consegui encontrar las funciones y el paquete en el siguiente link https://cran.r-project.org/web/packages/Goodreader/Goodreader.pdf

- **ggrepel**: Soluciona uno de los problemas más comunes en visualizaciones: el solapamiento de etiquetas de texto. Automáticamente ajusta la posición de las etiquetas para que sean legibles.

- **ggthemes**: Ofrece temas prediseñados profesionales que pueden mejorar significativamente la estética de nuestros gráficos.
```

# Paso 2: Obtener los datos de Goodreads

Una vez cargadas las librerías, procedemos a obtener los datos directamente desde Goodreads. Realizamos dos búsquedas independientes para asegurarnos de capturar toda la información relevante:
Importante señalar que la búsqueda se realizó hasta octubre de 2025 y se señalaron 18 libros a pesar de que la autora posee menos. Esto se debe a que la base de datos de Goodreads toma algunas ediciones especiales que solo repetian datos (luego se filtra esta información).
```r
# Primera búsqueda: ordenada por ratings (calificaciones)
Hazelwood_ratings <- search_goodreads( search_term = "Ali Hazelwood", search_in = "author", num_books = 18, sort_by = "ratings") 

# Segunda búsqueda: ordenada por año de publicación
Hazelwood_Published <- search_goodreads( search_term = "Ali Hazelwood", search_in = "author", num_books = 18, sort_by = "published_year")

**Explicación detallada:**

La función `search_goodreads()` es el motor de nuestro análisis. Al realizar dos búsquedas con diferentes criterios de ordenamiento, nos aseguramos de obtener una vista completa de la bibliografía de la autora. El parámetro `num_books = 18` está configurado para capturar hasta 18 obras, cubriendo la producción literaria conocida de Ali Hazelwood hasta la fecha (Explicado anteriormente). La razón de realizar dos búsquedas es que diferentes ordenamientos pueden revelar diferentes conjuntos de datos, especialmente el año de publicación y la cantidad de calificaciones debido a que no se podía hacer un mismo código con ambas búsquedas.
```
## Paso 3: Combinar y consolidar los datos

Con dos datasets diferentes en mano, necesitamos consolidarlos en uno solo para facilitar el análisis:

```r
df_hazelwood <- left_join(
  Hazelwood_Published, 
  Hazelwood_ratings, 
  by = "book_id"
)

**¿Por qué `left_join()`?**

Utilizamos `left_join()` en lugar de otros tipos de joins porque queremos mantener todos los registros de nuestra tabla principal (`Hazelwood_Published`) y agregar la información de ratings donde esté disponible. El campo `book_id` actúa como nuestra clave primaria, asegurando que cada libro se empareje correctamente con sus calificaciones correspondientes, incluso si aparece en diferentes posiciones en cada búsqueda.
```
## Paso 4: Limpiar y seleccionar columnas relevantes

Los datos crudos de Goodreads vienen con muchas columnas que no necesitamos para esta visualización específica. Por ello, seleccionamos y renombramos las 5 filas de datos que buscamos:

```r
df_Ahazelwood <- df_hazelwood %>%
  select(
    book_id,                # Identificador único del libro
    titulo = title.x,       # Título del libro (renombrado)
    autor = author.x,       # Nombre del autor (renombrado)
    ratings,                # Calificación promedio
    published_year          # Año de publicación
  )

Al realizar el join, algunas columnas se duplican y R automáticamente les agrega sufijos `.x` y `.y`. Seleccionamos las versiones `.x` porque provienen de nuestra búsqueda principal. El renombrado a español (`titulo`, `autor`) hace el código más legible y facilita la colaboración con otros desarrolladores de habla hispana.
```
## Paso 5: Filtrar datos incompletos

Filtramos los datos incompletos. En este caso hay ciertos libros que no poseen datos porque todavía no salen publicados hasta la fecha de octubre 2025:
```r
df_Ahazelwood <- df_Ahazelwood |> 
  filter(!is.na(ratings))

**Importancia de este paso:**
Eliminar registros con `NA` (valores faltantes) en la columna `ratings` es crucial porque nuestra visualización depende completamente de esta variable. Intentar graficar valores `NA` resultaría en errores o visualizaciones incorrectas. Este es un paso fundamental en cualquier pipeline de análisis de datos: asegurar la integridad de los datos antes de visualizar.
```
## ✂️ Paso 6: Limpiar los títulos de los libros

Los títulos en Goodreads a menudo incluyen información adicional entre paréntesis (como subtítulos, series, o ediciones). Para una visualización más limpia, removemos esta información:
```r
df_Ahazelwood <- df_Ahazelwood |> 
  mutate(titulo_limpio = str_remove_all(titulo, "\\s*\\(.*?\\)\\s*"))

**Desglosando la expresión regular:**

- `\\s*`: Coincide con cero o más espacios en blanco antes del paréntesis
- `\\(`: Coincide con el paréntesis de apertura (escapado con `\\`)
- `.*?`: Coincide con cualquier carácter, cero o más veces, de forma no-greedy (se detiene en el primer paréntesis de cierre)
- `\\)`: Coincide con el paréntesis de cierre
- `\\s*`: Coincide con cero o más espacios después del paréntesis

El resultado es un título limpio como "The Love Hypothesis" en lugar de "The Love Hypothesis (Love Hypothesis, #1)".
```
## Paso 7: Optimizar el dataset final

Una vez limpiados los títulos, reorganizamos las columnas para quedarnos solo con lo que necesitamos para la visualización:

```r
df_Ahazelwood <- df_Ahazelwood |> 
  select(titulo_limpio, autor, ratings, published_year)
```

## Paso 8: Filtrar libros específicos (si es necesario)

En algunos casos, podemos querer excluir ciertos libros del análisis. Por ejemplo:

```r
df_Ahazelwood <- df_Ahazelwood |> 
  filter("Loathe to Love You" != titulo_limpio)
```
Este caso se debio principalmente porque el libro "Loathe to Love You" está formado por tres historias cortas que aparecen individualmente también. Estos libros son:
Under One Roof, Stuck with You y Below Zero

## Paso 9: Crear dispersión controlada con jitter

Uno de los desafíos más importantes en la visualización de datos es el solapamiento. Cuando múltiples libros se publican en el mismo año o tienen ratings similares, los puntos se superponen en el gráfico, haciendo imposible distinguirlos. Para resolver esto, implementamos una técnica llamada "jitter" o dispersión controlada (información encontrada a través de IA):

```r
set.seed(123)  # Semilla para reproducibilidad

df_HwdCoord <- df_Ahazelwood |> 
  mutate(
    year_jitter = published_year + runif(n(), -0.4, 0.4),    # Dispersión horizontal
    ratings_jitter = ratings + runif(n(), -0.3, 0.3)         # Dispersión vertical
  )
```
**¿Qué hace este código exactamente?**

- `set.seed(123)`: Establece una "semilla" para el generador de números aleatorios. Esto significa que cada vez que ejecutemos el código, obtendremos exactamente la misma dispersión. Es fundamental para la reproducibilidad científica.

- `runif(n(), -0.4, 0.4)`: Genera números aleatorios con distribución uniforme entre -0.4 y 0.4 para cada libro. El rango de ±0.4 años es lo suficientemente pequeño para mantener la precisión del año de publicación, pero lo suficientemente grande para separar visualmente los puntos.

- `runif(n(), -0.3, 0.3)`: Similar a lo anterior, pero para los ratings. Usamos un rango más pequeño (±0.3) porque los ratings son más sensibles y no queremos distorsionar significativamente las calificaciones reales.

Al crear las coordenadas dispersas ANTES de graficar, garantizamos que tanto los puntos como las etiquetas de texto usen exactamente las mismas coordenadas. Esto es crucial para mantener la correspondencia visual entre cada punto y su etiqueta.

## Paso 10: Crear la visualización completa

Finalmente, llegamos a la parte más visual del proyecto: crear el gráfico. Este es el código completo con explicaciones detalladas:

```r
df_HwdCoord |> 
  ggplot(aes(x = year_jitter, y = ratings_jitter, label = titulo_limpio)) +
  
  # Capa 1: Puntos con tamaño variable según rating
  geom_point(aes(size = ratings), color = "#FF6B9D", alpha = 0.7) +
  scale_size_continuous(range = c(5, 35)) +
  
  # Capa 2: Etiquetas inteligentes que evitan solaparse
  geom_text_repel(
    vjust = -0.3,                # Ajuste vertical (-0.3 = ligeramente arriba)
    hjust = 0.5,                 # Ajuste horizontal (0.5 = centrado)
    size = 3,                    # Tamaño de fuente
    color = "#8B4789",           # Color morado para el texto
    max.overlaps = 20,           # Permite hasta 20 solapamientos antes de ocultar
    box.padding = 0.5,           # Espacio invisible alrededor de cada etiqueta
    point.padding = 0.3,         # Distancia mínima entre punto y etiqueta
    segment.color = "#DA70D6",   # Color lila para las líneas conectoras
    segment.size = 0.3           # Grosor de las líneas conectoras
  ) +
  
  # Capa 3: Títulos y etiquetas del gráfico
  labs(
    subtitle = "If academia ever makes you feel like you're not good or smart enough . . .", 
    title = "It's not you, it's academia ✨",
    x = "Published Year",
    y = "Ratings"
  ) +
  
  # Capa 4: Tema base y personalización completa
  theme_minimal() +
  theme(
    plot.background = element_rect(fill = "#FFF0F5", color = NA),
    plot.title = element_text(
      hjust = 0.5,               # Centrado horizontal
      color = "#FF1493",         # Rosa fuerte (DeepPink)
      size = 12, 
      face = "bold"              # Negrita
    ),
    plot.subtitle = element_text(
      hjust = 0.5,               # Centrado horizontal
      color = "#DA70D6",         # Orquídea/Lila
      size = 10, 
      face = "italic"            # Cursiva
    ),
    axis.title = element_text(color = "#8B4789", face = "bold"),
    axis.text = element_text(color = "#C71585"),
    panel.grid.major = element_line(
      color = "#FFE4E1",         # Rosa muy claro (MistyRose)
      linetype = "dashed"        # Líneas discontinuas
    ),
    panel.grid.minor = element_blank(),
    legend.position = "none"     # Oculta la leyenda
  )
```
## Características avanzadas del gráfico

### 1. **Puntos con tamaño variable (encodificación visual)**
El tamaño de cada círculo representa directamente el rating del libro. Esta es una técnica de visualización llamada "encodificación por tamaño" que permite mostrar tres dimensiones de información en un gráfico 2D: año (eje X), rating numérico (eje Y), y rating visual (tamaño del punto). Los lectores pueden identificar instantáneamente los libros mejor valorados por su mayor tamaño.

### 2. **Dispersión controlada para claridad**
El jitter no es aleatorio descontrolado, sino cuidadosamente calibrado. Los valores de ±0.4 para años y ±0.3 para ratings fueron elegidos después de considerar:
- La densidad de datos (cuántos libros por año)
- La escala visual del gráfico
- La necesidad de mantener la precisión de los datos reales

### 3. **Sistema de etiquetas inteligente**
`geom_text_repel()` implementa un algoritmo sofisticado que:
- Calcula la posición óptima para cada etiqueta
- Minimiza las superposiciones usando un sistema de "fuerzas" (similar a física de partículas)
- Dibuja líneas conectoras solo cuando es necesario
- Ajusta dinámicamente según el espacio disponible

## 📊 Interpretación del gráfico resultante

El gráfico final permite observar varios patrones interesantes:

1. **Evolución temporal**: Podemos ver cómo la producción literaria de Ali Hazelwood se ha desarrollado a lo largo de los años.

2. **Consistencia de calidad**: El tamaño de los puntos nos muestra si la autora mantiene ratings consistentemente altos o si hay variabilidad.

3. **Libros destacados**: Los puntos más grandes identifican instantáneamente las obras más aclamadas por los lectores.

4. **Concentración de publicaciones**: Áreas con mayor densidad de puntos indican períodos de alta productividad.

## Parámetros personalizables

El código está diseñado para ser fácilmente personalizable. Algunos parámetros clave que puedes ajustar:

### Dispersión de puntos
```r
# Aumentar dispersión (puntos más separados)
year_jitter = published_year + runif(n(), -0.6, 0.6)
ratings_jitter = ratings + runif(n(), -0.4, 0.4)

# Reducir dispersión (puntos más juntos)
year_jitter = published_year + runif(n(), -0.2, 0.2)
ratings_jitter = ratings + runif(n(), -0.15, 0.15)
```
### Tamaño de puntos
```r
# Puntos más grandes
scale_size_continuous(range = c(8, 50))

# Puntos más pequeños
scale_size_continuous(range = c(3, 20))
```

### Etiquetas de texto
```r
# Texto más grande
geom_text_repel(size = 4, ...)

# Más tolerancia al solapamiento
geom_text_repel(max.overlaps = 30, ...)
```

## 📝 Notas técnicas avanzadas

### 1. Función `runif()` explicada
`runif(n(), min, max)` genera números aleatorios con distribución uniforme:
- `n()`: función de dplyr que devuelve el número de filas
- `min, max`: límites del rango de valores posibles
- Cada número tiene la misma probabilidad de ser generado dentro del rango

### 2. El rol de `set.seed()`
Sin `set.seed()`, cada ejecución produciría una disposición diferente de puntos. Con `set.seed(123)`:
- El "123" es arbitrario (puede ser cualquier número)
- Garantiza reproducibilidad para publicaciones científicas
- Permite a otros investigadores obtener exactamente el mismo gráfico

### 3. Por qué `geom_text_repel()` sobre `geom_text()`
La función estándar `geom_text()` simplemente coloca texto en coordenadas fijas, `geom_text_repel()` implementa un algoritmo de optimización que:
- Calcula posiciones óptimas automáticamente
- Usa "fuerzas de repulsión" simuladas entre etiquetas
- Mantiene las etiquetas cerca de sus puntos correspondientes
- Dibuja líneas conectoras cuando es necesario alejar la etiqueta

## Conclusión

Este proyecto utiliza datos de Goodreads y fue desarrollado con fines educativos y de análisis literario. La visualización y el código son de código abierto y pueden ser reutilizados con la debida atribución. Este primer proyecto fue un intento de tomar la ciencia de datos y su análisis con uno de mi mayores Hobbies en el mundo: ¡la lectura! 

Espero poder ayudar en algo y que otras personas puedan imaginar como ranquear sus libros en el futuro...

## Producto Final
A continuación, se aprecia el modelo final. ¡Puedes revisar que te pareció!

<img width="597" height="368" alt="image" src="https://github.com/user-attachments/assets/30a7d9a8-6e96-4334-9848-bd6a8d9a82d3" />
