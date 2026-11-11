# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla con HTML5 Canvas. Sin dependencias, sin build, sin tests, sin linter. Mantenerlo así: no añadir `package.json`, bundlers ni frameworks. Detalles de usuario, controles y personalización en @README.md.

## Ejecutar

- Abrir `index.html` directamente en el navegador, o
- Servidor estático: `python -m http.server 8000` y abrir `http://localhost:8000`.

No hay comandos de build, test ni lint. Verificar cambios jugando en el navegador.

## Arquitectura

Tres archivos: `index.html` (DOM + canvas), `style.css` (tema oscuro retro), `game.js` (toda la lógica, ~300 líneas, script clásico sin módulos).

`game.js`:
- **Estado global** (variables `let` compartidas): `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `lastTime`, `dropAccum`, `dropInterval`, `animId`. Todo se reinicia en `init()`.
- **Tablero**: matriz `ROWS × COLS`; `0` = vacía, `1–7` = índice de color/pieza (`COLORS`, `PIECES`).
- **Piezas**: `{ type, shape, x, y }`; `shape` es una matriz cuadrada. `rotateCW` = transponer + invertir filas. Siempre se copia `PIECES[type]` (nunca mutar la plantilla).
- **Colisión**: `collide(shape, ox, oy)` es la única comprobación; usada para mover, rotar, ghost y spawn. Permite `ny < 0` (sobre el tablero) sin colisionar.
- **Wall kicks**: `tryRotate` prueba desplazamientos `[0, -1, 1, -2, 2]`; si ninguno cabe, no rota.
- **Ciclo de pieza**: `softDrop`/`hardDrop`/`loop` → `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. Si la nueva pieza colisiona al aparecer → `endGame()`.
- **Bucle**: `loop(ts)` con `requestAnimationFrame`; acumula `dropAccum` y baja una fila al superar `dropInterval`; llama a `draw()` cada frame.
- **Puntuación y nivel** (en `clearLines`): `LINE_SCORES[n] * level`; hard drop +2 por celda, soft drop +1 por fila; `level = floor(lines/10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Render**: `draw()` pinta grid, tablero, ghost (`ghostY`, alpha 0.2) y pieza actual; `drawNext()` pinta la vista previa en `#next-canvas`; `drawBlock` es compartido por ambos canvas.
- **Input**: un único listener `keydown` (usa `e.code`: flechas, `KeyX`, `Space`, `KeyP`). Pausa y game over comparten el overlay (`#overlay`).

## Gotchas

- Cambiar `COLS`, `ROWS` o `BLOCK` obliga a actualizar `width`/`height` de `<canvas id="board">` en `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`, ahora 300×600). El canvas de `next` es 120×120 con bloques de 30 px fijos en `drawNext`.
- `game.js` obtiene elementos del DOM por id al cargar; renombrar ids en `index.html` lo rompe.
- `togglePause` reanuda llamando a `loop` directamente tras reiniciar `lastTime`; no olvidar `lastTime` al tocar pausa/reinicio o la pieza salta.
- README y textos de UI están en español; mantener el idioma al editar.
