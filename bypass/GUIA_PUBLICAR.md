# Publicar Bypass en Hostinger

Bypass es una web estática: unos pocos archivos, sin base de datos ni nada que instalar. Se sube en unos 10 minutos.

## 0. Antes de comprar

- Comprueba que el nombre está libre como marca: **OEPM** (oepm.es) y **EUIPO**, clases 9 y 42.
- Elige el dominio. Si `bypass.es` o `.com` están cogidos, prueba variantes: `bypassplanos.com`, `getbypass.app`…

## 1. Qué contratar

- **Hosting web, plan Premium o Business**. Incluyen dominio gratis el primer año, correo profesional y certificado SSL.
- **No** contrates el «Creador de sitios web» (Website Builder): necesitas poder subir tus propios archivos.
- **Truco:** contrata 24 o 48 meses. La renovación sube bastante (de ~3 €/mes a ~10 €/mes) y así mantienes el precio bajo más tiempo.

## 2. Poner tu dominio en los archivos (1 minuto)

En `index.html` hay 3 sitios donde pone `TU-DOMINIO` (la vista previa al compartir el enlace por WhatsApp o correo).
Cámbialos por tu dominio sin `https://` ni barra final. Por ejemplo, `TU-DOMINIO` → `bypass.es`.

> Si me dices el dominio, te lo dejo hecho.

## 3. Subir los archivos

1. En Hostinger: **hPanel → Sitios web → Administrar → Administrador de archivos**.
2. Entra en la carpeta **`public_html`**.
3. Borra lo que venga de ejemplo (`default.php` o `index.php`).
4. Sube **el contenido** del zip `bypass-web.zip`. Hostinger lo descomprime con clic derecho → *Extraer*. Tienen que quedar así:

```
public_html/
├── index.html      ← la web y el editor
├── app.html        ← redirige al editor (por si alguien tenía este enlace)
├── favicon.svg     ← icono de la pestaña
├── icon-180.png    ← icono al guardarlo en el móvil
├── icon-512.png
├── og.png          ← imagen al compartir el enlace
└── robots.txt      ← permiso para que Google la encuentre
```

## 4. Seguridad (https)

**hPanel → Seguridad → SSL** → activar el certificado gratuito. Tarda unos minutos.
Activa también **«Forzar HTTPS»**.

## 5. Comprobar

- Entra en `https://tu-dominio`: sale la web.
- **Pruébalo gratis** → se abre el editor y se puede dibujar.
- Pega el enlace en WhatsApp: debe salir la imagen con «El plano de tu unidad, sin el ruido del P&ID».
  Si no sale a la primera, espera unos minutos: WhatsApp guarda en caché la vista previa.

## 6. Correo profesional (opcional)

**hPanel → Correos → Crear cuenta**: por ejemplo `contacto@tu-dominio`.
Sirve para el botón «Consúltanos» de la versión Planta y para recibir la lista de espera.

## 7. Actualizar la web más adelante

Cada vez que haya una versión nueva, se sube el `index.html` nuevo encima del anterior.
Los planos de los usuarios **no se pierden**: viven en el navegador de cada uno, no en el servidor.

## Lo que falta para cobrar (cuando quieras)

- **Pago de 19 €:** crear un «Payment Link» en Stripe y ponerlo en el botón «Conseguir Pro».
- **Lista de espera:** crear un formulario en Formspree (gratis) y poner su dirección en `WAITLIST_URL` (en `index.html`).
- **Solicitudes de demo de empresas:** otro formulario de Formspree, con su dirección en `LEADS_URL` (en `index.html`).

Las dos cosas se configuran en 5 minutos y no necesitan servidor.
