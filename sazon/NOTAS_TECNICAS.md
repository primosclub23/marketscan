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

## Bugs y lecciones
- Al cambiar de dieta, un plato sustituido heredaba la marca "cocinado" del anterior. Arreglo: `syncPlan` borra `cooked` y `serv` de la casilla al sustituir.
