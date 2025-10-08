# Blog-Rob-tica-Servicios

## Práctica 1 - Aspiradora localizada

La primera práctica se basa en crear un algoritmo BSA de cobertura para una aspiradora. El objetivo es limpiar la mayor superficies posible de una casa, para ello nos proporcionan una imagen en blanco y negro del mapa de la casa, un mundo en Gazebo y las coordenadas del robot (tanto posición como ángulo de giro) en dicho mundo. 

Para poder abarcar la práctica poco a poco la podemos dividir en 4 fases principales.

### Registro del mapa

El primer paso para poder afrontar la práctica es entender los sistemas de coordenadas de cada mapa que tenemos. Por un lado, gazebotiene una representación en 3D, y la imagen en 2D, que si lo miramos como si fuese una matriz, las X son las columnas y la Y las filas.

(añadir imagen apuntes)

Una vez tenemos esto en cuenta, podemos empezar a hacer el registro del mapa. Para ello, tenemos que ir moviendo el robot a diferentes puntos y apuntar tanto la posición en el simulador como en la imagen. Algo a tener en cuenta es que la imagen es de 1024x1024 pixel. Lo más óptimo sería hacerlo con 12 puntos, pero en este caso lo haré con 4 puntos que serán las 4 esquinas de la casa. En el video se puede ver la toma de coordenadas de un punto:

( añadir video)

(añadir tabla)

Congosto puntos ya calculados, podemos empezar con las matrices de transformación. Necesitamos saber la traslación,  rotación y escala de un mapa al otro. Empezando por lo fácil, la rotación de un mapa a otro es de 0°, lo que hará la matriz más sencilla. Para poder calcular la traslación nos vamos a basar en el primer punto calculado: en el punto (0,0) de la imagen, la posición del robot es (-5.67 , 



registro del mapa

2.- creación de la rejilla de navegación

3.- planificación de ruta siguiendo algoritmo de cobertura BSA

4.- pilotaje reactivo para ejecutar la ruta planificada.
