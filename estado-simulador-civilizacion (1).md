# Crónica de una civilización: estado del proyecto para continuar

Proyecto de Santi: un simulador de civilización emergente en **un solo HTML**, sin dependencias externas. El texto de la interfaz está en español rioplatense.

Artifact publicado: https://claude.ai/artifact/7jmVNNYizXN8W7srX9fZgu

## Archivos

| Archivo | Qué es |
|---|---|
| `cronica-de-una-civilizacion.html` | **Última versión estable y publicada**, con celdas grandes (24 px) y figuras de tamaño fijo. |
| `mkeng.py` | Extrae el motor (todo el JS hasta `// ===========================  INTERFAZ`) a un `.js` para correrlo en node y expone `__T={newState,tick,S,WORLD,whereName}`. Uso: `python3 mkeng.py archivo.html eng.js`. |
| `runM.js`, `runS2.js` | Corren partidas en node, sin interfaz. `runM.js <geo> <años>` imprime los "primeros del mundo" y un resumen cada tanto. |
| `analisis-simulador-civilizacion.md` | El análisis original: modelos elegidos y diseño del primer motor. |

Para las pruebas visuales se usó **Playwright con Chromium** desde Python:
- `window.__civ = {S, tick, ensureStarted, camera, drainEvents, markers}` sirve para avanzar años y mover la cámara.
- Para medir cuadros por segundo se cuentan los `requestAnimationFrame`.

## Celdas grandes (terminado y publicado)

Pedido de Santi: "Todos construyen muy cerca y no se entiende nada. Que las celdas sean mucho más grandes y las personas mantengan su tamaño".

- Constantes: `CELL_PX = 24` (px de mundo por celda), `SPR = 8` (tamaño de figura), `TPX = 8` (resolución del caché de terreno, dibujado estirado), `SF = SPR/CELL_PX`.
- Casas, edificios, layout del pueblo, bandera, anillo, etiquetas, incendios, partículas y batallas se miden en `SPR`/`SF`.
- Layout: el anillo de edificios crece con la cantidad (`extrasR(m)`, `hutsR0(m)`), así las ciudades modernas no se apilan.
- Las casas no se construyen sobre agua ni sobre ríos (`riverMask` se llena al dibujar el caché; `isRiverPx`).
- Muelle corto que cruza la costa, con caminito de tierra si la costa queda lejos.
- Gente: paso `1.4*SF*1.5` celdas/s; tope `PER_CELL = 12`.
- Barcos: velocidad ×0.45, pesca a 0.5–3.2 celdas del puerto; estela y cañonazos en `SF`.
- Trenes y autos con período `L*900/SF*0.6` y `L*520/SF*0.6`; aviones y ascenso de cohetes a la mitad en vista detallada.
- Bug corregido: radio negativo en el ícono de batalla (y edades negativas en efectos) cuando `now` venía antes que `t0`; cortaba el cuadro entero.
- "Ir a nación" acerca hasta el nivel de detalle.
- Sin cambios a propósito: `DETAIL_ZOOM`, zoom máximo 16 y zoom del modo cine, porque miden tamaño de figura en pantalla.
- Probado en Chromium (Playwright): Tierra 🧬 años 7000 y 13000, mapa común año 3000; 55–61 fps.

## Resumen del motor actual

- **Población por cohortes.** Cada asentamiento guarda un `Int32Array` de edades de 0 a 100 y la media y varianza de 4 rasgos (fuerza, intelecto, cooperación, resistencia).
  - Mortalidad Gompertz-Makeham por edad.
  - Selección antes de los 45 años con piso de riesgo individual (`normCdf`).
  - Nacimientos con Verhulst por asentamiento y ecuación del criador sobre la media.
  - En guerra se aplica selección por truncamiento; al dividirse un pueblo, se separan los menos cooperativos (truncamiento sobre la cooperación).
- **Personajes icónicos** (`S.figs`): líderes, herederos, genios (cola de la campana), inventores, fundadores, rebeldes, profetas y generales, cada uno con su biografía.
- **Escala automática.** `S.scale` (la unidad del motor) sigue a la población para que el mundo tenga siempre unas 2500 unidades. `S.dispScale` salta de 1 a 10, 100, 1000 personas por figura, con histéresis. Los umbrales usan `SC(x)` y `U(x)`.
- **Distancias en km.**
  - `WORLD.km`: la Tierra usa 220 km por celda; los mapas comunes, 15, 30, 60 o 120 según "Tamaño del mundo".
  - `KF() = 25/km` convierte todas las distancias del motor.
  - `cellPeople()` convierte comida en personas con densidades reales: recolectores ~0,03 por km², agricultura con base `agriMult = 80×…`.
- **Rendimiento.**
  - Presupuesto de ~280 asentamientos, con `consolidateStep` que funde vecinos cuando hay demasiados. Los pioneros hacia tierras vacías siempre pueden salir.
  - El territorio se recalcula cada 2 años y las fusiones cada 3.
- **Árbol tecnológico.**
  - 33 inventos clásicos.
  - 16 modernos: brújula, método científico, vapor, ferrocarril, barco de vapor, medicina moderna, agricultura mecanizada, electricidad, telégrafo/radio, motor de combustión, autopistas, aviación, computación, cohetería, internet y satélites.
  - Después, una capa procedural a partir de 60000 de conocimiento.
  - Aceleradores del conocimiento: Kremer (según la población real), imprenta, método científico, computación, internet.
  - Slider "Ritmo en la Edad de Piedra" (`paleo`).
  - La agricultura solo se descubre donde hay tierra apta.
- **Organización política.** Banda, Tribu, Jefatura, Ciudad-estado, Reino, Imperio, Civilización, Estado industrial, Potencia mundial, Civilización espacial y Civilización planetaria.
- **Política.** Relaciones con rencores, incidentes fronterizos, alianzas, vasallaje, uniones, guerras con la ley de Lanchester, asabiya de Turchin, cultura según Axelrod y guerras civiles (solo entre pueblos asentados).
- **La Tierra.** Máscara Natural Earth en celdas de 2° (180×68), cortada en el Atlántico, con Beringia emergida, frío por latitud, desiertos, cordilleras, 21 ríos y nombres de regiones. El preset 🧬 arranca en el Omo con `ritmo 0.15` y `paleo 0.03`, y los sapiens ya conocen el fuego y la piedra tallada.
- **Línea de tiempo típica con 🧬 (en años simulados):**

  | Hecho | Año aproximado |
  |---|---|
  | Salida de África | ~210 |
  | Europa | ~765 |
  | América | ~4600 |
  | Agricultura | ~6600 |
  | Escritura | ~8100 |
  | Vapor | ~11900 |
  | Satélites | ~15800 |

  La población moderna llega a ~370 millones. Cuesta unos 9–10 ms por año, así que la velocidad real máxima ronda los 100 años por segundo.

## Resumen de la interfaz

- **Mapa:**
  - Cuatro vistas: político, terreno, recursos y creencias.
  - Nivel de detalle: de lejos se ven íconos; con `cssZ ≥ 1.1` aparecen personas, edificios y batallas animadas.
  - Solo se dibuja y anima lo que está en pantalla.
- **Herramientas** (arriba a la derecha):
  - 🖐️ mover.
  - 🖌️ pincel con 8 terrenos, río y quitar río, en 3 grosores (`paintCells` más `recomputeWorld`).
  - 👥 soltar un pueblo de 25, 250 o 2500 personas; si todavía no empezó la partida, define dónde nace la humanidad.
  - 🎬 modo cine.
- **Pantalla completa (⛶)** con HUD en forma de píldora: era, año, población, naciones, guerras, ▶/⏸ y velocidad.
- **Modo cine:** oculta los datos, muestra los puntos de inflexión como subtítulos y tiene 🎥 cámara automática.
- **Lienzo en blanco:** geografía "vacio", un océano para pintar.
- **Paneles:**
  - Naciones y detalle de nación, con sus personajes.
  - Personajes de la historia (Vivos / Grandes figuras), con ficha de vida.
  - Gráficos de población apilada por nación y de rasgos.
  - Crónica con filtro por defecto "Puntos de inflexión" (`e.imp`).
- **Lo visual por época:**
  - Carpas, chozas, casas de piedra y casas modernas.
  - Palacio, zigurat, mercado, fragua, granero, biblioteca, molino, fábrica, antena, rascacielos, aeropuerto y rampa de lanzamiento.
  - Murallas con torres, muelles y faro.
  - Caminos de tierra, empedrados, vías con trenes y autopistas con autos.
  - Canoas, veleros, galeones, vapores y cargueros, con rutas comerciales punteadas.
  - Aviones con estela, cohetes y satélites.
  - Batallas con garrotes, lanzas, espadas, arcos, mosquetes, cañones y tanques, e incendios de saqueo.

## Ideas pendientes que salieron en la charla

- ~~La historia se estancaba después de los satélites~~ (resuelto, ver abajo).
- Guardar y cargar partidas: hoy no hay persistencia, pero la misma semilla da la misma historia.
- Mejorar la velocidad para llegar a 250 años por segundo reales.
- Una página tipo portfolio para compartir el HTML.

## Rediseño de la interfaz y figuritas nuevas

- **Figuritas:** pixel art de 7×10 con borde suave (9×12), precalculado por color de nación en una tira (`personAtlas`): 4 tonos de piel × 3 colores de pelo × 2 pasos de caminata. Un solo `drawImage` por persona; `p.mv` marca si camina. Tamaño `PQ = 0.3` px de mundo por pixel. La gente ya no deambula por el agua.
- **Paleta y tipografía:** "mesa de cartógrafo": barra azul océano (`--chrome`), verde pasto para jugar (`--go`), dorado de corona para destacados (`--gold`). Tipografía redondeada del sistema (`--round`); la crónica sigue en serif. Modo claro y oscuro.
- **Barra superior fija:** título, era y año, ▶ Empezar/Pausar/Seguir, +1 año, velocidad en botones (`#speedChips`, sincroniza el `#speedSel` oculto) y Nueva partida.
- **Antes de jugar** (columna izquierda, `.setup`): 1) Elegí el mundo con tres tarjetas (historia humana 🧬, mundo nuevo 🌍, lienzo 🖌️) y chips de forma y tamaño que manejan los `select` ocultos; 2) Ajustá a la gente, con los sliders agrupados en `<details>` (la gente, la herencia, el entorno, el ritmo), palabras en los extremos y ayuda corta. Botón grande "Empezar la historia".
- **Durante la partida** (`body.playing`): la columna izquierda muestra el promedio de la humanidad y la crónica en vivo. Arriba del mapa van los números; debajo, los primeros inventos y las pestañas Naciones / Personajes / Gráficos (`openTab`). Los gráficos se redibujan al abrir su pestaña.
- En celular: el mapa va primero y el armado después; la barra superior deja de ser fija.

## La historia después de los satélites

**Diagnóstico original:** el árbol terminaba en satélites, la capa procedural era genérica e iba cada vez más lenta, la población chocaba contra un techo y la política se aquietaba. Resultado: 30–40 hechos importantes cada 500 años antes y 1–10 después.

**Inventos nuevos (16 + 3 de deportes)**, con `knowledgeMin` de 34k a 240k: física nuclear, armas nucleares, centrales nucleares, energías renovables, ingeniería genética, inteligencia artificial, estación espacial, base lunar, IA general, fusión, nanotecnología, colonia en Marte, ascensor espacial, terraformación y naves interestelares. Los deportes son juegos y deportes, competencias entre pueblos y deporte profesional. Suben la comida (`agriMult`, con `climateF()`), la medicina, lo militar, el alcance y Dunbar.

**IA:** multiplica el conocimiento (×1,4 y ×1,6) y la probabilidad de inventar (`aiF`). Cada nación bautiza a su IA (`n.aiName`, "XXX-4", que pasa a "XXX-Ω" con la IA general), y la IA puede figurar como autora de inventos en vez de una persona. Rara vez se sale de control.

**Conflicto nuclear** (`nuclearStep`): uso más probable si el país va perdiendo y según el carácter del líder. Hay disuasión mutua, tabú nuclear (cada bomba reciente divide la probabilidad), represalias, lluvia radiactiva durante 20 años, invierno nuclear si caen 4 bombas en 4 años, guerra fría, crisis de los misiles (que pueden terminar en tratado o en guerra) y escudo antimisiles con IA.

**Crisis** (`crisisStep`): calentamiento global según las emisiones de los países con motor (las renovables y la fusión lo reducen; la terraformación lo revierte). Trae más sequías, peores cosechas e inundaciones costeras desde 2,5 °C. También hay crisis económicas mundiales y pandemias que viajan en avión.

**Deportes** (`sportsStep`): cada pueblo inventa un deporte con nombre en su idioma o copia el del vecino. Los juegos entre pueblos mejoran las relaciones y dan una tregua. Con deporte profesional hay un campeonato mundial cada 4 años, con figuras "🏅 Deportista".

**Espacio** (`spaceStep`): población fuera de la Tierra proporcional a la de cada nación, independencia marciana, naves a otras estrellas sin techo, colonias contadas en `S.colonies` y el hallazgo de vida.

**Organización política nueva:** orbital, interplanetaria e interestelar (con eras). La capa procedural solo empieza con las naves interestelares y sus textos siguen hacia adelante.

**En el mapa:** hongo nuclear (`drawNukes`), estadio, central nuclear, reactor de fusión, molinos eólicos con paneles, centro de datos, ascensor espacial y más lanzamientos cuando hay bases lunares.

**Línea de tiempo con 🧬:**

| Hecho | Año aproximado |
|---|---|
| Física nuclear | 14800 |
| Bomba | 15500 |
| IA | 15960 |
| Luna | 16300 |
| IA general | 16500 |
| Marte | 17200 |
| Primer uso nuclear | 17300 |
| Nave interestelar | 18700 |
