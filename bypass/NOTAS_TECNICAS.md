# Bypass: notas técnicas

Editor web de esquemas de proceso para operadores. Es un único archivo, `index.html`: HTML, CSS y JS sin dependencias ni paso de compilación. Las fuentes se cargan de Google Fonts.

## Estructura del código (`index.html`)

- **`SYM`**: catálogo de símbolos. Cada uno tiene:
  - `n`: nombre;
  - `g`: grupo (`eq` equipos, `va` válvulas, `ot` anotaciones);
  - `tag`: prefijo del tag automático;
  - `bb`: caja que lo envuelve, centrada en (0,0);
  - `d(color, item)`: devuelve el SVG del símbolo.

  Para añadir un equipo basta con una entrada nueva: la paleta se genera sola.
- **Estado `S`**: `{name, rev, fluids:[{id,name,color}], items:[...]}`. Los tipos de item son:
  - `line`: `{pts:[[x,y]...], fluid, dash, weight:'n'|'b', arrow}`
  - `sym`: `{kind, x, y, rot, flip, scale, tag, fluid}`
  - `text`: `{x, y, text, size, fluid}`
  - `legend`: `{x, y}`. Se rellena sola con los fluidos usados.
  - `title`: `{x, y}`. Es el cajetín y toma `S.name` y `S.rev`.
- **Fluido `proc`** con `color:'ink'`: se dibuja blanco en el tema Noche y negro en el tema Papel.
- **Rejilla** de 10 unidades. Todo se ajusta a ella, y por eso las bifurcaciones se detectan de forma exacta: el punto final de una línea queda sobre el tramo de otra.
- **Dibujo**: `renderItems(paleta, hit)` genera el SVG del plano. Se usa igual en pantalla y al exportar (SVG y PNG a 2x).
- **Historial**: `commit()` guarda una copia JSON (hasta 150 pasos) y además hace el autoguardado en `localStorage` (`bypass.doc.v1`).
- **Comportamiento de la edición**:
  - La línea es ortogonal por defecto; con `Alt` el ángulo es libre.
  - La línea se termina con doble clic, `Enter` o `Esc`, o automáticamente al tocar otra línea o un equipo.
  - Una válvula colocada sobre un tramo vertical gira sola y toma el color del fluido de esa línea.
  - Con una línea seleccionada: los cuadrados mueven un vértice y los rombos mueven un tramo entero. Si el tramo es el primero o el último, se crea un codo nuevo.
- **Saltos en los cruces**: `hopPath()` hace que los tramos horizontales salten en arco sobre los verticales de otras líneas. Los cruces se calculan en `renderItems` (`VSEGS`).
- **Válvula abierta / cerrada**: `it.closed` rellena el cuerpo de la válvula (convenio de válvula cerrada). Solo en manual, automática, motorizada y tres vías. Tecla `C` o el inspector.
- **Modo maniobra** (botón «Maniobra» de la barra superior):
  - Datos en `S.manis = [{title, permit, by, steps:[{phase:'a'|'v', target, act, note, done, x, y, vert}]}]`. `a` = aislamiento, `v` = vuelta a servicio (normalizar).
  - Las acciones están en `ACTS`. Cada una puede tener su inversa (`inv`), el estado de válvula que deja (`st`) y si pone o quita candado (`lock`/`unlock`). `actsFor(item)` decide qué acciones se ofrecen según lo que se toque.
  - `manState()` calcula cómo queda el plano tras los pasos: válvulas cerradas en rojo, ciegos en las líneas y candados. Lo dibuja `drawManOverlay()`, junto con los números de paso. El dibujo original no se modifica.
  - `buildReturn()` genera la vuelta a servicio: recorre el aislamiento al revés con las acciones inversas.
  - `openSheet()` monta la hoja de maniobra A4. Se imprime con `window.print()` y unos estilos `@media print`.
- **Purgas ¾"** (`venteo`, `drenaje`, grupo `pm`): una válvula pequeña con su tramo y tapón. Se colocan cerradas por defecto. Se montan solas sobre una línea o en una boca de equipo, y su extremo es una boca donde se engancha una manguera.
- **Mangueras** (`type:'hose'`, `{kind:'agua'|'n2'|'vapor'|'metal', a, b, tag}`): se dibujan como una curva que cuelga. En modo maniobra admiten «Conectar» y «Desconectar»; desconectada se ve en transparente.
- **Imán (enganche tipo lego)**:
  - `PORTS` define las bocas de cada símbolo y `portsOf()` las pasa a coordenadas del plano, teniendo en cuenta giro, espejo y escala.
  - `magnet()` busca, por este orden: bocas, extremos de línea o manguera y, si no hay nada, un punto sobre una línea (bifurcación).
  - `snapSym()` monta las válvulas y purgas sobre la línea y las gira; en los equipos, encaja sus bocas en los extremos libres.
  - `findGlue()` y `applyGlue()` hacen que, al mover un equipo, las líneas y mangueras conectadas le sigan, manteniendo los ángulos rectos con codos automáticos (`moveEnd()`).
  - **Imán por cuerpo**: si llevas una línea o manguera dentro de un equipo, `bestPort()` elige la boca que la deja más recta. Primero las que quedan alineadas con el origen (sin codos), luego las que se encaran con un solo codo, y entre iguales la más cercana al cursor. Para elegir a propósito una boca concreta, acércate a ella desde fuera del equipo.
  - Con `Alt` se desactiva el imán.
- **Formato de archivo `.bypass`**: JSON `{app:'bypass', v:1, doc:S}`.

## Pendiente o ideas

- Selección múltiple.
- Exportar a PDF en A3.
- Más símbolos.
- Gestos táctiles con dos dedos para el zoom.
