### Tutorial 6
## Transformaciones de Posición

#### English: https://ogldev.org/www/tutorial06/tutorial06.html


## Contexto
En este tutorial comenzaremos a ver los diferentes tipos de transformaciones que un objeto en el entorno 3D y habilitaremos que nuestro programa pueda mostrarlo en la pantalla mientras mantenemos la ilusión de profundidad en la escena. La forma más común de hacer esto es representando cada transformación usnado una matriz, multiplicando cada una entre ellas y posteriormente multiplicando la posición del vértice po el producto final. Cada tutorial será dedicado a analizar cada transformación. 

Aquí echaremos un vistazo a la transformación de traslación, la cual es responsable te mover un objeto a través de un vector de longitud y dirección definida. Digamos que quieres mover el triángulo en la imagen de la izquierda a la posición de la derecha: 

Una forma de hacer esto es dando un vector de desfase (en este caso - 1,1) como una variable uniforma al shader y simplemente añadiendo la posición de cada uno de los vértices que vamos a procesar. Sin embargo, esto rompe el método de multiplicar un grupo de matrices en uno para obtener una única transformación. Adicionalmente, veremos después que la traslación no es usualmente la primera por lo que tendremos que multiplicar la posición por la matriz que representa las transformaciones previas a la traslación, después agregar la posciión y finalmente multiplicar por la matriz que representa la transforamción que sigue la traslación. Esto puede escucharse un tanto complicado. Una mejor forma de hacerlo sería encontrando la matriz que represente la traslación y que tome aprte en la multiplicación de todas las matrices. Pero, ¿podremos encontrar una matriz que multiplicada por el punto (0,0), el vértice en la parte inferior izquierda del triángulo, nos de el resultado (1,1)? La verdad es que no podemos hacer esto usando matrices 2D (y no puedes hacerlo con una matriz 3D para (0,0,0)). En general podemos decir que lo que necesitamos es una matriz M que dado el punto P(x,y,z) y el vector V(v1,v2,v3) nos de M * P = P1(x + v1, y + v2, z + v3). En palabras más simples esto significa que la matriz M traslada a P a la posición P+V. En P1 podemos ver que cada componente es la suma de un componente de P con su componente correspondiente en V. El lado izquierdo de cada ecuasión podemos ver la matriz de identidad: 
I * P = P(x,y,z). Parece entonces que debemos empezar con la matriz de identidad y encontrar los cambios que van a completar el lado derecho de la suma de la ecuación en cada componente (...+V1, ...+V2, ...+V3). Veamos como se ve una matriz de identidad: 

Queremos modificar la identidad de la matriz para que el resultado sea: 

No hay una forma sencilla de hacer esto si metemos una matriz de 3x3, pero si cambiamos a un matriz de 4x4 podemos hacer lo siguiente:

Representando a un vector3 usando a un vector4 de esta forma se le llaman coordenadas homogeneas y es algo muy popular y útil a la hora de trabajar con gráficos en 3D. El cuarto componente del vector se llama 'w'. De hecho, el símbolo interno del shader gl_Position que hemos visto en tutoriales pasados es un vector4, y el componente w tiene un rol muy importante para hacer la proyección de 3D a 2D. La notación común es usar w=1 para puntos y w=0 para vectores. La razón detrás de esto es que los puntos pueden trasladarse, pero los vectores no. Puedes cambiar la longitud deun vector o su dirección, pero todos los vectores que tienen la misma longitud/dirección son considerados iguales, independientemente de su "posición inicial". Así que podemos usar simplemente el origen de todos los vectores. Si asignamos w=0 y multiplicamos la matriz de transformación por el vector nos dará como resultado el mismo vector. 

## El Código Paso a Paso

```
struct Matrix4f {
    float m[4][4];
};
```
Agregamos la definición de una matriz de 4x4 a math_3d.h. Usaremos esto para la mayoría de nuestras transformaciones de matrices de ahora en adelante. 

`GLuint gWorldLocation;`

Usamos este manejador para acceder a la variable global de tipo matriz uniforme en nuestro shader. La llamaremos 'world' porque moveremos (trasladaremos) nuestro objeto a la posición que queremos en el sistema de coordenadas de nuestro 'mundo' virtual.

```
Matrix4f World;
World.m[0][0] = 1.0f; World.m[0][1] = 0.0f; World.m[0][2] = 0.0f; World.m[0][3] = sinf(Scale);
World.m[1][0] = 0.0f; World.m[1][1] = 1.0f; World.m[1][2] = 0.0f; World.m[1][3] = 0.0f;
World.m[2][0] = 0.0f; World.m[2][1] = 0.0f; World.m[2][2] = 1.0f; World.m[2][3] = 0.0f;
World.m[3][0] = 0.0f; World.m[3][1] = 0.0f; World.m[3][2] = 0.0f; World.m[3][3] = 1.0f;
```

En la función de renderizado prepararemos una matriz de 4x4 y la llenaremos de acuerdo a lo que explicamos arriba. Le asignamos a v2 y v3 el valor de cero por lo que esperamos que no haya cambios en las coordenadas Y y Z del objeto, y le asignamos a v1 el resultado de la función seno. Esto moverá (trasladará) la coordenada X por un valor que ronda a través de -1 y 1. Ahora lo que tenemos que hacer es cargarle la matriz al shader. 

`glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, &World.m[0][0]);`

Este es otro ejemplo de una función glUniform* que carga la información en una variable uniforme de shader. Esta función en particular carga matrices 4x4, pero también hay versiones para 2x2, 3x3, 3x2, 2x4, 4x2, 3x4 y 4x3. El primer parámetro es la ubicación de la variable uniforme (que llega después de la compilación del shader usando glGetUniformLocation()). El segundo parámetro indica el número de matrices que serán actualizadas. Usamos 1 para una matriz, pero también podemos usar esta función para actualizar y multiplicar las matrices en una sola llamada. El tercer parámetro suele confundir a personas que a penas comienzan a aprender OpenGL. Indica si el orden de la matriz es de tipo ascendente en filas o ascendente en columnas. Ascendente en filas significa que la matriz se ingresa fila por fila, empezando por la parte superior de la matriz. Ascendente en columnas es lo mismo que lo anterior pero en columnas. El punto de esto es que C/C++ son lenguajes que usan el orden ascendente en filas por defecto. Esto significa que cuando llenamos un arreglo de dos dimensiones con valores, estos se almacenarán en memoria fila por fila teniendo la primer fila en la dirección de memoria más baja. Por ejemplo, veamos el siguiente arreglo: 

```
int a[2][3];
a[0][0] = 1;
a[0][1] = 2;
a[0][2] = 3;
a[1][0] = 4;
a[1][1] = 5;
a[1][2] = 6;
```

Visualmente, el arreglo se ve de acuerdo a la siguiente matriz: 

```
1 2 3
4 5 6
```

Y el acomodo de memoria se ve como: 1 2 3 4 5 6 (donde 1 se encuentra en la posición de memoria más baja).

Entonces nuestro tercer parámetro para glUniformMatrix4fx() es GL_TRUE porque le dimos a la matriz un orden de fila ascendente. Tambien podríamos hacer que el tercer parámetro fuera GL_FALSE, pero si hicieramos eso tendríamos que transponer los valores de l amatriz (el acomodo de memoria en C/C++ se mantendrá igual pero OpenGL "pensará" que los primeros 4 valores que le demos serán una matriz de columna y así sucesivamente, y se comportará de acuerdo a esto). El cuarto parámetro es simplemente la dirección inicial de memoria de la matriz. 

El código restante es código de shader. 

`uniform mat4 gWorld;`

Esta es una variable uniforme de una matriz de 4x2. mat2 y mat3 también son tipos de matrices disponibles.

`gl_Position = gWorld * vec4(Position, 1.0);`

La posición de los vértices del triángulo en el búfer de vértices son vectores de 3 compontener, pero como ya vimos necesitamos un cuarto componente de valor 1. Tenemos dos opciones: tener vértices con 4 componentes en el búfer de vértices o agregar el cuarto componente al vertex shader. No hay ningún tipo de ventaja en la primer opción. Cada posición de vértice consumirá 4 bytes adicionales por un componente que sabemos siempre será 1. Es más eficiente permanecer con un vectores de 3 componentes y concatenar el componente w en el shader. En GLSL esto se hace usando 'vec4(Position, 1.0)'. Multiplicamos una matriz por el vector y el resultado se asigna a gl_Position. Para resumir, cada frame generamos una matrix de traslación que mueva la coordenada X por un valor que vaya entre -1 y 1. El shader multiplica la posición de cada vértcie por la matriz, lo cual resulta en que el objeto combinado se mueva de izquierda a derecha. En la mayoría de los casos, uno de los lados del triángulo irá fuera de la caja normaizada después del vertex shader y el clipper cortará ese lado. Solo podremos ver la región que se encuentra dentro de la caja normalizada. 

Para más información sobre este tema podemos seguir el siguiente [video tutorial de Frahaan Hussain](https://www.youtube.com/watch?v=aJRrgka4dpU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=11).
