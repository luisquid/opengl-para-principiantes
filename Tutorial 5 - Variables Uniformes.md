## Background
En este tutorial encontramos un nuevo tipo de variables de shader - las variables uniformes (uniform variables). La diferencia entre atributos y variables uniformes es que las variables de atributo contienen información que es específica del vértice, por lo que son recargadas con un nuevo valor del vertex buffer por cada invocation del shader, mientras que el valor de las variables uniforme se mantiene constante a través de todo el draw call. Esto significa que cargamos el valor antes de hacer el draw call y posteriormente podemos acceder al mismo valor en cada invocación del vertex shader. Las variables uniformes son útiles para almacenar datos como los parámetros de iluminación (posición de la luz, dirección, etc.), matrices de transformación, manejadores de objetos de textura, y entre otros. 

En este tutorial finalmente haremos que algo se mueva en pantalla. Lo haremos usando una variable uniforme cuyo valor será modificado cada frame, y la función de retorno idle que encontramos en GLUT. El punto de hacer esto es que GLUT no haga una llamdada a nuestra función de retorno del renderizado repetidamente - a menos que tenga que hacerlo. GLUT tiene una llamada a la función de retorno del renderizado después de eventos como el minimizado y maximizado de la ventana o sobreponemos otra ventana sobre la de nuestro programa. Si no cambiamos nada en el acomodo de nuestras ventanas a la hora de lanzar nuestra aplicación, entonces al función de retorno del renderizado se llamara solo una vez. Podemos verlo por nosotros mismos agregando una llamada a printf en la función de renderizado. Veremos que la función solamente es llamada una vez y veremos que se llama cada que minimizamos y maximizamos la ventana. Registrar nuestra función de retorno de renderizado en GLUT en los tutoriales pasados ha estado bien, pero aquí lo que buscamos es que el valor de nuestra variable sea modificado repetidamente. Podremos hacer esto registrando una función de retorno idle. La función idle es llamada por GLUT cuando no se reciben eventos del sistema de ventanas. Podemos tener una función dedicada para esta función de retorno donde podremos organizar elementos de nuestro programa como el tiempo de actualizado o simplemente registrar la función de retorno de renderizado como una función idle también. En este tutorial haremos lo segundo y actualizaremos la variable dentro de nuestra función de renderizado. 

## Source walkthru

```
glutPostRedisplay();
glutSwapBuffers();
```

Antes de la llamada existente a glutSwapBuffers dentro de nuestra función de retorno agregamos una llamada a glutPostRedisplay. En general, no necesitamos FreeGLUT para llamar nuestra función de renderizado varias veces. Solamente hace esto por los diferentes eventos que existen en el sistema. Como veremos más abajo, estamois creando una "animación" básica usando una variable que es actualizada en cada llamada a la función de renderizado, pero si esta función no se llama la animación parecerá que se congela. Por eso queremos lanzar la siguiente llamada a la función de renderizado y hacemos esto usando glutPostRedisplay. Esta función asigna una bandera dentro de FreeGLUT que forza el llamado de la función de renderizado una y otra vez. 

```
gScaleLocation = glGetUniformLocation(ShaderProgram, "gScale");
assert(gScaleLocation != 0xFFFFFFFF);
```

Después de adjuntar el programa, le hacemos una petición al objeto programa para obtener la locación de la variable uniforme. Este es otro ejemplo de un caseo en el que el ambiente de ejecución de la aplicación de C/C++ necesita ser mapeada al ambiente de ejecución del shader. Todavía no tenemos acceso directo al contenido del shader y no podemos acceder actualizar sus variables. Cuando compilamos el shader el compilador GLSL asigna el índice a cada una de las variables uniformes. En la representación interna del shader dentro del compilador resolvemos el acceso de la variable usando su índice. Este índice también está disponible para la aplicación mediante glGetUniformLocation. Podemos llamar esta función con el manejador del objeto programa y el nombre de la variable. La función nos regresa el índice o -1 si es que encuentra algún error. Es muy importante checar si hay algún error al llamar la función (como lo hacemos arriba con la aserción) o no podremos mandarle las actualizaciones de la variable posteriores al shader. Hay dos razones principales por la cual esta función puede fallar. La primera es que hayamos escrito el nombre de la variable incorrectamente o que haya sido optimizada por el compilador. Si el compilador GLSL encuentra que la variable no es usada en el shader simplemente se deshará de ella. En este caso glGetUniformLocation fallará. 

```
static float Scale = 0.0f;
Scale += 0.001f;
glUniform1f(gScaleLocation, sinf(Scale));
```

Mantenemos una variable estática de tipo flotante que incrementamos un poco cada que se llama la función de renderizado (tal vez sea bueno jugar con el valor de 0.001 si es que corre demasiado lento o demasiado rápido en nuestros computadores). El verdadero valor que se pasa al shader es el seno de la variable 'Scale'. Hacemos esto para crear un ciclo entre -1.0 y 1.0. Algo que debemos tomar en cuenta es que sinf() toma radianes como argumento y no grados, pero en este punto no debemos preocuparnos mucho por eso. Solo queremos los valores de tipo ola que genera la función de seno. El resultado de sinf() se pasa al shader usando glUniform1f. OpenGL nos da multiples instancias de esta función que siguen la forma general de glUniform{1234}{if}. Podemos usar esta función para cargar valores a variables vector 1D, 2D, 3D o 4D (basado en el número que siga después de 'glUniform') de tipo flotante o entero (para eso utilizamos el sufijo 'i' o 'f'). También hay versiones de la función que toman la dirección de un vector como parámetro, al igual que una versión especial enfocada a matrices. El primer parámetro de la función es la ubicación del índice que extrajimos usando glGetUniformLocation().

Ahora echaremos un vistazo a los cambios que se hicieron en el VS (el FS permanecerá sin cambios).

`uniform float gScale;`
Aquí declaramos el valor uniforme dentro del shader. 

`gl_Position = vec4(gScale * Position.x, gScale * Position.y, Position.z, 1.0);`
Multiplicamos los valores de X y Y de la posición del vector con el valor que es modificado dentro de la aplicación cada frame. ¿Podrías explicar por qué el triángulo se dibuja de cabeza la mitad del ciclo?