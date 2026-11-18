# HACKLINE: 0xFFFF

> **Purga de procesos corruptos** — juego de acción en tiempo real escrito 100 % en **Jack** para la plataforma **Hack** (Nand2Tetris).

**Curso:** Organización de Computadores 2026 · **Evaluación:** Examen Parcial 4

---

## Enlaces
| Video de funcionamiento y arquitectura | **https://youtu.be/H8Tuf92exSs** |

## Integrantes del equipo

- Manuela Buriticá Guzmán
- Miguel Martinez 
- Luis Fernando Bernal Ramírez

---

## Concepto del juego
 
Eres el **Recolector de Basura**, un proceso que patrulla el **bus de datos** de una máquina Hack. Procesos corruptos (calaveras) descienden por el bus y te persiguen lateralmente. Tu única arma es el **Disco de Puntero**, un proyectil que sale recto o en diagonal y rebota contra las paredes del bus. **Un golpe enemigo te mata**; sobrevive y encadena eliminaciones para multiplicar tu puntaje.
 
### Mecánicas
 
| Mecánica | Descripción |
|---|---|
| Movimiento | En **8 direcciones**, en tiempo real y sin pausas bloqueantes: WASD o flechas para las direcciones rectas y `Q` `E` `Z` `C` para las diagonales. |
| Disco de Puntero | Se lanza con ESPACIO en la dirección de apuntado (última dirección de movimiento): ±8 px/frame en recto, ±6 en cada eje en diagonal. Solo hay un disco activo a la vez y puede eliminar a más de un enemigo en su trayectoria. |
| Rebote | Cinemática 2D entera con reflexión vectorial (`Vx = -Vx`, `Vy = -Vy`) contra las cuatro paredes de la arena. El disco se desactiva al superar 5 rebotes (en el 6.º) y vuelve a estar disponible. |
| Enemigos | 4 procesos que caen a velocidad distinta (1, 2, 3 y 3 px/frame; máx. 3) y persiguen al jugador en X cada 2 frames. Reaparecen arriba en posición pseudoaleatoria. |
| Combo | Cada eliminación suma `100 × combo` puntos e incrementa el combo. Hay una ventana de 110 frames para encadenar; si expira, el combo vuelve a ×1. |
| Muerte | Colisión enemigo–jugador (AABB) = fin de la partida. |
| Reinicio | En *Game Over*, **ESPACIO** vuelve a la pantalla de inicio; desde ahí, **ESPACIO** inicia una nueva partida. |
 
### Controles
 
| Tecla | Acción |
|---|---|
| `W` / `↑` | Mover arriba |
| `S` / `↓` | Mover abajo |
| `A` / `←` | Mover izquierda |
| `D` / `→` | Mover derecha |
| `Q` | Mover arriba-izquierda |
| `E` | Mover arriba-derecha |
| `Z` | Mover abajo-izquierda |
| `C` | Mover abajo-derecha |
| `ESPACIO` | Iniciar partida / lanzar disco / volver al menú tras Game Over |
 
> Hack solo expone **una tecla a la vez** en su registro de teclado. Por eso las diagonales tienen teclas propias (`Q` `E` `Z` `C`) y, además, el juego recuerda durante 4 frames la última dirección horizontal y vertical para combinar dos teclas (por ejemplo `W` + `D`).
 
---
 
## Cumplimiento de los requisitos técnicos
 
| Requisito | Cómo se cumple | Clase |
|---|---|---|
| **Interactividad por teclado** | `Keyboard.keyPressed()` se consulta una vez por frame dentro del bucle principal; nunca se usa `readChar`/`readLine` (que bloquean). Movimiento en 8 direcciones. | `HacklineGame.handleInput()` |
| **Fondo móvil / entorno dinámico** | El "bus de datos" se desplaza continuamente hacia abajo escribiendo patrones directamente en la VRAM (columnas 8 y 23). | `HacklineGame.updateScrollingBus()` |
| **Comportamiento físico modelado** | El Disco usa posición y velocidad enteras y rebote elástico diagonal en los 4 bordes; los enemigos tienen persecución y los choques usan cajas AABB. | `Disc.update()`, `Enemy.update()`, `MathUtils.checkCollision()` |
| **Visualización de puntaje** | HUD con `SCORE`, `COMBO` y estado del `DISCO`; las etiquetas se dibujan una vez y solo los valores se actualizan cada frame. | `HacklineGame.drawStaticHUD()` / `updateHUD()` |
 
---
 
## Diferenciador técnico
 
- **Renderizado de sprites con precisión de píxel escribiendo en VRAM**: en lugar de `Screen.drawRectangle` o `drawPixel`, cada sprite (12×12 el jugador y los enemigos, 8×8 el disco) se guarda como un arreglo de palabras de 16 bits y se dibuja con `Memory.poke` repartiendo los bits entre **dos palabras adyacentes** de la memoria de pantalla. Así el movimiento es de 1 píxel aunque la VRAM se direccione por palabras de 16 bits.
- **Desplazamiento de bits sin instrucción de *shift***: Hack no tiene desplazamientos, así que se simulan multiplicando y dividiendo por potencias de 2, que se obtienen de una tabla (`MathUtils.pow2`) en lugar de calcularlas.
- **Movimiento en 8 direcciones con un solo registro de teclado**: teclas diagonales directas y memoria de 4 frames para combinar direcciones.
- **Disco con rebote diagonal tipo Pong** y **sistema de combos con temporizador**, con dificultad por velocidades escalonadas de enemigos.
- **Generador pseudoaleatorio congruencial lineal** propio (sin depender de librerías externas).
- **Gestión estricta de memoria**: etiquetas del HUD dibujadas una sola vez, textos liberados apenas se imprimen (`printAndDispose`), textos de estado reutilizados (`strAir`, `strReady`) y números con `printInt`, para no agotar el Heap.

 
---
 
## Arquitectura
 
```
                    ┌──────────────────┐
                    │      Main        │  menú, ciclo de partidas
                    └────────┬─────────┘
                             │ crea / ejecuta / libera
                    ┌────────▼─────────┐
                    │  HacklineGame    │  bucle principal, HUD,
                    │                  │  entrada, colisiones, score
                    └─┬───────┬──────┬─┘
          ┌───────────┘       │      └────────────┐
   ┌──────▼─────┐      ┌──────▼─────┐      ┌──────▼─────┐
   │   Player   │      │    Disc    │      │ Enemy (x4) │
   └──────┬─────┘      └──────┬─────┘      └──────┬─────┘
          └───────────────────┼───────────────────┘
                       ┌──────▼─────┐
                       │ MathUtils  │  pow2, rand, AABB
                       └────────────┘
```
 
`Player`, `Disc` y `Enemy` no se conocen entre sí: solo `HacklineGame` los conoce, les pide sus posiciones, decide las colisiones y les da órdenes.
 
### Ciclo de cada frame (`HacklineGame.run()`)
 
1. Actualizar fondo móvil (scroll en VRAM)
2. Leer teclado (no bloqueante)
3. Actualizar física del disco
4. Actualizar enemigos y resolver colisiones (disco→enemigo, enemigo→jugador)
5. Gestionar temporizador de combo
6. Dibujar jugador, disco y HUD
7. `Sys.wait(16)` para regular la velocidad
### Estructura de clases
 
| Archivo | Responsabilidad |
|---|---|
| `Main.jack` | Punto de entrada. Inicializa `MathUtils`, muestra el menú y crea/ejecuta/libera cada partida. |
| `HacklineGame.jack` | Motor principal: bucle de juego, lectura de teclado (8 direcciones), scroll del bus, HUD, puntaje, combo, colisiones y pantalla de Game Over. |
| `Player.jack` | Entidad del jugador: posición, vector de apuntado (recto o diagonal), límites del bus y dibujo/borrado en VRAM. |
| `Disc.jack` | Proyectil con cinemática 2D diagonal y rebote elástico; contador de rebotes y sprite propio. |
| `Enemy.jack` | Enemigos: caída vertical, persecución horizontal, reaparición y sprite propio. |
| `MathUtils.jack` | Utilidades de bajo nivel: tabla de potencias de 2, PRNG y detección de colisiones AABB. |
 
Cada clase implementa `new()`, `draw()`/`erase()` (cuando aplica) y `dispose()` para liberar su memoria.
 
### Mapa de memoria de video usada
 
- VRAM Hack: `16384` en adelante, **32 palabras por fila** (512 px / 16).
- Dirección de una palabra: `16384 + (y * 32) + (x / 16)`.
- Desplazamiento dentro de la palabra: `x - (x/16)*16`.
- En cada palabra el **bit 0 es el píxel de más a la izquierda**.
- Arena de juego: aprox. `x ∈ [145, 352]`, `y ∈ [18, 244]`; HUD en las filas superiores.
- Rieles del bus animados: columnas 8 (x = 128..143) y 23 (x = 368..383), que ningún sprite alcanza a tocar.
---
 
## Cómo ejecutarlo
 
1. Clonar este repositorio con los 6 archivos `.jack` en **una misma carpeta**.
2. Compilar la carpeta con el **JackCompiler** de Nand2Tetris (genera los `.vm`).
3. Abrir el **VM Emulator**, cargar la carpeta con los `.vm` (junto con la librería estándar del OS).
4. Configurar la animación en **"No animation"** para una velocidad jugable.
5. Ejecutar (`Run`) y presionar **ESPACIO** para iniciar.
```
HACKLINE/
├── Main.jack
├── HacklineGame.jack
├── Player.jack
├── Disc.jack
├── Enemy.jack
├── MathUtils.jack
└── README.md
```
 
