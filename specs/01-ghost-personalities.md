# SPEC 01 — Personalidades de los 4 fantasmas

> **Status:** Approved
> **Depends on:** —
> **Date:** 2026-10-17
> **Objective:** El juego spawnea 4 fantasmas, cada uno con su propia lógica de persecución estilo arcade, y uno de ellos (hunter) persigue a Pac-Man de forma agresiva y directa.

## Scope

**In:**

- Expandir `GHOST_STARTS` (`src/js/maze.js`) de 2 a 4 fantasmas: `hunter`, `ambusher`, `inky`, `shy`.
- Spawneo clásico: `hunter` fuera de la pen en `(13,11)`; los otros 3 dentro de la pen.
- Reordenar `GHOST_COLORS` (`src/js/render.js`) para que el índice coincida con el orden de `GHOST_STARTS`: rojo, rosa, cian, naranja.
- Reescribir `decideGhost` (`src/js/game.js`) con una lógica de "celda objetivo" por personalidad, eligiendo en cada intersección la dirección que minimiza la distancia de Manhattan a ese objetivo.

**Out of scope (for future specs):**

- Modo scatter/chase (períodos alternos de persecución y huida a esquinas).
- Power pellets, fantasmas asustados y fantasmas comidos.
- Release escalonado de la pen (timer de salida, puerta como filtro).
- Velocidades distintas por fantasma ni por nivel.

## Data model

Ninguna estructura nueva. Cambian valores existentes:

```js
// src/js/maze.js
const GHOST_STARTS = [
  { x: 13, y: 11, kind: 'hunter' },   // fuera de la pen, arriba del centro
  { x: 14, y: 14, kind: 'ambusher' }, // pen, centro
  { x: 12, y: 14, kind: 'inky' },     // pen, izquierda
  { x: 16, y: 14, kind: 'shy' },      // pen, derecha
];
```

```js
// src/js/render.js — orden alineado con GHOST_STARTS
const GHOST_COLORS = [ '#ff0000', '#ffb8ff', '#00ffff', '#ffb852' ];
// hunter=rojo, ambusher=rosa, inky=cian, shy=naranja
```

```js
// src/js/game.js — constantes de las personalidades
const PINKY_AHEAD = 4;            // ambusher apunta 4 celdas frente a Pac-Man
const INKY_AHEAD = 2;             // inky usa 2 celdas frente a Pac-Man
const SHY_THRESHOLD = 8;          // shy huye si distancia Manhattan <= 8
const SHY_CORNER = { x: 1, y: 29 }; // esquina a la que huye shy
```

Cada fantasma ya lleva `kind` (string). Se elimina el valor `'random'`; el `else` de `decideGhost` queda como red de seguridad con movimiento aleatorio para cualquier `kind` desconocido.

## Target de cada personalidad

En cada intersección (celda alineada), `decideGhost` calcula una celda objetivo y elige la dirección válida (sin invertir la actual, salvo callejón) con menor distancia de Manhattan a esa celda:

| kind       | Celda objetivo                                                                |
| ---------- | ----------------------------------------------------------------------------- |
| `hunter`   | Celda exacta de Pac-Man (`Math.round` de x,y). Persecución directa, agresiva. |
| `ambusher` | 4 celdas en la dirección actual de Pac-Man. Emboscada donde Pac-Man irá.      |
| `inky`     | Punto reflejado: `2 * (Pac-Man + 2 en su dirección) - posición de hunter`.    |
| `shy`      | Si distancia Manhattan a Pac-Man > 8: Pac-Man. Si no: `SHY_CORNER`.           |

Detalle de `inky`: sea `ahead = pacman + 2 * DIRS[pacman.dir]`, el objetivo es `2 * ahead - hunter`. Si `hunter` no existe en `game.ghosts`, usar la celda de Pac-Man (degrada a persecución directa).

## Implementation plan

1. En `src/js/maze.js`: reemplazar `GHOST_STARTS` por los 4 fantasmas del data model. Verificar en consola que `window.GHOST_STARTS.length === 4`. El juego ya dibuja 4 fantasmas (el loop de render es genérico).
2. En `src/js/render.js`: reordenar `GHOST_COLORS` a `[rojo, rosa, cian, naranja]`. Verificar: el fantasma que sale fuera de la pen es rojo; los de pen son rosa, cian, naranja.
3. En `src/js/game.js`: añadir las 4 constantes (`PINKY_AHEAD`, `INKY_AHEAD`, `SHY_THRESHOLD`, `SHY_CORNER`) junto a `PACMAN_SPEED`/`GHOST_SPEED`.
4. En `src/js/game.js`: extraer de `decideGhost` la elección por distancia de Manhattan a un objetivo en una función `towardTarget( choices, g, target )`. `hunter` usa el target = celda de Pac-Man (comportamiento actual, ahora vía la función). Verificar: el rojo sigue persiguiendo igual que antes.
5. En `src/js/game.js`: implementar el target de `ambusher` (4 celdas frente a Pac-Man en su dirección). Verificar: el rosa corta el paso por delante de Pac-Man en vez de ir a su celda.
6. Implementar el target de `inky` (reflejo con posición de hunter). Verificar: el cian toma trayectorias poco directas que dependen de la posición del rojo.
7. Implementar la lógica de `shy` (persigue si distancia > 8, si no va a `SHY_CORNER`). Verificar: el naranja se aleja hacia abajo-izquierda cuando Pac-Man se acerca.
8. Eliminar el `kind: 'random'` residual y dejar el `else` aleatorio solo como fallback. Verificar que el juego corre sin errores de consola en una partida completa.

## Acceptance criteria

- [ ] El juego carga sin errores en consola y en la pantalla hay 4 fantasmas.
- [ ] `hunter` arranca en `(13,11)` fuera de la pen; los otros 3 arrancan en `(12|14|16, 14)` dentro de la pen.
- [ ] Colores por índice: primer fantasma rojo, segundo rosa, tercero cian, cuarto naranja.
- [ ] En cada intersección, ningún fantasma invierte su dirección salvo que sea un callejón sin salida.
- [ ] `hunter` siempre elige la dirección que más reduce la distancia de Manhattan a la celda de Pac-Man.
- [ ] `ambusher` elige la dirección que más reduce la distancia a la celda 4 celdas frente a Pac-Man en su dirección actual.
- [ ] `inky` elige la dirección que más reduce la distancia al punto `2 * (pacman + 2 en su dirección) - hunter`.
- [ ] `shy` persigue a Pac-Man cuando la distancia Manhattan es mayor que 8; cuando es 8 o menos, se dirige a `(1,29)`.
- [ ] Chocar con cualquiera de los 4 fantasmas sigue restando una vida (comportamiento de colisión sin cambios).

## Decisions

- **Yes:** personalidades clásicas del arcade (hunter/ambusher/inky/shy). Probadas, bien diferenciadas y documentadas.
- **No:** scatter/chase. Añade timer y esquinas por fantasma; se puede sumar después sin tocar estas lógicas.
- **No:** power pellets y fantasmas asustados. Es otro spec; aquí chocar siempre mata.
- **No:** release escalonado de la pen. El foco es la IA, no la coreografía de salida.
- **Yes:** nombres por conducta (`hunter`, `ambusher`, `inky`, `shy`) en vez de colores. El código describe intención, no pintura.
- **Yes:** Inky con el vector clásico reflejado sobre hunter. Es lo que lo hace "impredecible" de verdad; un random puro no aporta.
- **Yes:** un solo mecanismo de decisión (minimizar Manhattan a una celda objetivo variable por kind). Un solo camino de código que probar.
- **Yes:** `hunter` sale fuera de la pen desde el primer frame. Es el agresivo; su valor es presionar ya.
- **No:** replicar el bug de overflow de Pac-Man "arriba" del arcade (Pinky apuntaba arriba-izquierda). Es un bug histórico, no una feature.
- **Yes:** umbral de `shy` en 8 celdas y esquina `(1,29)`. Celdas del laberinto real de este repo; fácil de ajustar si se siente mal al jugar.
- **Sí (parcial):** el `else` aleatorio se conserva como fallback para `kind` desconocido; no es un comportamiento oficial, es robustez.

## Risks

| Risk                                                       | Mitigation                                                                       |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `inky` degrada si se toca el `kind` de hunter              | Fallback documentado: si no hay `hunter` en `game.ghosts`, target = celda de Pac-Man. |
| Manhattan ignora el túnel: objetivo "al otro lado" se ve lejos | Acceptable; elige peor dirección en el túnel pero no rompe nada. Ajustar al jugar. |
| Esquina de `shy` `(1,29)` podría quedar rodeada            | Es celda de pasillo del laberinto actual; si se cambia el laberinto, revisar.   |

## What is **not** in this spec

- Scatter/chase y esquinas de scatter de Blinky/Pinky/Inky.
- Power pellets, modo asustado, fantasma comido, regreso a la pen.
- Release de pen con timer.
- Diferencia de velocidad por fantasma o por nivel.
- Cualquier cambio en Pac-Man, scoring o vidas.

Cada uno de esos, si aterriza, va en su propio spec.
