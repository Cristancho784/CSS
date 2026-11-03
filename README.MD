# Catálogo semántico y especificidad CSS

## Herencia

El selector `body` declara un color de texto base. Los encabezados, párrafos y
listas heredan ese color, salvo cuando otra regla aplicable establece un color
propio. Se omitieron los cambios de tipografía.

## Enlaces por zona

Los selectores combinados `header nav ul li a`, `aside nav ul li a` y
`footer ul li a` asignan colores distintos a los enlaces de navegación
principal, barra lateral y pie de página, respectivamente.

## Cálculo de especificidad

| Tipo de selector | Puntos |
| --- | ---: |
| Etiqueta | 1 |
| Clase | 10 |
| ID | 100 |

Para calcular la especificidad se suman los puntos de cada componente del
selector. Por ejemplo, `a` suma 1 punto, `.comprar` suma 10 puntos y
`#btn-destacado` suma 100 puntos. El selector combinado `header nav ul li a`
suma 5 puntos porque contiene cinco selectores de etiqueta.

Cuando varias reglas aplican al mismo elemento, gana la de mayor
especificidad. Si empatan, prevalece la última regla declarada. El enlace
destacado conserva sus clases e ID en el HTML, pero la hoja de estilos final
usa únicamente etiquetas y clases: no contiene selectores por ID ni usa
`!important`.

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

Las tarjetas `<article>` tienen `padding: 20px` y `margin: 15px`. En DevTools,
al seleccionar una tarjeta, el diagrama del modelo de caja debe mostrar esos
valores como relleno interno y margen externo, respectivamente.
