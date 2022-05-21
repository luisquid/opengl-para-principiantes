### Tutorial 2
## ¡Hola Punto!

#### English: https://ogldev.org/www/tutorial02/tutorial02.html

## Contexto
Este es nuestro primer encuentro con GLEW, la Librería de Extension Wrangler de OpenGL. GLEW nos ayuda con el dolor de cabeza que puede ser el manejo de extensiones en OpenGL. Una vez inicializado consulta todas las extensiones disponibles en tu plataforma, las carga dinámicamente y te da acceso fácil a ellas usando un solo archivo header. 

En este tutorial veremos el uso de vertex buffer objects (VBOs) por primera vez. Como lo dice su nombre, son objetos que se utilizan para almacenar vértices. Los objetos que queremos visualizar dentro de nuestro entorno 3D, ya sean monstruos, castillo o un solo cubo girando, se construyen conectando grupos de vértices. VBOs son la forma más eficiente de cargar vértices en el GPU. Son buffers que se pueden almacenar en la memoria de video y nos da el tiempo de acceso más corto al GPU, por eso su uso es ampliamente recomendado. 

Este tutorial y el siguiente son los únicos de la serie que van a apoyarse en las funciones existentes del pipeline en vez de las que podamos programar. De hecho, no haremos transformaciones en ninguno de estos dos tutoriales. Nos apoyaremos únicamente en la forma con la que los datos fluyen por el pipeline. En los siguientes tutoriales haremos un análisis más profundo de este flujo, pero por ahora es suficiente entneder que antes de llegar a la parte de renderizador (el que se encarga de dibujar los puntos, las líneas y los triángulos usando coordenadas de pantalla), los vértices visibles tienen sus coordenadas en X, Y y Z en el rango [-1.0, 1.0]. El renderizador mapea estas coordeanadas en el espacio de pantalla (p.e, si el ancho de pantalla es 1024 entonces la coordenada X -1.0 será mapeada a 0 y 1.0 será mapaeda a 1023).
Finalmente, el renderizador dibuja las primitivas de acuerdo a la topología especificada en la llamada de dibujo (draw call) (ve abajo en el código paso a paso). Como no unimos ningún tipo de shader a nuestro pipeline los vértices no pasarán por niguna transformación. Esto significa que únicamente necesitamos darles un valor en el rango descrito previamente para poder visualizarlos. De hecho, si seleccionamos cero para ambos X y Y posicionará el vértice en el punto medio de ambos ejes - en otras palabras, en la mitad de la pantalla. 

Para instalar GLEW: GLEW está disponible en su página principal: http://glew.sourceforge.net/. La mayoría de las distribuciones de Linux proveen paquetes precompilados para la librería. En Ubuntu puedes instalarlo corriendo el siguiente comando: 

`apt-get install libglew1.6 libglew1.6-dev`

## El código paso a paso 

`#include <GL/glew.h>`

Aquí incluiremos el haeder de GLEW. Si incluyes otros headers de OpenGL debes tener cuidado de incluir este archivo antes de los demás o podrías tener problemas de compatibilidad. Para poder enlazar la aplicación con GLEW necesitamos '-lGLEW' al makefile. 

`#include "math_3d.h"`

Este archivo header está ubicado en 'ogldev/Include' y contiene las estructuras de ayuda como la estructura de vector. Expandiremos más el contenido de este header conforme vayamos avanzando en los tutoriales. Hay que asegurarnos de clonar el código fuente del repositorio de acuerdo a las instrucciones que podemos encontrar [aquí](https://ogldev.org/instructions.html). Considera también que cada carpeta de tutorial contiene un scrip llamado 'build.sh' que puede ser usado para compilar el tutorial. Si usas tu propio sistema de compilado usa ese script como referencia para las banderas de compilado/linker necesarias. 

```
GLenum res = glewInit();
if (res != GLEW_OK)
{
    fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
    return 1;
}
```
Aquí inicializamos GLEW y checamos por errores. Esto se debe de hacer después de que GLUT haya sido inicializado. 

```
Vector3f Vertices[1];
Vertices[0] = Vector3f(0.0f, 0.0f, 0.0f);
```
Creamos un arreglo de una estructura tipo Vector3f (este tipo de dato es definido en math_3d.h) e inicializamos XYZ en cero. Esto hará que el punto aparezca en medio de la pantalla. 

`GLuint VBO;`

Hacemos una alocación de un GLuint en la parte global del programa para almacenar el handle del objeto de búfer de vértice (VBO). Veremos más adelante que la mayoría (si no es que todos) los objetos en OpenGL son accesidos por variables de tipo GLuint. 

`glGenBuffers(1, &VBO);`

OpenGL define varias funciontes de tipo glGen* para generar objetos de diferentes tipos. Usualmente toman dos parámetros - el primero especifica el número de objetos que queremos creare y el segundo es la dirección del arreglo de GLuints que almacena los handles (manejadores) que el driver alocará para ti (¡asegurate que el arreglo sea lo suficientemente grande para manejar tu petición!). Llamadas futuras a esta función no generarán los mismos manejadores de objetos a menos que los borres primero con la función glDeleteBuffers. Algo que debemos considerar es que en este punto no estamos especificando lo que queremos hacer con los buffers para que puedan ser tratados con "genéricos". El trabajo de especificar la función de los buffers será del siguiente método. 

`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

OpenGL tiene una forma única de usar los handles. En muchas APIs los handles simplemente se pasan a cualquier función a la que puedan ser relevantes y luego se realiza la acción en ese handle. En OpenGL unimos el handle con un nombre de objetivo y ejecutamos los comandos en dicho objetivo. Estos comandos afectan al hanlde que fue adjuntado hastsa que otro sea adjuntado en su lugar o hasta que la llamada tome cero como manejador. El objetivo GL_ARRAY_BUFFER significa que el búfer contendrá un arreglo de vértices. Otro objetivo que puede ser útil es GL_ELEMENT_ARRAY_BUFFER que significa que el búfer contiene los índices de los vértices en otro búfer. Existen otros tipos de objetivos disponibles que estudiaremos en tutoriales más adelante. 

`glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);`

Después de unir nuestro objeto debemos llenarlo con información. La llamada que acabamos de escribir toma el nombre del objetivo (el mismo que fue usado en la unión), el tamaño de los datos en bytes, la dirección del arreglo de vértices y la bandera que indique el patrón de uso para estos datos. Como no vamos a cambiar el contenido del búfer debemos especificar GL_STATIC_DRAW. Para especificar lo opuesto sería GL_DYNAMIC_DRAW. Así como esto pueda ser único para OpenGL es bueno pensar bien en las banderas que debemos usar. El driver puede apoyarse en esta bander para optimizar heurísticas (tales como cual es el mejor lugar en memoria para almacenar el búfer).

`glEnableVertexAttribArray(0);`

En el tutorial de shaders veremos que los atributos de los vértices usados en el shader (posición, normal, etc) tienen un índice mapeado a ellos que nos permite create una unión entre los datos del programa de C/C++ y los nombres de los atributos dentro del shader. Adicionalmente debemos también habilitar el índice de cada atributo de vértice. En este tutorial todavía no utilizamos shaders, pero la posición de vértice que hemos cargado en el búfer será tratado como el índice del atributo de vértice 0 en la función fija del pipeline (que se activa cuando no hay un shader que limite el programa). Debes activar cada atributo de vértice o si no los datos no podrán ser accedidos por el pipeline. 

`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

Aquí unimos nuestro búfer de nuevo y nos preparamos para hacer el draw call. En este pequeño programa únicamente tenemos un búfer de vértice, entonces hacer esta llamada cada frame es redundante, pero en programas más complejos hay múltiples búfers para almacenar varios modelos y debemos actualizar el estado del pipeline con cada búfer que penemos utilizar. 

`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);`

This call tells the pipeline how to interpret the data inside the buffer. The first parameter specifies the index of the attribute. In our case we know that it is zero by default but when we start using shaders we will either need to explicitly set the index in the shader or query it. The second parameter is the number of components in the attribute (3 for X, Y and Z). The third parameter is the data type of each component. The next parameter indicates whether we want our attribute to be normalized before it is used in the pipeline. It our case we want the data to pass un-changed. The fifth parameter (called the 'stride') is the number of bytes between two instances of that attribute in the buffer. When there is only one attribute (e.g. the buffer contains only vertex positions) and the data is tightly packed we pass the value zero. If we have an array of structures that contain a position and normal (each one is a vector of 3 floats) we will pass the size of the structure in bytes (6 * 4 = 24). The last parameter is useful in the case of the previous example. We need to specify the offset inside the structure where the pipeline will find our attribute. In the case of the structure with the position and normal the offset of the position is zero while the offset of the normal is 12.

`glDrawArrays(GL_POINTS, 0, 1);`

Finalmente, hacemos la llamada a dibujar la geometría. Todos los comandos que hemos visto hasta ahora son importantes, pero solo nos ayudan a configurar el espacio para el comando de renderizado. Aquí es donde el GPU realmente empieza a trabajar. Combinará los parametros de cada draw call con el estado que hemos estado construyendo hasta este punto y hará el renderizado de los resultados en pantalla. 

OpenGL nos da diferentes tipos de draw calls y cada uno de ellos sirve para diferentes casos. En general podemos dividirlos en dos categorías - renderizado ordenado y renderizado indexado. El renderizado ordenado es más simple. El GPU manda los búfers de vértices, yendo uno por uno por cada vértice, y los interpreta de acuerdo a la topología especificada en el draw call. Por ejemplo, si especificamos GL_TRIANGLES entonces los vértices 0-2 se convierten en el primer triánguo, los vértices 3-5 en el segundo triángulo, etc. Si quieres que el mismo vértice aparezca en más de un triángulo deberás especificarlo dos veces en el búfer de vértices, lo cual es un desperdicio de espacio. 

El renderizado indexado es más complejo e involucra un búfer adicional llamado búfer de índice (index buffer). El index buffer contiene los índices de los vértices en el búfer de vértices. El GPU escanea el index bufer y, de forma similar a como lo hace con el renderizado ordenado, los índices 0-2 se convierten en el primer triángulo y así sucesivamente. Si quieres el mismo vértice en dos triángulos diferentes únicamente debes especificar su índice dos veces en el index buffer. El búfer de vértices solo necesita una copia. El dibujado de índices es más común en juegos porque la mayoría de los modelos son creados utilizando triángulos que representen una superficie (piel en el caso de una persona, una pared de un castillo o escenario, etc.) con una cantidad grande de vértices compartidos entre si. 

En este tutorial usamos el draw call más simple - glDrawArrays. Este es un tipo de renderizado ordenado, entonces no necesimtamos un index buffer. Debemos especificar la topología de los puntos, lo que significa que cada vértice es un punto. El siguiente parámetro es el índice del primer vértice a dibujar. En nuestro caso queremos empezar al principio del búfer para poder especificar cero, y esto a su vez nos permite almacenar múltiples modelos en el mismo búfer y posteriormente seleccionar el que queremos dibujar basado en su desfase dentro del búfer. El último parámetro es el número de vértices a dibujar. 

`glDisableVertexAttribArray(0);`

Es buena práctica deshabilitar cada atributo del vértice cuando no es utilizado inmediatamente. Dejandolo habilitado cuando un shader no lo está utilizando es una forma segura de hacernos las cosas más difíciles. 