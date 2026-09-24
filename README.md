# Página de cobro para llavero NFC

Página de una sola pantalla con los datos de transferencia de un cliente. El
llavero NFC guarda el enlace, el cliente acerca el teléfono y la página se abre
con los datos listos para copiar y pegar en la app del banco.

Todo vive en un solo archivo, `index.html`, sin dependencias ni compilación.
Se publica con GitHub Pages.

## Idea práctica: dos personas, dos mensajes

El llavero NFC se usa en la calle, con apuro. El caso más claro —y el de los
llaveros para Uber más adelante— es este:

1. El pasajero (el **tercero**) se sube al auto.
2. El conductor (el **cliente**) le muestra el llavero.
3. El pasajero acerca el teléfono, copia los datos y hace la transferencia
   en la app de su banco.
4. El pasajero toca el ícono verde de WhatsApp.
5. El conductor recibe ese mensaje y se entera de que le van a transferir.

Hay dos lectores. No se les puede decir lo mismo en el mismo texto.

| Quién | Dónde lo lee | Qué tiene que entender |
| --- | --- | --- |
| El tercero (quien paga) | La página que abre el NFC | Que debe subir el voucher o mostrárselo a quien le está cobrando (en Uber: al conductor). |
| El cliente (quien cobra) | El WhatsApp que le llega | Que le van a hacer una transferencia. No que el pago ya está hecho. |

Por eso la página lleva el aviso con el pin (`aviso-voucher`):

> 📌 Importante: sube tu voucher por WhatsApp o muéstraselo a quien te está cobrando.

Eso lo lee el tercero, en su teléfono, mientras copia los datos. En un llavero
para Uber se puede cambiar “a quien te está cobrando” por “al conductor”.

El texto prellenado de WhatsApp (`CONTACTO.mensajeWhatsapp`) va dirigido al
cliente, en primera persona, porque ese chat lo recibe él:

```js
mensajeWhatsapp: 'Hola Luis, te aviso que haré la transferencia.',
```

Si el WhatsApp dice “sube tu voucher”, el conductor cree que le hablan a él.
Si dice “envío el comprobante”, parece que el pago ya está hecho y se presta
para malentendidos.

Al armar llaveros para Uber, copiar esta plantilla, poner los datos del
conductor en `CLIENTE` / `CONTACTO` / `CUENTAS` y dejar esa separación:
aviso de voucher en la página, aviso de transferencia en el WhatsApp.

## Hacer un llavero para un cliente nuevo

1. Crear el repositorio nuevo a partir de este (botón **Use this template** en
   GitHub, o copiar `index.html` a mano).
2. Abrir `index.html` y buscar el bloque **DATOS DEL CLIENTE**, cerca del inicio
   del `<script>`. Está delimitado así:

   ```js
   /* ==================================================================
      DATOS DEL CLIENTE
      ...
      ================================================================== */
   ```

3. Cambiar los tres bloques que hay adentro: `CLIENTE`, `CONTACTO` y `CUENTAS`.
4. Publicar con GitHub Pages (ver más abajo).
5. Grabar la URL en el llavero.

**Debajo de la línea `FIN DE LOS DATOS DEL CLIENTE` no hay que tocar nada.** Si
algo obliga a modificar el código de más abajo, conviene arreglarlo en la
plantilla para que el próximo llavero tampoco lo necesite.

## Los tres bloques

### `CLIENTE`

| Campo | Qué es |
| --- | --- |
| `titulo` | Lo que se ve en la pestaña del navegador y al compartir el enlace. |
| `empresa` | Encabezado de la tarjeta. Se escribe como se lee, no en mayúsculas. |
| `razonSocial` | Nombre legal completo, tal como está en el SII. |
| `nombreCorto` | Versión corta para el formulario del banco. |
| `rut` | Se escribe una sola vez, con puntos y guion. |
| `fondo` | Fondo de la página: una imagen entre `url('...')`, o un color o degradado. |
| `acento` | Color de los botones. |

Sobre `nombreCorto`: los campos de nombre de los bancos cortan alrededor de 30
caracteres, así que una razón social larga llega mutilada al destinatario. Esa
es la versión que se copia al formulario.

Sobre `fondo`: lo habitual es una foto oscura del rubro del cliente. Sirve
cualquier URL pública; las de Unsplash funcionan bien. La página le aplica encima
una capa negra al 80% para que el texto se lea.

También acepta un color o un degradado, y ahí no aplica esa capa: si el color se
eligió a mano, oscurecerlo sería pelear contra la decisión. Conviene cuando la
marca tiene su propia paleta y una foto la ensuciaría, y de paso la página deja de
depender de una imagen externa. Por ejemplo:

```js
fondo: 'radial-gradient(circle at 50% 0%, #3a1526 0%, #0b0b0b 62%)',
```

Sobre `acento`: tiene que contrastar contra texto oscuro, porque las letras de
los botones son casi negras. Los tonos claros funcionan; los muy saturados no.

### `CONTACTO`

Quien recibe el comprobante. Alimenta el botón flotante de WhatsApp y el botón
"Guardar Contacto en Agenda", y no cambia según el banco.

`mensajeWhatsapp` es lo que el tercero envía al cliente. Tiene que avisar que
harán la transferencia, no ordenarle al cliente que suba un voucher. El aviso
de voucher está en la página, para quien paga.

El campo `whatsapp` va con código de país y solo dígitos: `56912345678`, sin `+`
ni espacios ni guiones.

### `CUENTAS`

Una entrada por cuenta, en el orden en que aparecen las pestañas. Con una sola
cuenta la barra de pestañas no se muestra y la página queda igual que antes de
tener pestañas.

```js
const CUENTAS = [
    {
        banco: 'bci',
        tipoCuenta: 'Cuenta Corriente',
        cuenta: '97485799'
    },
    {
        banco: 'bancoDeChile',
        tipoCuenta: 'Cuenta Corriente',
        cuenta: '2602899910',
        correo: 'joserespaldo4@gmail.com'
    }
];
```

`banco` es una clave del catálogo `BANCOS`, no el nombre escrito.

`correo` es opcional: si no está, la cuenta usa el correo de `CONTACTO`. Se pone
solo cuando esa cuenta recibe los avisos en otra casilla.

También son opcionales `empresa`, `razonSocial`, `nombreCorto` y `rut`. Sirven
cuando un cliente factura con más de una sociedad: la cuenta que los declare usa
los suyos y las demás heredan los de `CLIENTE`.

## Agregar un banco al catálogo

El catálogo `BANCOS` está justo debajo de los datos del cliente y se comparte
entre todos los llaveros. Hoy tiene BCI y Banco de Chile con su logo, y Santander
y BancoEstado con la sigla, porque sus sitios no dejan bajar el SVG oficial.

Para un banco que no esté, la versión mínima es:

```js
bancoEstado: { nombre: 'BancoEstado', sigla: 'BE', color: '#e35205' },
```

Con eso la pestaña muestra un círculo con la sigla y el nombre escrito al lado,
sobre el color del banco. Se ve bien y no requiere buscar nada más.

Para agregarle el logo:

1. Buscar el SVG en el sitio del propio banco, no en páginas de logos de
   terceros, que suelen tener versiones viejas. Suele estar en el pie de página
   o en la sala de prensa. Conviene la **versión para fondo oscuro** (blanca o
   invertida), porque la tarjeta es oscura.
2. Pegarlo como un `<symbol>` dentro del `<svg class="sprite-logos">` que está al
   inicio del `<body>`, con un `id` propio y conservando su `viewBox`.
3. Agregar `logo` y `logoAlto` a la entrada del catálogo:

   ```js
   bancoEstado: {
       nombre: 'BancoEstado',
       sigla: 'BE',
       color: '#e35205',
       logo: 'logoBancoEstado',
       logoAlto: 20
   },
   ```

`logoAlto` se ajusta mirando la pestaña: cada marca tiene su proporción y el
ancho se calcula solo a partir del `viewBox`.

Los logos van incrustados en el archivo y no enlazados desde afuera. El llavero
se usa en la calle y una imagen que depende de la red deja la pestaña rota justo
cuando importa.

Si el color del banco es parecido a algún color del propio logo, conviene
oscurecerlo un poco para que el logo no se pierda. Es lo que pasó con BCI: su
azul de marca es `#2c70b8` y en la pestaña se usa `#1f5d9f`.

## Publicar

En **Settings → Pages** del repositorio, elegir la rama `main` y la carpeta
raíz. La URL queda como `https://simpleideal.github.io/nombre-del-repo/`.

Los cambios tardan un par de minutos en aparecer. Si la página se ve vieja,
suele ser caché del navegador: recargar con Ctrl+Shift+R.

## Pegar todos los datos

El botón «Copiar todos los datos» arma un texto con una etiqueta por línea.
Las apps lo parten por esa etiqueta, no por el orden de los renglones:

```
Nombre: ...
Nombre destinatario: ...
RUT: ...
Banco: ...
Tipo de cuenta: ...
Número de cuenta: ...
Correo: ...
```

`Nombre:` va escrito así, sin abreviar. Sin esa etiqueta el titular queda
vacío en varios bancos. `Número de cuenta` tampoco se abrevia: Banco
Falabella ignora `N° de cuenta` y sí reconoce la frase completa.

### Banco de Chile y el nombre

Banco de Chile exige el nombre para poder continuar. No lee la línea
`Nombre:`. La etiqueta que asocia a ese campo es `Nombre destinatario:`, con
el mismo nombre. `Nombre:` queda primero: un banco que solo busca la palabra
"nombre" no debe llevarse "destinatario" como si fuera parte del titular.

### Cuenta RUT de BancoEstado

La Cuenta RUT es una cuenta vista. El número que se pega es el RUT sin
puntos, sin guion y sin dígito verificador. El RUT completo, con el dígito,
va solo en la línea `RUT:`.

El tipo no se llama igual en todas las apps:

- En BancoEstado aparece como Cuenta RUT.
- En otros bancos aparece como Cuenta Vista, o como una sola opción
  Cuenta RUT/Vista.

Dale Coopeuch no tiene una opción llamada exactamente `Cuenta RUT`. El pegado
compara `Tipo de cuenta: Cuenta RUT` con la lista del formulario y, al no
encontrar esa frase, deja el tipo sin elegir. El resto de los campos sí
entra. Hay que elegir a mano Cuenta Vista o Cuenta RUT/Vista: es la misma
cuenta y el número no cambia.

## Antes de entregar el llavero

- Abrir la página en un teléfono, no solo en el computador.
- Copiar todos los datos y pegarlos en el formulario de un banco de verdad para
  confirmar que el pegado automático los reconoce.
- Revisar el número de cuenta y el correo de confirmación contra la cartola. Es
  el único error de esta página que cuesta plata.
- Probar el botón de WhatsApp y el de guardar contacto.

## Cómo está hecho por dentro

`index.html` tiene tres partes: los estilos, la tarjeta en HTML y el script.

La tarjeta en HTML viene con marcadores `—` en lugar de datos. Los valores los
escribe el script al cargar, leyendo el bloque del cliente. Está hecho así a
propósito: si alguien copia la plantilla y se olvida de cambiar algo, la página
muestra un guion y no los datos bancarios del cliente anterior.

El script no usa librerías. La copia al portapapeles intenta primero la
Clipboard API y cae a un método antiguo cuando el navegador la bloquea, que pasa
seguido dentro de los navegadores de WhatsApp e Instagram. Si las dos fallan,
abre un cuadro con el texto seleccionado para copiar a mano.
