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

- Guardar y cargar partidas: hoy no hay persistencia, pero la misma semilla da la misma historia.
- Mejorar la velocidad para llegar a 250 años por segundo reales.
- Una página tipo portfolio para compartir el HTML.
