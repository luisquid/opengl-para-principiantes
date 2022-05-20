## Contexto
De este tutorial en adelante cada efecto y técnica que implmentaremos se realizará utilizando shaders. Los shaders son la forma moderna de hacer gráficos en 3D. De cierta forma podríamos decir que es dar un paso atrás ya que la mayoría de la funcionalidad 3D que se nos dió por las funciones fijas del pipeline y que requerían que el desarrollador solo especificara parametros de configuración (atributos de luz, valores de rotación, etc.) debe ahora ser implementado por el desarrollador (a través de shaders), sin embargo, esto nos permite tener más flexibilidad con nuestro programa y la capacidad de implementar elementos más inovadores. 

El pipeline de programación de OpenGL se puede visualizar de la siguiente forma: 


El procesador de vértices está a cargo de ejecutar el vertex shader (shader de vértice) en cada uno de los vértices que pase a través del pipeline (cuyo número está determinado de acuerdo a los parámetros del draw call). Los vertex shaders no tienen ningún conocimiento de la topología de las primitivas que se están renderizando. Adicionalmente, no podemos descartar vértices dentro del procesador de vértices. Cada vértice entra el procesador de vértices exactamente una vez, se le aplican las transformaciones correspondientes y continua por el flujo del programa.

El siguiente paso es el procesador de geometría. En este paso se le pasa al shader el conocimiento de la primitiva (p.e. todos los vértices de la figura), al igual que los vértices adyacentes. Esto habilita diferentes ténicas que deben tomar en cuenta información más allá del vértice mismo. El geometry shader (shader de geometría) también tiene la habilidad de cambiar la topología resultante a una diferente que la que fue seleccionada para el draw call. Por ejemplo, podríamos proporcionarle una lita de puntos y generar dos triángulos (p.e. un quad) de cada punto (una técnica llamada billboarding). Adicionalmente, tenemos la opción de emitir múltiples vértices por cada invocación del geometry shader, y así generar varias primitivas de acuerdo a la topología resultante seleccionada. 

Después de eso tenemos el clipper en nuestro flujo. Esta es una función fija de tipo unit con una tarea bastante directa - recorta las primitivas a la caja normalizada que habíamos visto en el tutorial anterior. También las recorta a los planos cercanos y lejanos en Z. También hay una opción de darle planos de recorte especificados por nosotros y que el clipper recorte a partir de ellos. La posición de los vértices que sobrevivan al recorte del clipper serán mapeadas a las coordenadas del espacio de pantalla y posteriormente el renderizador los dibujará en la pantalla de acuerdo a su topología. Por ejemplo, en el caso de los triángulos significaría encontrar todos los puntos que hay dentro del triángulo. Por cada punto el renderizador invoca el fragment processor. Aquí tienes la opción de determinar el color del pixel haciendo un muestreo de una textura o usando la técnica que queramos. 

Los tres pasos que podemos programar (vertex, geometry y fragment processors) son opcionales. Si no quieres adjuntar un shader a alguno de estos pasos se ejecutará algún tipo de funcionalidad por defecto. 

El manejo de shaders es muy similar a la creación de programas en C/C++. Primero escribimos el texto del shader y lo volvemos disponible para nuestro programa. Esto puede hacerse simplemente incluyedno el texto en un arreglo de caracteres en el código fuente, o cargandolo de un archivo de texto externo (de nuevo dentro del arreglo de caracteres). Luego compilas los shaders uno por uno para crear objetos de shader. Después de esto adjuntamos los shaders a un programa único y lo cargamos a la GPU. Adjuntar los shaders le da la oportunidad al driver de recortar los shaders y optimizarlos de acuerdo a la relación que haya entre ellos. Por ejemplo, podríamos emparejar un vertex shader que emita una normal con un fragment shader que la ignore. En este caso el compilador GLSL dentro del driver puede remover la funcionalidad relacionada con la normal del shader y permite una ejecución más rápida del vertex shader. Si posteriormente el shader se empareja con un fragment shader que use dicha normal, entonces adjuntar el otro programa generará un vertex shader diferente. 

## El código paso a paso

`GLuint ShaderProgram = glCreateProgram();`

Empezamos el o configurando nuestros shaders y creamos el objeto programa. Adjuntaremos todos los shader a este objeto.

`GLuint ShaderObj = glCreateShader(ShaderType);`

Creamos dos objetos shdaers usando esta llamada. Uno de ellos con un shader de tipo GL_VERTEX_SHADER y el otro de GL_FRAGMENT_SHADER. El proceso de especificar la fuente del shader y compilar el shader es el mismo para ambos. 

```
const GLchar* p[1];
p[0] = pShaderText;
GLint Lengths[1];
Lengths[0]= strlen(pShaderText);
glShaderSource(ShaderObj, 1, p, Lengths);
```

Antes de compilar el objeto shader debemos especificar su código fuente. La función glShaderSource toma el objeto shader como parámetro y nos da flexibilidad de especificar la fuente. La fuente puede ser distribuida a través de diferentes arreglos de caracteres y necesitamos darle un arreglo de arreglos, al igual que un arreglo de enteros donde cada espacio contenga el tamaño del arreglo de caracteres correspondientes. Para fines prácticos usamos un solo arreglo de chars para todo nuestro shader y usamos solo un espacio para el puntero que apunta a la fuente, al igual que su tamaño. El segundo parámetro a llamar es el número de espacios en los dos arreglos (solo 1 en nuestro caso).

`glCompileShader(ShaderObj);`

Compilar el shader es bastante sencillo...

```
GLint success;
glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);
if (!success) {
    GLchar InfoLog[1024];
    glGetShaderInfoLog(ShaderObj, sizeof(InfoLog), NULL, InfoLog);
    fprintf(stderr, "Error compiling shader type %d: '%s'\n", ShaderType, InfoLog);
}
```

... sin embargo, usualmente obtenemos varios errores de compilación. Este bloque de código nos da el estado de compilación y nos muestra todos los errores que el compilador encuentre. 

`glAttachShader(ShaderProgram, ShaderObj);`

Finalmente, adjuntamos el objeto shader ya compilado a nuestro objeto programa. Esto es muy similar a especificar la lista de objetos a adjuntar en el makefile. Como no tenemos un makefile aquí, emulamos este comportamiento mediante programación. Solo los objetos adjuntados toman parte del proceso de enlazado. 

`glLinkProgram(ShaderProgram);`

Después de compilar todos los objetos shader y de enlazarlos al programa finalmente podemos enlazar este objeto. Debemos notar que después de enlazar el programa podemos deshacernos de los objetos shader intermedios llamando la función glDetachShader y glDeleteShader para cada uno de ellos. El driver de OpenGL mantiene un contador de referencias en la mayoría de los objetos que genera. Si un objeto shader es creado y después borrado, el driver se deshará de él, pero si está adjuntado a un programa llamar glDeleteShader únicamente lo marcará para ser borrado y también deberemos llamar glDetachShader para que el contador de referencias regrese a cero y pueda ser eliminado.  

```
glGetProgramiv(ShaderProgram, GL_LINK_STATUS, &Success);
if (Success == 0) {
    glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
    fprintf(stderr, "Error linking shader program: '%s'\n", ErrorLog);
}
```
Hay que notar que checamos por errores relacionados al programa (p.e. errores de enlace) de una forma ligerament ediferent a errores relacionados con el shader. En vez de glGetShaderiv usamos glGetProgramiv y en vez de glGetShaderInfoLog usamos glGetProgramInfoLog. 

`glValidateProgram(ShaderProgram);`

En este punto tal vez nos podamos preguntar ¿para qué necesito validar un programa después de que fue enlazado exitosamente? La diferencia es que enlazarlo checa por errores basado en la combinación de los shaders, mientras que esta llamada checa si el programa puede ejecutarse dado el estado actual del pipeline. En una aplicación más compleja con múltiples shaders y varios cambios de estados es mejor validar antes de cada draw call. En nuestra app sencilla podemos checarlo solo una vez. También, tal vez querramos hacer esto únicamente en desarrollo y evitar este gasto extra de memoria en el producto final. 

`glUseProgram(ShaderProgram);`

Finalmente, para usar el shader program adjuntado lo necesitamos configurar dentro del estado del pipeline usando esta llamada. El programa permanecerá en efecto para todos los draw calls hasta que lo remplacemos con otro o hasta que desactivemos su uso explícitamente (y activar la función fija del pipeline) llamando glUseProgram con NULL. Si creamos un shader program que contenga solo un tipo de shader, luego otros estados operan usando su funcionalidad fija por defecto. 

Hemos completado los pasos de las llamadas de OpenGL relacionadas al manejo de shaders. El resto de este tutorial trata sobre el contenido de los vertex y fragment shaders (contenido en las variables 'pVS' y 'pFS'). 

`#version 330`

Esto le dice al compilador que estamos seleccionando la versión 3.3 de GLSL. Si el compilador no lo soporta nos mandará un error. 

`layout (location = 0) in vec3 Position;`

Esta declaración aparece en el vertex shader. Declara que un atributo especifico de un vértice, que es un vector de 3 flotantes, será llamado como 'Position' en el shader. 'Vertex specific' significa que por cada invocación del shader en el GPU, se proveerá el valor de un nuevo vértice del búfer. La primer sección de la declaración, layout (location = 0), crea la unión etnre el nombre del atributo y el atributo dentro del búfer. Esto es requerido para los casos en los que nuestro vértice contenga varios atributos (position, normal, texture coordinates, etc.). Debemos decirle al compilador que atributo en el vértices dentro del búfer debe ser mapeado al atributo declarado en el shader. Hay dos formas de hacer esto. Podemos asignarlo explícitamente como lo hacemos aquí (a cero). En ese caso podemos usar el valor que ha sido hard codeado en nuestra aplicación (como lo hicimos con el primer parámetro en llamar al glVertexAttributePointer). O podemos dejarlo fuera (y simplemente declarar 'in vec3 Position' en el shader) y después hacer la petición de la locación por la aplicación en ejecución usando glGetAttribLocation. En este caso necesitamos proporcionar el valor retornado a glVertexAttributePointer en vez de usar el valor hard codeado. Aquí elegimos el camino simple, pero para aplicaciones más complejas es mejor dejar que el compilador determine los índices de atributos y que los pida en tiempo de ejecución. Esto hace sencilla la integración de shaders de varias fuentes sin tener que adaptar cada una a nuestro buffer layout. 

`void main()`

Podemos crear un shader enlazando varios shader objects. Sin embargo, solo puede haber una función main por cada etapa del shader (VS, GS, FS) la cual es usada como punto de entrada del shader. Por ejemplo, podemos crear una librería de iluminado con diversass funciones y enlazarla nuestro shader siempre y cuando ninguna de las funciones incluidas se llame 'main'.

`gl_Position = vec4(0.5 * Position.x, 0.5 * Position.y, Position.z, 1.0);`

Aquí hacemos la transformación hard codeada a la posición del vértice que viene. Cortamos los valores de la X y Y por la mitad y dejamos Z sin cambios. 'gl_Position' es una variable incorporada especial que se supone ontiene la posición del vértice homogénea (contiene los componentes X, Y, Z y W). El renderizador buscará por esa variable y la usará como la posición en el espacio de pantalla (seguido pro varias transformaciones extra). Cortandso el valor de X y Y a la mitad significa que veremos un triángulo que es un cuarto del tamaño del triángulo del tutorial pasado. Hay que tomar en cuenta que estamos asignandole el valor de 1.0 a W. Esto es extremadamente importante para poder obetener el triángulo presentado correctamente. Para obtener la proyección de 3D a 2D es logrado en dos etapas diferentes. Primero necesitamos multiplicar todos nuestros vértices por la mátrix de proyección (la cual desarrollaremos algunos tutoriales adelante) y después el GPU automaticamente realiza lo que es conocido como división de perspectiva (perspective divide) a la atributo de posición ants de que llegue al renderizador. Esto significa que dividirá todos los componentes de gl_Position por el componente W. En este tutorial no estamos haciendo ningún tipo de proyección en el vertex shader todavía, pero la etapa de perspectiva dividia es algo que no podemos deshabilitar. Cualquier valor de gl_Position que podamos recibir del vertex shader será dividido por HW usando su componente W. Debemos recordar que de otra forma no obtendríamos el resultado que esperamos. En orden de eludir el efecto de la perspectiva dividida asignamos el valor de 1.0 a W. Divisiones entre 1.0 no afectarán otros componentes del vector de posición que permanecesrá dentro de nuestra caja normalizada. 

Si todo funcionó correctamente, tres vétrtices con los valores (-0.5, -0.5), (0.5, -0.5) y (0.0, 0.5) alcanzarán el renderizador. El clipper no necesita hacer nada porque todos los vértices están dentro de la caja normalizada. Estos valores son mapeados a las coordenadas del espacio de pantalla y el renderizador comienzar corriendo todos los puntos que hay dentro del triángulo. Por cada punto se ejecuta el fragment shader. El siguiente código de shader es tomada del fragment shader. 

`out vec4 FragColor;`

Usualmente el trabajo del fragment shader es el de determinar el color del fragmento (pixel). Adicionalmente, el fragment shader puede desacartar el pixel enteramente o cambiar su valor en Z (que afectará los resultados subsecuentes del Z test). Sacar el color se hace declarando la variable que acabamos de especificar. Los cuatro componentes representan R, G, B y A (para el valor de alpha). El valor que asignamos a esta variable será recibida por el renderizador y eventualmente escritas en el framebuffer. 

`FragColor = vec4(1.0, 0.0, 0.0, 1.0);`

En los tutoriales pasados no hubo un fragment shader por lo que todo fue dibujado de color blanco por defecto. Aquí configuramos FragColor a rojo. 

Para más información de este tema podemos revisar el siguiente [video tutorial por Frahaan Hussain](https://www.youtube.com/watch?v=aA112viAx7c&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=9).