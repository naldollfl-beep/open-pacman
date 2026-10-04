# AGENTS.md — Pac-Man (JS vanilla)

Clon de Pac-Man usado para practicar spec-driven development (ver `README.md`).

## Tooling (no hay)

- No hay `package.json`, ni npm, ni bundler, ni lint/typecheck/tests. No inventes pasos de build.
- Ejecutar: abrir `src/index.html` directamente en el navegador (funciona desde `file://`).
- Verificar: prueba manual en navegador con la consola abierta y sin errores. No hay suite de tests; los criterios de aceptación de los specs son checklists booleanos que un humano verifica en el navegador.

## Arquitectura

Cuatro scripts planos cargados en este orden exacto vía etiquetas `<script>` en `src/index.html`:

1. `src/js/maze.js` — laberinto prístino + constantes (`MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`)
2. `src/js/game.js` — estado y reglas (`createGame`, `update`)
3. `src/js/render.js` — dibujado en canvas (`draw`)
4. `src/js/main.js` — bucle, teclado, overlay

- Sin módulos ES; los archivos se comunican mediante globals en `window.*`. Añadir un archivo implica añadir su etiqueta `<script>` en `index.html` en orden de dependencia.
- Que funcione con doble clic desde `file://` solo es posible porque no hay módulos ni fetch. No refactorees a módulos ES sin añadir un dev server.
- `MAZE` es la plantilla prístina; `createGame()` la copia a `game.grid` en cada partida. Nunca mutar `MAZE` — la lógica del juego y el render leen `game.grid`.
- Invariante entre archivos: el tamaño del canvas en `src/index.html` (560×620) equivale a las dimensiones del laberinto (28×31 celdas) × `TILE = 20` de `render.js`. Si cambias una, actualiza las otras.

## Flujo spec-driven

Es el propósito declarado del repo. Los skills `spec` y `spec-impl` están instalados (`.agents/skills/`, pineados en `skills-lock.json`):

- Las features nuevas empiezan como specs en `specs/NN-slug.md` (plantilla: `.agents/skills/spec/template.md`), estado `Draft`.
- Solo el humano cambia el estado de un spec a `Approved` — el agente nunca lo hace.
- `spec-impl` implementa un spec aprobado en la rama `spec-NN-slug`, paso a paso con pausas de revisión; rechaza cualquier spec que no esté en estado Approved.
- Los comentarios del código citan el spec del que viene el cambio, p. ej. `// ... (spec 04)`. Sigue haciéndolo.

## Convenciones

- Español: los comentarios del código, los textos de UI y los specs se escriben en español. Mantenlo.

## Gotchas

- Montaje WSL de un disco Windows: los archivos `.agents/skills/**` aparecen como totalmente modificados en `git status` por churn de CRLF/LF (contenido idéntico, todas las líneas difieren). No commitees ese churn ni "arregles" los line endings de todo el repo sin preguntar.
