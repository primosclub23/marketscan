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
  - **Cuadraditos de conexión**: las bocas se pintan como cuadraditos. Libres: amarillos huecos. Conectadas: verdes. Objetivo: amarillo relleno con anillo. Salen todos al dibujar o arrastrar; con la herramienta Mover salen solo los del equipo bajo el ratón (`hoverId`). Un clic en uno empieza una línea desde esa boca.
  - Con `Alt` se desactiva el imán.
- **Flujo animado** (botón «Ver flujo» o tecla `A`): rayitas que avanzan en el sentido de cada línea (`drawFlow()`).
  - `flowIntervals()` calcula qué tramos de cada línea tienen flujo. Se cortan en válvulas cerradas y ciegos, y reviven donde entra otra línea con flujo (por ejemplo, el retorno de un bypass).
  - Una línea que sale de otra hereda el flujo si su punto de partida lo tiene.
  - En modo maniobra usa el estado de la maniobra; en modo dibujo, el de las válvulas del plano.
  - Las mangueras desconectadas no se animan. No sale en las exportaciones.
- **Modo animación** (tercer botón de modo, `mode==='anim'`):
  - Las tuberías con flujo se dibujan con brillo (`filter #glow`) y partículas o corriente en movimiento (`fluidPath()`). Lo que no tiene flujo se ve apagado.
  - Velocidad con la variable CSS `--fd`; pausa con la tecla `P`.
  - **Simulador**: `SIM` guarda las válvulas, purgas, mangueras, ciegos y PSV que se abren o cierran tocándolos. No modifica el plano.
  - Estado de partida (`animBase`): las válvulas del plano o la maniobra activa (aislamiento).
  - Las purgas abiertas sobre una línea con flujo muestran la descarga. La PSV corta el flujo a antorcha salvo que esté disparada.
- **Nivel en depósitos** (modo animación): `drawLiquids()` dibuja el líquido recortado con la forma del equipo (`clipPath`), con ola en la superficie. `liqTick()` (requestAnimationFrame) mueve el nivel hacia su objetivo:
  - entra y sale líquido: 50 %, estable;
  - solo entra: 92 %, se llena;
  - solo sale: 4 %, se vacía.

  Solo cuentan como salida de líquido las bocas de la mitad de abajo. El color es el del fluido que entra.
- **Estado de válvulas en maniobra**: `manState()` completa el estado de todas las válvulas (las que no tocan los pasos toman el del plano). En modo maniobra y en la hoja (`SHOWST`) se rellenan de color: `VC()` cerrada y `VO()` abierta. El convenio rojo/verde se puede invertir y se guarda en `localStorage` (`bypass.vconv`).
  - Visor paso a paso (`manView`): estado tras el paso k; los pasos posteriores se ven atenuados.
  - Panel «Estado de válvulas»: listas de cerradas y abiertas con su tipo. Tocar una centra el plano en ella.
- **Red de flujo por entradas** (sustituye al cálculo antiguo por sentido de dibujo):
  - `flowNet()` arma un grafo no dirigido. Los nudos son extremos de tramo, bifurcaciones y bocas. Las válvulas cerradas, los ciegos y las PSV sin disparar cortan; los equipos y las válvulas abiertas conectan sus bocas; las mangueras y las purgas abiertas también.
  - `flowRun()` recorre el grafo desde las entradas y orienta cada tramo según por dónde le llega el flujo. El sentido en que se dibujó cada línea ya no importa.
  - Entradas: `S.flowSrc` (se guardan con el plano) o, si no hay, `autoSources()`. Automáticas: límites de batería con algo conectado en la punta de la flecha (salvo los que empiezan por «A …») y líneas que empiezan sueltas.
  - En el modo animación, «Elegir entradas» activa `animPick`: tocar un extremo de línea o manguera, un límite o un equipo pone o quita esa entrada.
  - El nivel de los depósitos usa `vesselFlows()` y `hasExit()`: solo cuenta la salida si el flujo llega a algún sitio, no si acaba en una válvula cerrada.
- **Retenciones con sentido**: el símbolo lleva una flecha. El sentido permitido es el +x local, con giro y espejo aplicados (`chkDir()`). Al soltarla sobre una línea se orienta según el sentido de dibujo (`tangentAt()`); con `F` o el inspector se invierte. En `flowNet()` la línea se parte en la retención y los dos lados se unen con un enlace de un solo sentido (`linkDir`). Si las líneas llegan a sus bocas, el enlace también es de un solo sentido.
- **Flechitas de sentido**: `smallArrows()` pone flechas pequeñas cada ~120 unidades, sin tapar símbolos (`blockedAt()`). `buildArrows()` las orienta con el flujo real (`flowRun`). En los tramos sin flujo usa el sentido de diseño (`designRun()`: todo abierto y las PSV disparadas) y, si no, el de dibujo. Se desactivan por línea con «Sentido de flujo: Sin flecha».
- **Selección múltiple** (`msel`): con el ratón, arrastrar en vacío hace un rectángulo de selección; Mayús+clic añade o quita; Ctrl+A selecciona todo. Se mueve en grupo (`mmove`); Ctrl+D duplica; Supr borra; las flechas desplazan. Ctrl+C / Ctrl+V usan `localStorage` (`bypass.clip`), así que sirven entre planos. En táctil, arrastrar en vacío sigue moviendo el plano.
- **Símbolos nuevos**: eyector (`J`), turbina (`TB`), filtro (`F`) y filtro en Y (en línea, no se cierra).
- **Numerar válvulas** (botón «Numerar» en 04 Válvulas): pone V-01, V-02… a las válvulas y retenciones sin tag, en orden de lectura.
- **Letras A/C y blanco y negro**: en maniobra y en la hoja, cada válvula lleva una placa «A» o «C». El convenio `bn` (B/N) pinta las cerradas rellenas de tinta y las abiertas huecas.
- **PDF** (botón «PDF»): `openPrint()` monta una página A4 o A3, horizontal o vertical según la forma del plano, con el plano y un cajetín (plano, maniobra y permiso si se está en modo maniobra, fecha, revisión y formato). El tamaño de página se fija con `@page` (`#pgsize`) y se guarda con «Imprimir → Guardar como PDF». En B/N los fluidos pasan a grises oscuros sobre fondo blanco.
- **Formato de archivo `.bypass`**: JSON `{app:'bypass', v:1, doc:S}`.

## Móvil y tablet

- Por debajo de 700 px: barra superior compacta (los tres modos siempre visibles), panel lateral como cajón que se abre con ☰ (`.app.drawer`) y se cierra al tocar el plano o elegir una herramienta. El botón «Ver flujo» pasa a la izquierda.
- Zoom con dos dedos: `PTRS` guarda los punteros y el arrastre de tipo `pinch`.

## Pendiente o ideas

- Más símbolos.
- Gestos táctiles con dos dedos para el zoom.
