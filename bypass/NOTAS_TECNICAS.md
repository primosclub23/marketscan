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
- **Formato de archivo `.bypass`**: JSON `{app:'bypass', v:1, doc:S}`.

## Pendiente o ideas

- Saltos de línea en los cruces.
- Conexión magnética a las bocas de los equipos.
- Selección múltiple.
- Exportar a PDF en A3.
- Más símbolos.
- Gestos táctiles con dos dedos para el zoom.
