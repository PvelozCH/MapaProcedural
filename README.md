# Generador de Mapa Procedural

## 1. Descripción General

Este proyecto es una aplicación de escritorio desarrollada en Python que genera y visualiza mapas procedurales en 2D. Utiliza la biblioteca `pygame` para la renderización y el manejo de la ventana, y el algoritmo de **ruido Perlin** para crear un terreno de apariencia natural y coherente.

El mapa generado incluye múltiples biomas como océanos, playas, praderas y cadenas montañosas, definidos por la elevación y la humedad. Adicionalmente, se generan estructuras aleatorias como casas, torres y ruinas en zonas habitables. La aplicación permite al usuario generar nuevos mapas al instante con solo presionar una tecla.

## 2. Características

- **Generación Procedural:** Creación de mapas únicos y complejos a partir de una semilla aleatoria.
- **Ruido Perlin:** Uso de ruido Perlin de múltiples octavas para generar valores de elevación y humedad, resultando en un terreno orgánico.
- **Biomas Diversos:** Simulación de diferentes biomas basados en la altitud y la humedad.
- **Estructuras Aleatorias:** Colocación de estructuras (casas, torres, ruinas) en lugares apropiados del mapa.
- **Interactividad:** Posibilidad de regenerar el mapa con una nueva semilla en tiempo de ejecución.
- **Visualización Clara:** Uso de una paleta de colores definida para distinguir fácilmente los diferentes tipos de terreno y estructuras.

## 3. Requisitos Previos

Para ejecutar este proyecto, es necesario tener Python 3 instalado. Las siguientes bibliotecas de Python son requeridas:

- `pygame`: Para la creación de la ventana, el dibujado y la gestión de eventos.
- `noise`: Para la implementación del algoritmo de ruido Perlin.

## 4. Instalación

1.  Clona o descarga este repositorio en tu máquina local.
2.  Navega al directorio del proyecto a través de una terminal.
3.  Instala las dependencias listadas en el archivo `requirements.txt` ejecutando el siguiente comando:

    ```bash
    pip install -r requirements.txt
    ```

## 5. Uso

Una vez instaladas las dependencias, puedes ejecutar la aplicación con el siguiente comando:

```bash
python main.py
```

Se abrirá una ventana de 800x600 píxeles mostrando el mapa generado.

### Controles

-   **`R`**: Presiona la tecla 'R' en cualquier momento para descartar el mapa actual y generar uno completamente nuevo con una semilla diferente.
-   **Cerrar Ventana**: Cierra la ventana para finalizar la aplicación.

---

## 6. Análisis Detallado del Código (`main.py`)

A continuación, se desglosa el funcionamiento de cada parte del script.

### 6.1. Importaciones

```python
import pygame
import random
import noise
from pygame.locals import *
```

-   `pygame`: Es la biblioteca fundamental que provee los módulos necesarios para crear aplicaciones multimedia y juegos. Se usa para inicializar el sistema, crear la pantalla, dibujar formas y gestionar eventos del usuario.
-   `random`: Módulo estándar de Python para la generación de números pseudoaleatorios. Es crucial para seleccionar la semilla (`SEED`) del mapa y para decidir dónde colocar las estructuras.
-   `noise`: Biblioteca externa que implementa el algoritmo de ruido Perlin (`pnoise2`), una técnica de gradiente de ruido que genera texturas de apariencia natural. Es la base para la creación del terreno.
-   `from pygame.locals import *`: Importa todas las constantes y funciones del módulo `pygame.locals`, como `QUIT` (evento de cierre), `KEYDOWN` (evento de tecla presionada) y `K_r` (código de la tecla 'R'). Esto permite un código más limpio al no tener que usar prefijos como `pygame.QUIT`.

### 6.2. Configuración Inicial

```python
# Inicialización de Pygame
pygame.init()

# Configuración de la pantalla
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Generador de Mapa Procedural")

# Configuración del mapa
TILE_SIZE = 20
MAP_WIDTH = WIDTH // TILE_SIZE
MAP_HEIGHT = HEIGHT // TILE_SIZE

# Semilla aleatoria
SEED = random.randint(0, 9999)
random.seed(SEED)
```

-   `pygame.init()`: Llama a las rutinas de inicialización de todos los módulos de Pygame importados. Es un paso obligatorio.
-   `WIDTH`, `HEIGHT`: Definen las dimensiones de la ventana en píxeles.
-   `pygame.display.set_mode()`: Crea la superficie principal de dibujado (la pantalla) con las dimensiones especificadas.
-   `TILE_SIZE`: Define el lado de cada celda cuadrada del mapa en píxeles. Un valor más pequeño resultaría en un mapa más detallado pero con mayor carga de cómputo.
-   `MAP_WIDTH`, `MAP_HEIGHT`: Calculan las dimensiones del mapa en unidades de celdas, usando la división entera (`//`).
-   `SEED`: Se genera una semilla aleatoria. El ruido Perlin, aunque parece aleatorio, es determinista: para la misma semilla, siempre producirá el mismo resultado. Esto es útil para poder reproducir un mapa específico. `random.seed(SEED)` inicializa el generador de números aleatorios con esta semilla.

### 6.3. Paletas de Colores y Configuración de Ruido

```python
# Paleta de colores para los biomas
WATER_COLORS = [(65, 105, 225), (70, 130, 180), (0, 105, 148)]
# ... (otros colores)

# Configuración del ruido Perlin
OCTAVES = 6
PERSISTENCE = 0.5
LACUNARITY = 2.0
SCALE = 100.0
```

-   **Colores**: Se definen tuplas `(R, G, B)` para los distintos tipos de terreno. Agruparlos en listas (`WATER_COLORS`) permite seleccionar variaciones de color fácilmente (por ejemplo, para representar la profundidad del agua).
-   **Parámetros de Ruido Perlin**:
    -   `SCALE`: Controla el "zoom" del ruido. Un valor alto produce un terreno con cambios suaves y a gran escala, mientras que un valor bajo crea un terreno más caótico y con cambios abruptos.
    -   `OCTAVES`: El ruido Perlin se genera sumando varias capas de ruido (octavas) con diferentes frecuencias y amplitudes. Un mayor número de octavas añade más detalle y complejidad al terreno.
    -   `PERSISTENCE`: Controla cómo disminuye la amplitud de cada octava sucesiva. Un valor de 0.5 significa que cada octava tiene la mitad de la influencia que la anterior. Una persistencia baja significa que los detalles se hacen de forma sutil y suave. Una persistencia alta es caotica y desordenada.


    -   `LACUNARITY`: Controla cómo aumenta la frecuencia de cada octava sucesiva. Un valor de 2.0 significa que cada octava es dos veces más "densa" que la anterior.

### 6.4. Función `generate_terrain()`

```python
def generate_terrain():
    for y in range(MAP_HEIGHT):
        for x in range(MAP_WIDTH):
            nx = x / MAP_WIDTH - 0.5
            ny = y / MAP_HEIGHT - 0.5
            
            elevation = noise.pnoise2(nx * SCALE, ny * SCALE, ...)
            moisture = noise.pnoise2((nx + 1000) * SCALE, (ny + 1000) * SCALE, ...)
            
            # ... Lógica de asignación de biomas ...
```

-   Esta función itera sobre cada celda (`x`, `y`) del mapa.
-   **Normalización de Coordenadas**: `nx` y `ny` se calculan para mapear las coordenadas del mapa a un rango consistente (en este caso, de -0.5 a 0.5). Esto es importante para que la función de ruido trabaje sobre un espacio de entrada uniforme. Se multiplican por `SCALE` para ajustar el nivel de zoom.
-   **Generación de Ruido**: Se llama a `noise.pnoise2()` dos veces para generar dos mapas de ruido independientes:
    1.  `elevation`: Determina la altura del terreno.
    2.  `moisture`: Determina la humedad. Se añade un gran offset (`+ 1000`) a las coordenadas para obtener un mapa de ruido diferente al de la elevación, pero usando la misma semilla.
-   **Asignación de Biomas**: Se usa una cadena de `if/elif/else` para asignar un tipo de terreno a la celda `world_map[y][x]` basándose en umbrales de `elevation`. Por ejemplo, un valor de elevación muy bajo se convierte en agua, uno medio en pradera y uno alto en montaña. El valor de `moisture` se usa para crear sub-biomas, como hierba seca o húmeda.
-   **Generación de Estructuras**: `if random.random() < 0.005:` introduce una probabilidad muy baja (0.5%) de que se genere una estructura en una celda válida (tierra o arena).

### 6.5. Función `draw_map()`

```python
def draw_map():
    for y in range(MAP_HEIGHT):
        for x in range(MAP_WIDTH):
            tile = world_map[y][x]
            rect = pygame.Rect(x * TILE_SIZE, y * TILE_SIZE, TILE_SIZE, TILE_SIZE)
            
            # ... Lógica de asignación de color ...
            
            pygame.draw.rect(screen, color, rect)
    
    # Dibujar estructuras
    # ...
```

-   Esta función es responsable de la visualización.
-   Itera sobre la matriz `world_map` ya generada.
-   `pygame.Rect()`: Crea un objeto `Rect` que define la posición y el tamaño de la celda a dibujar en la pantalla.
-   **Selección de Color**: Se elige un color de las paletas basándose en la información almacenada en `tile` (el diccionario de la celda).
-   `pygame.draw.rect()`: Dibuja el rectángulo (la celda) en la superficie `screen` con el color elegido.
-   Finalmente, itera sobre la lista `structures` y las dibuja encima del terreno.

### 6.6. Bucle Principal

```python
while running:
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_r:
                # ... Lógica de regeneración ...
    
    screen.fill((0, 0, 0))
    draw_map()
    
    # ... Mostrar información de la semilla ...
    
    pygame.display.flip()
    clock.tick(60)
```

-   Este es el corazón de la aplicación, un bucle `while` que se ejecuta continuamente.
-   `pygame.event.get()`: Obtiene una lista de todos los eventos que han ocurrido desde la última vez que se llamó (movimiento del ratón, pulsaciones de teclas, etc.).
-   **Manejo de Eventos**: Se comprueba si el usuario ha cerrado la ventana (`QUIT`) o ha presionado la tecla 'R' (`KEYDOWN` y `K_r`).
-   **Renderizado**:
    -   `screen.fill((0, 0, 0))`: Limpia toda la pantalla, pintándola de negro. Es necesario para no dejar rastros del fotograma anterior.
    -   `draw_map()`: Dibuja el estado actual del mapa.
    -   Se renderiza y muestra el texto con la semilla actual en pantalla.
    -   `pygame.display.flip()`: Actualiza el contenido de toda la pantalla para mostrar lo que se ha dibujado en este ciclo del bucle.
    -   `clock.tick(60)`: Pausa el programa el tiempo necesario para que el bucle no se ejecute más de 60 veces por segundo (60 FPS), evitando un consumo de CPU innecesario.

### 6.7. Finalización

```python
pygame.quit()
```

-   Una vez que el bucle principal termina (cuando `running` es `False`), `pygame.quit()` se encarga de desinicializar todos los módulos de Pygame de forma segura. Es lo contrario a `pygame.init()`.
