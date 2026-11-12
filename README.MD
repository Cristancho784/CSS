# Catálogo semántico y especificidad CSS

## Herencia

El selector `body` declara un color de texto base. Los encabezados, párrafos y
listas heredan ese color, salvo cuando otra regla aplicable establece un color
propio. Se omitieron los cambios de tipografía.

## Enlaces por zona

Los selectores `.site-header__link`, `aside nav ul li a` y
`.site-footer__link` asignan colores distintos a los enlaces de navegación
principal, barra lateral y pie de página, respectivamente.

## Cálculo de especificidad

| Tipo de selector | Puntos |
| --- | ---: |
| Etiqueta | 1 |
| Clase | 10 |
| ID | 100 |

Para calcular la especificidad se suman los puntos de cada componente del
selector. Por ejemplo, `a` suma 1 punto, `.btn` y `.card` suman 10 puntos cada
uno, y un ID suma 100 puntos. El selector combinado `header nav ul li a` suma 5
puntos porque contiene cinco selectores de etiqueta.

Cuando varias reglas aplican al mismo elemento, gana la de mayor
especificidad. Si empatan, prevalece la última regla declarada. Los modificadores
del HTML, como `.card--destacada` y `.btn--secundario`, también son clases y no
añaden especificidad por sí mismos cuando se usan como un selector individual.
La hoja de estilos no contiene selectores por ID ni usa `!important`.

## Hallazgos: El Modelo de Caja

- **Margin y Padding:** `padding` es el espacio interno entre el contenido y el
  borde del elemento; `margin` es el espacio externo que lo separa de otros
  elementos.
- **`box-sizing: border-box`:** hace que el padding y el borde se incluyan en el
  tamaño declarado del elemento. Esto permite añadir relleno sin que el ancho o
  alto total exceda las dimensiones previstas.
- **Márgenes grandes en elementos hermanos:** aumentan la separación exterior
  entre las tarjetas y pueden empujarlas más lejos unas de otras, haciendo que
  ocupen más espacio y que la distribución cambie.

Las tarjetas `.card` usan `padding: var(--espaciado-tarjeta)` y no agregan
márgenes propios; la separación entre ellas se controla en la cuadrícula con
`column-gap` y `row-gap`. Así, los espacios no se duplican entre los márgenes
de las tarjetas y los espacios de la cuadrícula.

## Hallazgos: Centralización de Identidad Visual y Espaciado

La pseudoclase `:root` de `style.css` centraliza colores, tipografía y medidas
de componentes. Los tokens de color y espaciado se nombran por zona o función,
por ejemplo `--color-hover-boton`, `--color-focus-lateral`,
`--espaciado-header-nav` y `--espaciado-filtro-gap`. Así se pueden ajustar de
forma independiente sin modificar estilos no relacionados. También se
mantienen tokens de tipografía como `--fuente-principal` y de detalles como
`--radio-boton`, `--radio-insignia` y `--tamano-insignia`.

Reutilizar tokens de espaciado hace que las tarjetas mantengan medidas
coherentes y permite ajustar el aire de varias reglas desde un solo lugar, en
vez de acumular valores independientes difíciles de armonizar. Los tokens
específicos de tarjeta conservan las dimensiones que ya tenía el catálogo.

Al centralizar colores, tipografía y medidas en `:root`, cada componente toma
sus valores de una fuente común. Así, los cambios de identidad visual se
aplican de forma consistente y se reduce el riesgo de que reglas nuevas
introduzcan colores o separaciones incompatibles a medida que crece la hoja de
estilos.

## Hallazgos: Pseudoclases y Pseudoelementos

Las pseudoclases describen estados o relaciones dinámicas de los elementos. Por
ejemplo, `:hover` responde a que el cursor pase sobre un enlace o botón,
`:active` identifica el momento de presión y `:focus` señala que un elemento
recibió el foco del teclado. En cambio, los pseudoelementos crean una pieza
virtual vinculada al elemento seleccionado: `::before` se inserta antes de su
contenido y `::after` después, sin modificar el HTML real.

La pseudoclase `:focus` es esencial para la accesibilidad. En los enlaces del
menú lateral y en los botones se muestra un contorno visible construido con sus
tokens independientes `--color-focus-lateral` y `--color-focus-boton`. De esta
manera, una persona que navega con Tab
puede reconocer exactamente qué control está activo, incluso sin usar el
cursor. El estilo incluye `outline-offset` para separar el contorno del enlace
sin depender del contorno predeterminado del navegador.

Los elementos decorativos deben inyectarse desde CSS cuando forman parte de la
presentación y no aportan contenido independiente ni acciones. Colocar `•` con
`::before` y la insignia `¡Oferta!` con `::after` mantiene el HTML limpio, evita
etiquetas vacías o repetidas y permite cambiar todos los detalles visuales desde
la hoja de estilos. Los colores y medidas de estas insignias provienen de las
variables de `:root`, por lo que conservan la identidad visual del catálogo.

Para auditar los estados se pueden seleccionar las tarjetas o los enlaces del
aside en DevTools, activar `:hov` para simular `:hover`, `:active` o `:focus`, y
revisar el panel de Estilos. Allí se puede comprobar que `::before` aporta el
indicador de categoría y que `::after` genera la etiqueta `¡Oferta!` dentro del
árbol virtual sin existir como elemento HTML adicional.

## Hallazgos: Arquitectura BEM y Modularidad

BEM significa **Block, Element, Modifier** (bloque, elemento, modificador). Su
forma visual separa el bloque del elemento con dos guiones bajos y agrega una
variación con dos guiones medios: `.card`, `.card__title` y
`.card--destacada`. Los modificadores pueden acompañar al bloque base en HTML,
por ejemplo `class="btn btn--secundario"`, para identificar una variante sin
introducir nombres genéricos.

La hoja maestra importa módulos por componente como `css/tarjeta.css`,
`css/botones.css`, `css/menu-lateral.css`, `css/header.css`, `css/aside.css` y
`css/footer.css`. Así, quienes trabajan en cada zona pueden localizar y
modificar sus reglas con menos interferencia entre sí. `style.css` conserva los
tokens de `:root` y el contexto global, mientras los módulos reutilizan las
variables específicas de cada componente.

Los selectores BEM planos como `.card` y `.btn` tienen una especificidad de una
clase (10 puntos en la escala simplificada de esta guía). Evitar cadenas como
`.catalog section article h2` reduce la acumulación de peso y la necesidad de
sobrescribir reglas, lo que ayuda a prevenir conflictos de cascada. Los estados
como `:hover` y los pseudoelementos como `::after` añaden su propia
especificidad; por eso conviene mantenerlos asociados a una clase BEM concreta
y evitar anidamientos innecesarios.

## Hallazgos: Arquitectura Total con Flexbox

1. **El pie de página y la altura de la ventana:** `.app-layout` usa
   `display: flex`, `flex-direction: column` y `min-height: 100vh`. Esto crea
   una columna que como mínimo ocupa toda la altura visible. Como `main` usa
   `flex: 1`, toma el espacio vertical libre y mantiene el pie al fondo cuando
   hay poco contenido; si el contenido crece, la página puede crecer con él.

2. **Separación consistente en la navegación:** las listas de navegación del
   encabezado y del pie usan Flexbox con `gap` y reinician sus márgenes. En vez
   de añadir márgenes a cada enlace y acumularlos en los extremos, `gap`
   controla directamente la distancia entre elementos. Los tokens
   `--espaciado-header-nav` y `--espaciado-footer-nav` permiten ajustar cada
   navegación por separado, y `flex-wrap` deja que los enlaces pasen a otra
   línea si falta espacio.

3. **Tarjetas resistentes a textos de distinta longitud:** BEM distingue el
   bloque `.card` de elementos como `.card__description` y
   `.card__price-wrapper`. La tarjeta es una columna flex; la descripción usa
   `flex-grow: 1` para absorber el espacio libre, mientras el botón queda al
   fondo. La cuadrícula usa filas de igual fracción y la tarjeta ocupa la
   altura de su celda, de modo que las tarjetas mantienen dimensiones
   uniformes. El área de precios alinea la oferta y el precio original con
   Flexbox, y las Custom Properties permiten ajustar rellenos y separaciones
   por componente sin propagar cambios a otras zonas.
