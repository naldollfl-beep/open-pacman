# SPEC 01 — Fantasmas con personalidades clásicas

> **Estado:** Aprobado
> **Depende de:** —
> **Fecha:** 2026-10-05
> **Objetivo:** Los 4 fantasmas pasan de hunter/ambusher/patrol/random a las personalidades clásicas del arcade: Blinky persigue directo a Pac-Man, Pinky apunta delante de él, Inky triangula con Blinky y Clyde alterna persecución y retirada.

## Por qué existe este spec

El juego ya tiene 4 fantasmas con personalidades propias (`decideGhost` en `src/js/game.js`), pero no son las del arcade. Este spec las rediseña según las clásicas sin tocar nada más del juego. Los comentarios del código que citan `(spec 01..04)` pertenecen a specs previos no conservados; este spec no depende de ellos.

## Scope

**In:**

- Renombrar los `kind`: `hunter`→`blinky`, `ambusher`→`pinky`, `patrol`→`inky`, `random`→`clyde` en `GHOST_STARTS` (`src/js/maze.js`) y `GHOST_COLORS_BY_KIND` (`src/js/render.js`).
- Reescribir `decideGhost` (`src/js/game.js`) con las 4 personalidades clásicas.
- Constantes en `src/js/game.js`: añadir `PINKY_AIM_STRIDE`, `INKY_AIM_STRIDE`, `CLYDE_RETREAT_DIST`, `CLYDE_CORNER`; eliminar `AMBUSHER_AIM_STRIDE` y `PATROL_CORNERS`.

**Out of scope (para futuros specs):**

- Modo scatter (alternancia persecución/dispersión periódica).
- Cruise Elroy (Blinky acelera con pocos dots).
- Bug del arcade en Pinky (mirar arriba apunta 4 arriba + 4 izquierda).
- Timing de salida de la pen (se mantiene el escalonado actual de 1.5 s).
- Modo frightened/eyes, velocidades, pen, túnel y puntuación (no cambian).

## Data model

```js
// maze.js — GHOST_STARTS (posiciones y orden de salida sin cambio; solo kind)
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky' }, // sale 1º
  { x: 12, y: 14, kind: 'pinky' },  // sale 2º
  { x: 15, y: 14, kind: 'inky' },   // sale 3º
  { x: 14, y: 14, kind: 'clyde' },  // sale 4º
];

// game.js — constantes nuevas (reemplazan a AMBUSHER_AIM_STRIDE y PATROL_CORNERS)
const PINKY_AIM_STRIDE = 4;         // celdas por delante de Pac-Man
const INKY_AIM_STRIDE = 2;          // celdas por delante para el vector de Inky
const CLYDE_RETREAT_DIST = 8;       // distancia Manhattan que activa la retirada
const CLYDE_CORNER = { x: 1, y: 29 }; // esquina inferior-izquierda de Clyde

// Targets de cada personalidad (celdas tx,ty dentro de decideGhost):
// blinky: celda de Pac-Man
// pinky:  Pac-Man + PINKY_AIM_STRIDE * dir de Pac-Man
// inky:   2 * (Pac-Man + INKY_AIM_STRIDE * dir) - posición de blinky (vector doblado)
// clyde:  Pac-Man si dist Manhattan > CLYDE_RETREAT_DIST; CLYDE_CORNER si <=
```

## Plan de implementación

1. `src/js/game.js`: añadir las 4 constantes nuevas (sin tocar la lógica; nada las usa aún y el juego corre igual).
2. `src/js/maze.js` + `src/js/render.js`: renombrar `kind` y claves de `GHOST_COLORS_BY_KIND` (colores sin cambio). Los 4 caen temporalmente a la rama aleatoria de `decideGhost` (kinds aún no reconocidos); el juego sigue funcional.
3. `src/js/game.js`: reescribir `decideGhost` con las 4 ramas clásicas y eliminar `AMBUSHER_AIM_STRIDE` y `PATROL_CORNERS` (ya sin referencias). Verificación manual en navegador con la checklist.

## Criterios de aceptación

- [ ] El juego carga desde `file://` sin errores en la consola.
- [ ] Hay 4 fantasmas con colores clásicos: rojo (blinky), rosa (pinky), cian (inky), naranja (clyde).
- [ ] blinky (rojo) elige en cada cruce la dirección que reduce la distancia a la celda de Pac-Man (persecución directa).
- [ ] pinky (rosa) apunta 4 celdas delante de la dirección en que Pac-Man mira (se adelanta al camino).
- [ ] El blanco de inky se calcula como `2 * (2 celdas delante de Pac-Man) - posición de blinky` (verificable con un `console.log` puntual o breakpoint en `decideGhost`).
- [ ] clyde (naranja) persigue a Pac-Man a más de 8 celdas (Manhattan) y se dirige a `(1,29)` a 8 o menos.
- [ ] El modo asustado (aleatorio), los ojos y la salida escalonada de la pen (1 cada 1.5 s) no cambian.
- [ ] No queda ninguna referencia a `hunter`/`ambusher`/`patrol`/`random` ni a `AMBUSHER_AIM_STRIDE`/`PATROL_CORNERS` en `src/`.

## Decisiones

- **Sí:** personalidades clásicas del arcade — es el rediseño pedido; sustituyen a las actuales.
- **Sí:** renombrar los `kind` a `blinky`/`pinky`/`inky`/`clyde` — el nombre documenta la personalidad en los 3 archivos.
- **Sí:** blinky = persecución directa pura, sin Cruise Elroy ni velocidad extra.
- **No:** bug del arcade en pinky — siempre 4 celdas delante; más simple y verificable.
- **No:** scatter — la retirada de clyde por distancia no es scatter periódico; va en otro spec.
- **No:** Euclídea para clyde — se usa Manhattan, consistente con la métrica existente de `decideGhost`.
- **Sí:** esquina de clyde = `(1,29)` (abajo-izquierda) — replica su esquina del arcade.
- **Sí:** numerar como SPEC 01 — `specs/` está vacía; las citas `(spec 01..04)` del código son de specs previos no conservados.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Inky depende de la posición de blinky (p. ej. blinky dentro de la pen) | El blanco solo rompe empates de dirección; un blanco raro no rompe nada, solo cambia el giro |
| Verificar pinky/inky/clyde a simple vista es difícil | Verificación manual con `console.log`/breakpoint puntual, permitido por AGENTS.md (no hay tests) |

## Qué **no** está en este spec

- Modo scatter, Cruise Elroy, bug de Pinky, timing de salida de la pen, cambios en frightened/eyes/score. Cada uno, si algún día llega, va en su propio spec.
