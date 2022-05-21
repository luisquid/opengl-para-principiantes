### Tutorial 6
## Transformaciones de Posición

#### English: https://ogldev.org/www/tutorial06/tutorial06.html


## Background
En este tutorial comenzaremos a ver los diferentes tipos de transformaciones que un objeto en el entorno 3D y habilitaremos que nuestro programa pueda mostrarlo en la pantalla mientras mantenemos la ilusión de profundidad en la escena. La forma más común de hacer esto es representando cada transformación usnado una matriz, multiplicando cada una entre ellas y posteriormente multiplicando la posición del vértice po el producto final. Cada tutorial será dedicado a analizar cada transformación. 

Aquí echaremos un vistazo a la transformación de traslación, la cual es responsable te mover un objeto a través de un vector de longitud y dirección definida. Digamos que quieres mover el triángulo en la imagen de la izquierda a la posición de la derecha: 

Una forma de hacer esto es dando un vector de desfase (en este caso - 1,1) como una variable uniforma al shader y simplemente añadiendo la posición de cada uno de los vértices que vamos a procesar. Sin embargo, esto rompe el método de multiplicar un grupo de matrices en uno para obtener una única transformación. Adicionalmente, veremos después la traslación 

One way to do it is to provide the offset vector (in this case - 1,1) as a uniform variable to the shader and simply add it to the position of each processed vertex. However, this breaks the method of multiplying a group of matrices into one to get a single comprehensive transformation. In addition, you will see later that translation is usually not the first one so you will have to multiply the position by the matrix that represent the transformations before translation, then add the position and finally multiple by the matrix that represent the transformation that follow translation. This is too awkward. A better way will be to find a matrix that represents the translation and take part in the multiplication of all matrices. But can you find a matrix that when multiplied by the point (0,0), the bottom left vertex of the triangle on the left, gives the result (1,1)? The truth is that you can't do it using a 2D matrix (and you cannot do it with a 3D matrix for (0,0,0) ). In general we can say that what we need is a matrix M that given a point P(x,y,z) and a vector V(v1,v2,v3) provides M * P=P1(x + v1, y + v2, z + v3). In simple words this means that matrix M translates P to location P+V. In P1 we can see that each component is a sum of a component from P and the corresponding component of V. The left side of each sum equation is provided by the identity matrix:
I * P = P(x,y,z). So it looks like we should start with the identity matrix and find out the changes that will complete the right hand side of the sum equation in each component (...+V1, ...+V2, ...+V3). Let's see how the identity matrix looks like:

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

The position of the triangle vertices in the vertex buffer are vectors of 3 components, but we agreed that we need a fourth component with the value of 1. There are two options: place vertices with 4 components in the vertex buffer or add the fourth component in the vertex shader. There is no advantage to the first option. Each vertex position will consume an additional 4 bytes for a component which is known to be always 1. It is more efficient to stay with a 3 component vector and concatenate the w component in the shader. In GLSL this is done using 'vec4(Position, 1.0)'. We multiply the matrix by that vector and the result goes into gl_Position. To summarize, in every frame we generate a translation matrix that translates the X coordinate by a value that goes back and fourth between -1 and 1. The shader multiplies the position of every vertex by that matrix which results in the combined object moving left and right. In most cases the one of the triangles sides will go out of the normalized box after the vertex shader and the clipper will clip out that side. We will only be able to see the region which is inside the normalized box.

Para más información sobre este tema podemos seguir el siguiente [video tutorial de Frahaan Hussain](https://www.youtube.com/watch?v=aJRrgka4dpU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=11).
