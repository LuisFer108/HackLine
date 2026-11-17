# HACKLINE: 0xFFFF

> **Purga de procesos corruptos** — juego de acción en tiempo real escrito 100 % en **Jack** para la plataforma **Hack** (Nand2Tetris).

**Curso:** Organización de Computadores 2026 · **Evaluación:** Examen Parcial 4

---

## Enlaces
|---|---|
| Video de funcionamiento y arquitectura | **[]** |

## Integrantes del equipo

- Manuela Buriticá Guzmán
- Miguel Martinez 
- Luis Fernando Bernal Ramírez

---

## Concepto del juego

Eres el **Recolector de Basura**, un proceso que patrulla el **bus de datos** de una máquina Hack. Procesos corruptos (calaveras) descienden por el bus y te persiguen lateralmente. Tu única arma es el **Disco de Puntero**, un proyectil que rebota contra las paredes del bus. **Un golpe enemigo te mata**; sobrevive y encadena eliminaciones para multiplicar tu puntaje.

### Mecánicas

| Mecánica | Descripción |
|---|---|
| Movimiento | WASD o flechas, en tiempo real, sin pausas bloqueantes. |
| Disco de Puntero | Se lanza con ESPACIO en la dirección de apuntado (última dirección de movimiento). Solo hay un disco activo a la vez. |
| Rebote | Cinemática 2D entera con reflexión vectorial (`Vx = -Vx`, `Vy = -Vy`) contra las cuatro paredes de la arena. El disco desaparece al superar 4 rebotes. |
| Enemigos | 4 procesos que caen a velocidad distinta (máx. 3 px/frame) y persiguen al jugador en X cada 2 frames. Reaparecen arriba en posición pseudoaleatoria. |
| Combo | Cada eliminación suma `100 × combo` puntos e incrementa el combo. Hay una ventana de 110 frames para encadenar; si expira, el combo vuelve a ×1. |
| Muerte | Colisión enemigo–jugador (AABB) = fin de la partida. |
| Reinicio | En *Game Over*, la tecla **R** devuelve a la pantalla de inicio para jugar de nuevo. |

### Controles

| Tecla | Acción |
|---|---|
| `W` / `↑` | Mover arriba |
| `S` / `↓` | Mover abajo |
| `A` / `←` | Mover izquierda |
| `D` / `→` | Mover derecha |
| `ESPACIO` | Iniciar partida / lanzar disco |
| `R` | Reiniciar tras Game Over |

---

## Cumplimiento de los requisitos técnicos

| Requisito | Cómo se cumple | Clase |
|---|---|---|
| **Interactividad por teclado** | `Keyboard.keyPressed()` se consulta una vez por frame dentro del bucle principal; nunca se usa `readChar`/`readLine` (que bloquean). | `HacklineGame.handleInput()` |
| **Fondo móvil / entorno dinámico** | El "bus de datos" se desplaza continuamente hacia abajo escribiendo patrones directamente en la VRAM (columnas 8 y 23). | `HacklineGame.updateScrollingBus()` |
| **Comportamiento físico modelado** | El Disco usa posición y velocidad enteras y rebote elástico en los 4 bordes; los enemigos tienen persecución y los choques usan cajas AABB. | `Disc.update()`, `Enemy.update()`, `MathUtils.checkCollision()` |
| **Visualización de puntaje** | HUD con `SCORE`, `COMBO` y estado del `DISCO`; las etiquetas se dibujan una vez y solo los valores se actualizan cada frame. | `HacklineGame.drawStaticHUD()` / `updateHUD()` |

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
| `HacklineGame.jack` | Motor principal: bucle de juego, lectura de teclado, scroll del bus, HUD, puntaje, combo, colisiones y pantalla de Game Over. |
| `Player.jack` | Entidad del jugador: posición, vector de apuntado, límites del bus y dibujo/borrado en VRAM. |
| `Disc.jack` | Proyectil con cinemática 2D y rebote elástico; contador de rebotes y sprite propio. |
| `Enemy.jack` | Enemigos: caída vertical, persecución horizontal, reaparición y sprite propio. |
| `MathUtils.jack` | Utilidades de bajo nivel: tabla de potencias de 2, PRNG y detección de colisiones AABB. |

Cada clase implementa `new()`, `draw()`/`erase()` (cuando aplica) y `dispose()` para liberar su memoria.

### Mapa de memoria de video usada

- VRAM Hack: `16384` en adelante, **32 palabras por fila** (512 px / 16).
- Dirección de una palabra: `16384 + (y * 32) + (x / 16)`.
- Desplazamiento dentro de la palabra: `x - (x/16)*16`.
- Arena de juego: aprox. `x ∈ [145, 352]`, `y ∈ [18, 244]`; HUD en las filas superiores.
- Rieles del bus animados: columnas 8 (x = 128..143) y 23 (x = 368..383).

---

## Cómo ejecutarlo

1. Clonar este repositorio con los 6 archivos `.jack` en **una misma carpeta**.
2. Compilar la carpeta con el **JackCompiler** de Nand2Tetris (genera los `.vm`).
3. Abrir el **VM Emulator**, cargar la carpeta con los `.vm` (junto con la librería estándar del OS).
4. Configurar a la máxima velocidad.
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
