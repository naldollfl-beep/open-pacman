# SPEC 02 — Salida de la pen por teletransporte

> **Estado:** Aprobado
> **Depende de:** SPEC 01
> **Fecha:** 2026-10-15
> **Objetivo:** Los fantasmas dejan de caminar fuera de la pen: al dispararse su temporizador de salida aparecen directamente fuera, en la celda (13,11), listos para perseguir.

## Por qué existe este spec

La salida a pie actual funciona (verificado con una simulación en Node del código real: los 4 salen — blinky ~2.0 s, pinky ~4.3 s, inky ~6.2 s, clyde ~7.3 s), pero tras cada muerte de Pac-Man los 4 regresan a la pen y vuelven a esperar escalonados (hasta 6 s) más el camino por la puerta: eso se percibe como fantasmas "trancados" en su jaula. Este spec sustituye el camino por un teletransporte al soltarlos.

## Scope

**In:**

- `src/js/game.js`, rama de suelta en `moveGhost`: al dispararse `releaseAt`, colocar al fantasma en `PEN_EXIT` (13,11) con `leftPen = true` (teletransporte; el bloque de alineación de ese mismo frame llama a `decideGhost`).
- Eliminar `exitPenStep` y la rama `if ( g.released && !g.leftPen )` de `moveGhost` (código muerto tras el cambio).
- Fantasma comido (rama de `update`): reaparece al instante en `PEN_EXIT`, modo chase, `dir = 'left'` (dirección válida desde (13,11)); sin volver a la pen ni re-caminar.
- Rama de seguridad al final de `moveGhost` (fantasma fuera del mapa): mismo teletransporte a `PEN_EXIT` con `leftPen = true`.
- Eliminar `startX`/`startY` de `createGame` (quedan sin referencias).

**Out of scope (para futuros specs):**

- Escalonado de salida y `GHOST_RELEASE_INTERVAL_MS` (1 cada 1.5 s): sin cambio.
- `MAZE`, `GHOST_STARTS`, pen, puerta y su bloqueo (Pac-Man nunca; fantasmas con `leftPen`): sin cambio.
- `src/js/render.js` y `src/js/maze.js`: sin cambios (el teletransporte no afecta al dibujado).
- Modo eyes (vestigial, nunca se activa), personalidades (spec 01), velocidades, túnel y puntuación: sin cambio.

## Data model

No introduce datos nuevos; reutiliza `PEN_EXIT`, que pasa de "meta del camino a pie" a "celda de aparición":

```js
// game.js — PEN_EXIT (constante sin cambio de valor, cambia su rol)
const PEN_EXIT = { x: 13, y: 11 };

// moveGhost, rama de suelta (antes: released=true y exitPenStep caminaba):
g.released = true;
g.x = PEN_EXIT.x; g.y = PEN_EXIT.y; g.leftPen = true;

// update, fantasma comido (antes: g.x = g.startX; g.y = g.startY; ...):
g.x = PEN_EXIT.x; g.y = PEN_EXIT.y; g.dir = 'left';
g.mode = 'chase'; g.leftPen = true; g.released = true;

// Eliminados: exitPenStep(), startX/startY
```

## Plan de implementación

1. `src/js/game.js` — `moveGhost`: teletransporte en la rama de suelta + eliminar `exitPenStep` y la rama del camino a pie. La salida inicial y los resets por muerte ya funcionan con la nueva mecánica (verificable en navegador: iniciar partida y morir).
2. `src/js/game.js` — fantasma comido en `update` y rama de seguridad en `moveGhost`: teletransporte a `PEN_EXIT` + `leftPen = true`; eliminar `startX`/`startY` de `createGame`. Verificación manual completa con la checklist (power pellet, comer fantasmas, morir).

## Criterios de aceptación

- [ ] El juego carga desde `file://` sin errores en la consola.
- [ ] Ningún fantasma camina por la puerta (fila 12, cols 13–14) tras iniciarse la partida, morir Pac-Man o ser comido: aparece directamente en (13,11).
- [ ] Cada fantasma espera visible en su celda de la pen hasta su temporizador y aparece fuera escalonado: blinky ~1.5 s, pinky ~3 s, inky ~4.5 s, clyde ~6 s.
- [ ] Al aparecer en (13,11), el fantasma persigue desde ese mismo instante con su personalidad del spec 01.
- [ ] Fantasma comido: reaparece al instante en (13,11) en modo chase, sin pasar por la pen.
- [ ] Al morir Pac-Man, los 4 regresan a la pen y re-aparecen escalonados en (13,11) (misma mecánica que la salida inicial).
- [ ] Ningún fantasma queda parado dentro de la pen más allá de su temporizador (los "ratos trancado" desaparecen).
- [ ] Ningún fantasma re-entra a la pen una vez fuera (la puerta sigue siendo muro con `leftPen`).
- [ ] Pac-Man sigue sin poder entrar a la pen.
- [ ] No queda ninguna referencia a `exitPenStep` ni a `startX`/`startY` en `src/`.

## Decisiones

- **Sí:** teletransporte a (13,11) al dispararse el temporizador — decisión del usuario; elimina de raíz el tramo puerta que mantenía a los fantasmas dentro tras cada muerte.
- **Sí:** diagnóstico previo (simulación Node del código actual): la salida a pie funciona; el problema es de diseño/percepción (resets escalonados tras cada muerte), no un bug de lógica.
- **Sí:** mantener el escalonado de 1.5 s — spec 01 lo dejó expresamente para su propio spec; este es ese spec y lo conserva.
- **Sí:** misma mecánica en las 3 vías de suelta (inicial, comido, muerte) — consistencia; el comido deja de re-caminar la puerta.
- **Sí:** seguir esperando visibles en la pen hasta el temporizador — el teletransporte ocurre al soltarlos, no al inicio.
- **No:** sacar a los 4 a la vez — se descarta al mantener el escalonado.
- **No:** tocar `MAZE`/`GHOST_STARTS`/render — la pen y su dibujado no cambian.
- **No:** re-activar el modo eyes para el regreso del comido — vestigial; el comido re-aparece directamente (más simple).

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Fantasma aparece en (13,11) justo cuando Pac-Man pasa por ahí → contacto instantáneo | Riesgo ya existente (hoy el fantasma llega caminando a la misma celda); se acepta tal cual |
| Dos fantasmas coinciden un instante en (13,11) (p. ej. comido justo cuando otro se suelta) | No existe colisión fantasma-fantasma (igual que hoy); se separan al perseguir |
| El "pop" del teletransporte puede verse brusco | Diseño elegido por el usuario; la alternativa (caminar) se descartó |

## Qué **no** está en este spec

- Scatter, Cruise Elroy, modo eyes, velocidades, timing de 1.5 s y cambios en `maze.js`/`render.js`. Cada uno, si algún día llega, va en su propio spec.
