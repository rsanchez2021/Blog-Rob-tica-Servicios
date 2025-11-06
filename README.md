# Blog-Rob-tica-Servicios

**Recomendación antes de leer**: algunas imágenes no tienen fondo, por lo que se recomienda usar el modo claro para poder visualizarlas correctamente.

## Práctica 1 - Aspiradora localizada

La primera práctica se basa en crear un algoritmo BSA de cobertura para una aspiradora. El objetivo es limpiar la mayor superficie posible de una casa, para ello nos proporcionan una imagen en blanco y negro del mapa de la casa, un mundo simulado en Gazebo y las coordenadas del robot (posición y orientación) en dicho mundo. 

Para poder abordar la práctica poco a poco, podemos dividirla en 4 fases principales.

### Registro del mapa

El primer paso para poder afrontar la práctica es entender los sistemas de coordenadas de cada mapa que tenemos. Por un lado, Gazebo tiene una representación en 3D, y la imagen en 2D. Si la observamos como si fuese una matriz, el eje X representa las columnas y la Y las filas.

<img width="729" height="310" alt="image" src="https://github.com/user-attachments/assets/7d5fd6be-0e74-4250-a45e-52fb8851f01c" />


Una vez tenemos esto en cuenta, podemos empezar a hacer el registro del mapa. Para ello, tenemos que ir moviendo el robot a diferentes puntos y apuntar tanto la posición en el simulador como en la imagen. Algo a tener en cuenta es que la imagen tiene una resolución de 1012x1012 píxeles. Lo más óptimo sería hacerlo con 12 puntos, pero en este caso lo haré con 4 puntos que serán las 4 esquinas de la casa.

| IMAGEN | GAZEBO |
| ------------- | ------------- |
| (938,310)  | (-4.21 , -1.40)  |
| (850, 995)  | (-3.017 , 6.08)  |
| (49,995)  | (5.14 , 5.55)  |
| (49,71)  | (5.67 , -4.17)  |


Con estos puntos ya calculados, podemos empezar con las matrices de transformación. Necesitamos saber la traslación,  rotación y escala de un mapa al otro. Empezando por lo fácil, la rotación de un mapa a otro es de 0°, lo que hará la matriz más sencilla. Para poder calcular la traslación nos vamos a basar en el primer punto calculado: en el punto (0,0) de la imagen, la posición del robot es (5.67 , 0 , -4.17). Haciendo la relación entre sistemas de coordenadas entre simulador e imagen podemos sacar que la traslación en X es de 5.67 y en Y -4.17. Sólo nos quedaría calcular la escala, para ello, he seguido la siguiente fórmula

|           | Imagen         | Gazebo        |
| :---      |     :---:      |          ---: |
| Punto A   |   (0,0)        | (5.67, -4.17) |
| Punto B   | (1012,1012)    | (-4.27,5.95)  |

<img width="192" height="69" alt="CodeCogsEqn" src="https://github.com/user-attachments/assets/443c91b6-c03b-4b68-b2ad-3deae170cac3" />

<img width="176" height="69" alt="CodeCogsEqn (1)" src="https://github.com/user-attachments/assets/cade5aaa-7689-42b3-bb04-1e8725c5cfa5" />


Teniendo en cuenta que U y J corresponden a las coordenadas X e Y de la imagen respectivamente, calculando entre varios puntos y haciendo una media, nos sale que el escalar es de aproximadamente 101.81, siendo negativo en X y positivo en Y.

Teniendo ya todos los datos, podemos sustituir en la fórmula:

<img width="772" height="219" alt="image" src="https://github.com/user-attachments/assets/c5c9d42d-04f1-425b-a9b9-ac2083722a4a" />

Por último, podemos ignorar la coordenada Z. Necesitamos la transformada de dicha matriz, cambiando el signo de la traslación. La matriz sería:

``` python
T_inv = [[1, 0, -5.6], [0, 1, 4.17], [0, 0, 1]]
scale_x = -101.81
scale_y = 101.81
```

### Creación de la rejilla de navegación

El propio enunciado de la práctica nos dice que la aspiradora tiene un tamaño de 35x35 píxeles, además de saber que el mapa mide 1012x1012, por ello, haciendo varias pruebas, el tamaño que he usado para las celdas es de 44 píxeles, de esta forma, no quedan píxeles sueltos, y obtenemos una matriz de 23x23 que corresponde a cada celdilla, donde se representan los tipos de celdas como:

- Obstáculo --> 0
- Sucio --> 1
- Limpia --> 2
- Punto crítico --> 3
- Punto de retorno --> 4

De esta forma, es más sencillo a la hora de planificar.
Además, para poder saber cuándo una celda debe ser obstáculo o no, en función de la proporción de píxeles blancos respecto a los negros, determinamos so la celda es libre o no.

Mapa solo con las rejillas:

<img width="553" height="311" alt="Screenshot from 2025-09-28 12-25-33" src="https://github.com/user-attachments/assets/911bfd6e-0195-48ed-a79f-1bf71b982bb7" />

Mapa con celdillas ya clasificadas:

<img width="553" height="311" alt="Screenshot from 2025-09-28 12-38-11" src="https://github.com/user-attachments/assets/0563e582-5f26-4d72-90e1-7b30714f523f" />

<img width="1155" height="750" alt="Screenshot from 2025-09-28 19-05-34" src="https://github.com/user-attachments/assets/ac22564f-9954-4384-9045-29f70f531f68" />

### Planificación de ruta siguiendo algoritmo de cobertura BSA

El algoritmo de cobertura BSA se basa en seguir siempre una dirección, hasta encontrar un obstáculo y seguir en otra dirección, así sucesivamente. Mientras avanza va marcando los puntos ya visitados (en este caso limpios) mientras que marca también los vecinos libres (que serán nuestros puntos de retorno). Cuando ya no puede moverse a ninguna dirección sin pasar por obstáculo o por un sitio ya visitado, entonces se crea un punto crítico. Para poder salir, busca el punto de retorno más cercano y vuelve a repetir el algoritmo. De esta forma, genera espirales hasta completar la cobertura total del mapa. 

Para poder pasar de teoría a código voy a ir poco a poco.

1. **Direcciones:**
En mi caso, el orden de prioridad de las direcciones es: ESTE , NORTE , OESTE , SUR . Todo tomándolo como sistema de referencia la imagen, no la del propio robot, es decir, puede no girar hacia el ESTE (derecha) de la aspiradora.

2. **Obstáculos y puntos visitados:**
Ambos casos se consideran obstáculos, excepto a la llegada de un punto crítico que se toman los puntos visitados como los únicos posibles para poder moverse. De esta forma, optimizamos el algoritmo y nos evitamos choques con mobiliario y/o paredes.

3. **Vecinos libres:**
Para establecer los vecinos libres como posibles puntos de retorno, mientras voy generando la ruta, se comprueban los vecinos de cada celdilla y nos quedamos solo con los sucios no visitados.

4. **Punto crítico:**
Se considera punto crítico cuando ya no puedo moverme a ninguna dirección sin pasar por obstáculo o celda libre. Para buscar el siguiente punto de retorno, calculamos la distancia Manhattan de cada posible punto de retorno con el punto crítico. Nos quedamos con la distancia más corta y generamos una nueva ruta de celdillas (sólo limpias) hasta el punto de retorno. De esta forma, podemos simplificar el movimiento entre celdillas más alejadas y evitar una vez más los posibles choques.

Vídeo ejemplo de la planificación:


https://github.com/user-attachments/assets/a25e48d2-1eaa-4c94-89ba-b0afbbafb267

Más ejemplos de planes desde diferentes posiciones iniciales:


https://github.com/user-attachments/assets/4237c3df-ad6b-4fdc-bedd-1b8e6fa3223b



### Pilotaje reactivo para ejecutar la ruta planificada.

Teniendo la ruta entera que debe seguir la aspiradora, el pilotaje es más sencillo: lo único que debemos hacer es comprobar la posición de la aspiradora respecto a las celdillas (usando lo visto en el punto 1) e ir moviéndonos de celdilla a celdilla sin chocarnos. Para calcular la posición del robot respecto a la matriz de celdas usamos:

``` python
pos_rum_gaz = [HAL.getPose3d().x , HAL.getPose3d().y , 1]
pos_rum_t = np.dot(T_inv , pos_rum_gaz)
pos_rum_img = [pos_rum_t[0] * scale_x , pos_rum_t[1] * scale_y]

# Calcular casilla
pos_rum_mc = [int(pos_rum_img[1] / 44), int((pos_rum_img[0]) / 44)]
```

### Dificultades de la práctica

La mayor dificultad que he tenido durante la práctica es la implementación de los puntos de retorno una vez llegaba a un punto crítico, ya que solo buscaba el primer punto, sin tener en cuenta la distancia a los demás o el hecho de tener que atravesar algún obstáculo. Pero una vez implementado la creación de una ruta por celdas usando el algoritmo de los vecinos cercanos, no tuve más complicaciones.

La segunda gran dificultad fue la falta de precisión en la toma de los puntos. En un principio, saqué solo dos puntos, y había momentos en los que se chocaba con las paredes o con el mobiliario, pero al tomar más puntos, los valores se hacen más certeros y me evité los choques.

Por último, no fue una dificultad pero si un inconveniente, era el tiempo de ejecución que tardaba la aspiradora en recorrer toda la casa. Hubo muchas pruebas en las que la aspiradora se chocaba a último momento tras más de 20 minutos de ejecución.


### Vídeo final

Enlace al vídeo [aquí](https://youtu.be/rjfFn_xc5UA)

Si no puede visualizarlo copie y pegue el enlace: https://youtu.be/rjfFn_xc5UA

## Práctica 2 - Rescatar personas

La segunda práctica consiste en realizar un barrido sobre una superficie del mar y detectar a todos las personas flotando. Para ello, nos proporcionan la imagen frontal y ventral de un dron, las posición del barco del que despega el dron, una posición aproximada de la zona a recorrer y tres posibilidades del control del dron (posición, velocidad o mixto). 

A la hora de realizar la práctica se divide en tres apartados


### Coordenadas

El enunciado de la práctica nos dice que el barco se enccuentra en la posición **40º16’48.2” N, 3º49’03.5” W** y la zona de rescate cerca de **40º16’47.23” N, 3º49’01.78” W**, en coordenadas GPS.

EL primer paso es convertir las coordenadas a UTM, para ello usamos la página web proporcionada por el enunciado y obtenemos [[1]](http://rcn.montana.edu/Resources/Converter.aspx):

| GPS | UTM |
| ------------- | ------------- |
| 40º16’48.2” N, 3º49’03.5” W  | Zona 30 430492E  4459162N |
| 40º16’47.23” N, 3º49’01.78” W  | Zona 30  430532E 4459132N |

Una vez tenemos las coordenadas UTM podemos calcular la posición en metros respecto a las coordenadas globales. Simplemente nos tenemos que quedar con los tres últimos números de Easting y Northing:
+ Bote: (492,161)
+ Zona: (532,132)

El último paso es realizar la translación correspondiente a la posición local del dron, es decir, el dron comienza en la posición (0,0) que corresponde con la posición de bote.

<img width="1600" height="708" alt="image" src="https://github.com/user-attachments/assets/9b0af804-a3f6-46e9-9e53-1e4780c6153c" />



Hay que tener en cuenta también que el movimiento del dron es: Forward (+x), Left (+y), Up (+z). Esto se refleja en que las posiciones en el eje Y deben ser negativas y que los ejes X e Y están invertidos.

<img width="338" height="290" alt="image" src="https://github.com/user-attachments/assets/2e39ea7d-b149-4427-b24b-95e68ffa9d43" />

Por tanto, la tabla de conversión de coordenadas quedaría:

| GPS | UTM | Global | Local |
| ------------- | ------------- | ------------- | ------------- |
| 40º16’48.2” N, 3º49’03.5” W  | Zona 30 430492E  4459162N | (492,161) | (0,0) |
| 40º16’47.23” N, 3º49’01.78” W  | Zona 30  430532E 4459132N | (532,132) | (29, -40) |

Comentar también que en mi caso, el dron no empieza en (29,-40) exactamente ya que el primer movimiento que realiza es hacia *Forward*, ppor lo que comienza en (22, -40) 

### Barrido del área

Teniendo las primeras coordenadas calculadas, el primer comando para el dron se hace por posición, manteniendo durante todo el barrido de la zona una altura constante y un yaw (ángulo de rotación del dron) sin modificar. Esta será útil para el siguiente paso de detección de caras.

Una vez llega a la zona, el barrido lo realizo con control por velocidad, haciendo un recorrido constante hasta que se cubre un área determinada.

<img width="1199" height="1164" alt="image" src="https://github.com/user-attachments/assets/5aef516b-0d43-4b5d-8080-b879ee2c9d01" />


### Detección de caras

El último paso es la detección de caras. Para ello he usado Haar Cascade que nos proporciona el propio enunciado. La única implementación extra que he tenido que usar es el giro de la imagen.

Tenemos la posición de las caras en (x,y) en píxeles respecto a la imagen ventral, no en las coordenadas locales del dron. Para poder calcularlas necesitamos saber:
+ FOV del dron: 60º horizontal y 45º vertical
+ Tamaño de la imagen en pixeles: 320px x 240px
+ Altura constante del dron: 2 metros

Con estos datos podemos calcular la correspondencia en metros de la siguiente manera:



<img width="346" height="37" alt="CodeCogsEqn" src="https://github.com/user-attachments/assets/7fa464d7-c95e-45b5-a879-29b57901b3be" />

<img width="360" height="37" alt="CodeCogsEqn(1)" src="https://github.com/user-attachments/assets/a8aecf00-6c65-4ede-824d-778815b9d5df" />


<img width="1567" height="1282" alt="image" src="https://github.com/user-attachments/assets/c75335bf-a038-433a-a0f5-d4d851bb1c51" />


### Vídeo final

Enlace al [vídeo](https://youtu.be/WOwCfB-IScM)


## Práctica 3

https://youtu.be/CzMuiJkklKE
