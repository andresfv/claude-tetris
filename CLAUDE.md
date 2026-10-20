# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Resumen

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin `package.json`, sin build, sin linter ni tests. El código, la UI y el README están en español; mantener ese idioma en textos visibles al usuario y comentarios.

## Ejecutar

Abrir `index.html` en el navegador, o servirlo estáticamente (recomendado):

```bash
python -m http.server 8000   # luego http://localhost:8000
npx serve .
```

No hay suite de tests: la verificación es manual jugando en el navegador.

## Arquitectura

Tres archivos: `index.html` (DOM: `<canvas id="board">`, panel lateral con `#score`/`#lines`/`#level`/`#next-canvas`, y un único `#overlay` reutilizado para PAUSA y GAME OVER), `style.css` y `game.js` (toda la lógica, script clásico en modo estricto, sin módulos).

Puntos de `game.js` que conviene conocer antes de editar:

- **Estado global mutable**: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId` se declaran a nivel de módulo y `init()` los reinicia todos. Cualquier estado nuevo debe inicializarse en `init()` (que también se usa para el botón Reiniciar).
- **Codificación de celdas**: el valor de cada celda en `PIECES` es el índice del tipo de pieza (1–7 clásicas, 8 = tuerca), que a su vez indexa `COLORS`. La tuerca es un anillo 3×3 con hueco central (`0`) y sale con probabilidad `NUT_CHANCE` (5 %) en `randomPiece()`. `board` guarda esos mismos índices (`0` = vacío). Índice 0 de `PIECES`/`COLORS` es `null` a propósito.
- **Piezas**: `current`/`next` son `{ type, shape, x, y }`; `shape` es una copia de la matriz de `PIECES` y la rotación (`rotateCW`) la reemplaza. `tryRotate` aplica kicks horizontales `[0, -1, 1, -2, 2]` (no es SRS).
- **Colisión** (`collide`): permite `y < 0` (filas por encima del tablero) y es la única fuente de verdad para movimiento, rotación, ghost y caída.
- **Ciclo de pieza**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promueve `next` a `current`, genera una nueva `next`, detecta Game Over si la pieza ya colisiona, y redibuja la vista previa.
- **Game loop**: `requestAnimationFrame(loop)` acumula `dt` en `dropAccum` y baja una fila al superar `dropInterval`. La pausa y el game over cancelan el frame con `cancelAnimationFrame(animId)`; reanudar llama a `loop` directamente tras resetear `lastTime`.
- **Puntuación/nivel**: `LINE_SCORES[n] * level` por líneas, +1 por fila en soft drop, +2 por celda en hard drop; nivel = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Temas**: colores en variables CSS (`:root` oscuro, `[data-theme="light"]` claro). El switch `#theme-toggle` alterna `document.documentElement.dataset.theme`, recalcula `gridColor` (leído de `--grid`) y redibuja. El tema no se toca en `init()` ni se persiste.
- **Dimensiones acopladas**: si se cambia `COLS`, `ROWS` o `BLOCK`, hay que ajustar `width`/`height` del `<canvas id="board">` en `index.html` (`COLS×BLOCK` × `ROWS×BLOCK`). La vista previa asume una rejilla de 4×4 bloques de 30 px (canvas de 120×120).
