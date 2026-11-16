# Sazón — notas técnicas

Planificador de menú semanal (desayuno, comida, merienda opcional y cena) con lista de la compra. Nombre provisional.

## Arquitectura
- App web estática de un solo archivo (`index.html`): HTML + CSS + JS sin dependencias ni build. Vive en `sazon/` dentro del repo `marketscan` hasta que tenga repo propio.
- PWA: `manifest.webmanifest`, `sw.js` (red primero, caché de respaldo) e iconos PNG.
- Datos del usuario solo en `localStorage` (clave `sazon.v1`). No hay backend ni cuentas.
- Estilo: sistema visual propio en modo beige (Playfair Display / Inter / Space Mono, bordes 0.5px, botones pill).

## Decisiones no obvias
- **Precios estimados, no reales.** Cada ingrediente lleva un precio orientativo escrito a mano en `ING`. No se consulta ningún supermercado. La app lo dice en la lista. Los básicos de despensa (aceite, sal, especias) no suman al total.
- **"Compra guiada" en vez de "comprar online".** Ningún súper ofrece una API pública para volcar un carrito, así que se abre el buscador web de la tienda elegida producto a producto (`STORES`). Las URLs de búsqueda no se han podido verificar desde el entorno de desarrollo (red bloqueada): comprobarlas en un móvil real.
- **Sin logos de supermercados** (marcas registradas): solo el nombre en texto.
- **Ilustración con emoji** sobre fondo tintado en lugar de fotos: no hay banco de fotos con licencia.
- **Calorías y macros** son aproximaciones por ración, no cálculo nutricional.
- El generador elige platos al azar sin repetir, respetando dieta y exclusiones, y después sustituye los más caros hasta entrar en presupuesto (`fitBudget`). `syncPlan` ajusta el plan cuando cambian los ajustes sin regenerarlo entero.
- La semana es siempre la actual (lunes–domingo); todavía no hay navegación entre semanas.

- **Evitar ingredientes**: además de los grupos predefinidos (`AVOID`), `S.avoidIng` guarda ids sueltos elegidos con el buscador. Solo busca en el catálogo de la app (un ingrediente que no sale en ninguna receta no hace falta evitarlo).
- **Recetas propias**: se guardan en `S.custom` y se inyectan en `R`/`RID` al arrancar. Los ingredientes que no existen en `ING` (o que se dan en otra unidad) se crean en `S.cing` con precio 0 y pasillo "Otros"; no suman al precio estimado. Sin grupo de dieta: los filtros vegetariano/vegano/sin gluten no pueden detectarlos. El texto del usuario se escapa al guardar (`esc`) porque todo se pinta con `innerHTML`. `pick` da un 50 % de preferencia a las recetas propias aún no usadas.
- **Táper**: `S.tup` marca casillas día+comida (`"0l"`). En esas casillas `pool` solo admite recetas con `tp` (1 se recalienta, 2 se come fría; mapa `TP`). Se activa por día en ajustes/onboarding (solo comida) o con el botón de la cabecera de Comida/Cena.

## Bugs y lecciones
- El estado inicial compartía por referencia los objetos de `DEF` (`Object.assign` superficial), así que "borrar mis datos" no restauraba los valores por defecto. Arreglo: `fresh()` clona `DEF`.
- Al cambiar de dieta, un plato sustituido heredaba la marca "cocinado" del anterior. Arreglo: `syncPlan` borra `cooked` y `serv` de la casilla al sustituir.
