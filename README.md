# Blog-Rob-tica-Servicios

Recomendación antes de leer: algunas imagenes no tienen fondo, recomendado usar el modo claro para poder visualizarlas.

## Práctica 1 - Aspiradora localizada

La primera práctica se basa en crear un algoritmo BSA de cobertura para una aspiradora. El objetivo es limpiar la mayor superficies posible de una casa, para ello nos proporcionan una imagen en blanco y negro del mapa de la casa, un mundo en Gazebo y las coordenadas del robot (tanto posición como ángulo de giro) en dicho mundo. 

Para poder abarcar la práctica poco a poco la podemos dividir en 4 fases principales.

### Registro del mapa

El primer paso para poder afrontar la práctica es entender los sistemas de coordenadas de cada mapa que tenemos. Por un lado, gazebotiene una representación en 3D, y la imagen en 2D, que si lo miramos como si fuese una matriz, las X son las columnas y la Y las filas.

(añadir imagen apuntes)

Una vez tenemos esto en cuenta, podemos empezar a hacer el registro del mapa. Para ello, tenemos que ir moviendo el robot a diferentes puntos y apuntar tanto la posición en el simulador como en la imagen. Algo a tener en cuenta es que la imagen es de 1012x1012 pixel. Lo más óptimo sería hacerlo con 12 puntos, pero en este caso lo haré con 4 puntos que serán las 4 esquinas de la casa. En el video se puede ver la toma de coordenadas de un punto:

( añadir video)

(añadir tabla)

!!!!!!Comprobar signos y valores¡¡¡¡¡¡

Congosto puntos ya calculados, podemos empezar con las matrices de transformación. Necesitamos saber la traslación,  rotación y escala de un mapa al otro. Empezando por lo fácil, la rotación de un mapa a otro es de 0°, lo que hará la matriz más sencilla. Para poder calcular la traslación nos vamos a basar en el primer punto calculado: en el punto (0,0) de la imagen, la posición del robot es (-5.67 , 0 , 4.17). Haciendo la relación entre sistemas de coordenadas entre simulador e imagen podemos sacar que la translación en X es de 5.67 y de Y -4.17. Sólo nos quedaría calcular el escalar, para ello, he seguido la siguiente fórmula

|           | Imagen         | Gazebo        |
| :---      |     :---:      |          ---: |
| Punto A   |   (0,0)        | (5.67, -4.17) |
| Punto B   | (1012,1012)    | (-4.27,5.95)  |

<img width="192" height="69" alt="CodeCogsEqn" src="https://github.com/user-attachments/assets/443c91b6-c03b-4b68-b2ad-3deae170cac3" />

<img width="176" height="69" alt="CodeCogsEqn (1)" src="https://github.com/user-attachments/assets/cade5aaa-7689-42b3-bb04-1e8725c5cfa5" />


Teniendo en cuenta que U y J corresponden a las coordenadas X e Y de la imagen respectivamente, calculando entre varios puntos y haciendo una media, nos sale que el escalar es de aproximadamente 101.81, siendo negativo en X y positivo en Y.

Teniendo ya todos los datos, podemos sustituir en la fórmula:

<img width="772" height="219" alt="image" src="https://github.com/user-attachments/assets/c5c9d42d-04f1-425b-a9b9-ac2083722a4a" />

Lo último a tener en cuenta es que podemos olvidarnos de la coordenada Z, y necesitaríamos la Tranformada de dicha matriz, cambiando el signo de la translación. La matriz sería:

``` python
T_inv = [[1, 0, -5.6], [0, 1, 4.17], [0, 0, 1]]
scale_x = -101.81
scale_y = 101.81
```

### Creación de la rejilla de navegación

El propio enenciado de la práctica nos dice que la aspiradora tiene un tamaño de 35x35 pixel, además de saber que el mapa mide 1012x1012

2.- creación de la rejilla de navegación

3.- planificación de ruta siguiendo algoritmo de cobertura BSA

4.- pilotaje reactivo para ejecutar la ruta planificada.
