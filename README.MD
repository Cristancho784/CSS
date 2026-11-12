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

Las tarjetas `.card` tienen `padding: 20px` y `margin: 15px`. En DevTools, al
seleccionar una tarjeta, el diagrama del modelo de caja debe mostrar esos
valores como relleno interno y margen externo, respectivamente.

## Hallazgos: Centralización de Identidad Visual y Espaciado

La pseudoclase `:root` de `style.css` concentra estas variables globales:

- **Colores:** `--color-primario` (`#166534`), `--color-secundario` (`#1a56db`),
  `--color-fondo` (`#f3f4f6`), `--color-texto` (`#000000`),
  `--color-fondo-encabezado` (`#dbeafe`), `--color-fondo-lateral` (`#dcfce7`),
  `--color-fondo-pie` (`#e5e7eb`), `--color-enlace-lateral` (`#047857`),
  `--color-enlace-pie` (`#b45309`) y `--color-texto-inverso` (`#ffffff`).
- **Tipografía:** `--fuente-principal` usa Nunito Sans, cargada desde Google Fonts.
- **Espaciado:** `--espaciado-pequeno` (8 px), `--espaciado-medio` (16 px),
  `--espaciado-grande` (24 px), `--espaciado-tarjeta` (20 px) y
  `--espaciado-entre-tarjetas` (15 px).
- **Detalles de componentes:** `--radio-boton` (4 px), `--radio-insignia`
  (999 px), `--tamano-insignia` (0.75 rem) y
  `--relleno-insignia-vertical` (2 px).

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
menú lateral y en los botones se muestra un contorno visible construido con la
variable `--color-secundario`. De esta manera, una persona que navega con Tab
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

La hoja maestra importa módulos por componente: `css/tarjeta.css`,
`css/botones.css` y `css/menu-lateral.css`. Así, quienes trabajan en tarjetas,
botones o navegación lateral pueden localizar y modificar sus reglas con menos
interferencia entre sí. `style.css` conserva los tokens compartidos de `:root`
y el contexto global, mientras los módulos reutilizan esas variables para
colores y espaciados coherentes.

Los selectores BEM planos como `.card` y `.btn` tienen una especificidad de una
clase (10 puntos en la escala simplificada de esta guía). Evitar cadenas como
`.catalog section article h2` reduce la acumulación de peso y la necesidad de
sobrescribir reglas, lo que ayuda a prevenir conflictos de cascada. Los estados
como `:hover` y los pseudoelementos como `::after` añaden su propia
especificidad; por eso conviene mantenerlos asociados a una clase BEM concreta
y evitar anidamientos innecesarios.
