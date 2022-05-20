## Telon de fondo 

El espectro de OpenGL no especifica ninguna API para crear y manipular ventanas. Sistemas de ventanas modernas que soportan OpenGL incluyen un sub-sistema que proporciona la unión entre un contexto OpenGL y el sistema de ventanas. En el sistema X Window la interfaz se llama GLX. Microsoft proporciona WGL (pronunciado: Wiggle) para Windows y MacOS tiene CGL. Al trabajar directamente con estas interfaces con el fin de crear una ventana en la que mostrar gráficos suele ser un trabajo duro y por eso utilizamos una biblioteca de alto nivel que abstrae los detalles finos. La biblioteca que usamos aquí se llama la 'OpenGL utility library', o GLUT. Proporciona una API simplificada para la gestión de ventanas, así como la gestión de eventos, control IO y algunos otros servicios. Además, GLUT es multiplataforma de modo que hace más fácil portabilidad. Alternativas a GLUT incluyen SDL y GLFW.  

## El código paso a paso 

`glutInit(&argc, argv);`

Esta funcion inicializa GLUT. Los parametros pueden ser previstos directamente de la linea de comandos y pueden incluir opciones utiles como    '-sync' y '-gldebug' los cuales desactivan la naturaleza asincrona de X y automaticamente checkean errores en GL y los muestra (respectivamente).  

`glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);`

Aquí configuramos algunas opciones de GLUT. GLUT_DOUBLE permite el doble buffer (dibuja en un búfer en segundo plano mientras se visualiza otro buffer) y el buffer de color donde la mayor representación termina (es decir, la pantalla). Por lo general vamos a querer estos dos, así como otras opciones que veremos más adelante.  

```
glutInitWindowSize(1024, 768);
glutInitWindowPosition(100, 100);
glutCreateWindow("Tutorial 01");
```

Estas llamadas especifican los parámetros de la ventana y la crean. Usted también tiene la opción de especificar el título de la ventana.  
 
`glutDisplayFunc(RenderSceneCB);` 
 
Dado que estamos trabajando en un sistema de ventanas la mayor parte de la interacción con el programa en ejecución se produce a través de funciones en respuesta a determinados eventos. GLUT se encarga de interactuar con el sistema de ventanas subyacente y nos ofrece algunas opciones de devolución de llamada. Aquí se utiliza sólo una devolucion de llamada (callback) denominada "principal" (RenderSceneCB) para hacer todo el renderizado de un frame. Esta función se llama continuamente por bucle interno de GLUT.  
 
`glClearColor(0.0f, 0.0f, 0.0f, 0.0f);` 
 
Este es nuestro primer encuentro con el concepto de estado en OpenGL. La idea detrás de este estado es que la representación es una tarea tan compleja que no puede ser tratada como una llamada de función que recibe un par de parámetros (y funciones diseñadas correctamente nunca reciben una gran cantidad de parámetros). Es necesario especificar shaders, buffers y diversas banderas que afectan la forma en que la represtación se llevará a cabo. Además, usted suele desear conservar la misma pieza de configuración a través de varias operaciones de representación (por ejemplo, si usted nunca deshabilita la prueba de profundidad, entonces no hay necesidad de especificarla que para que ocurra en cada llamada al render). Es por eso que la mayor parte de la configuración de las operaciones de renderizado se realiza mediante el establecimiento de indicadores y valores en la máquina de estados de OpenGL y las llamadas de representación por sí mismas se limitan generalmente a los pocos parámetros que giran en torno al número de vértices para dibujar y su desplazamiento inicial (offset). Después de llamar a una función de cambio de estado, la configuración particular se mantiene intacta hasta la siguiente llamada a la misma función con un valor diferente. La función citada establece el color que se utilizará para despejar el framebuffer (descrito más adelante). El color tiene cuatro canales (RGBA) y se especifica como un valor normalizado entre 0.0 y 1.0.  

`glutMainLoop();` 

Esta llamada le otorga el control a GLUT que ahora comienza su propio bucle interno. En este bucle se escucha a los eventos del sistema de ventanas y los pasa a través de las devoluciones de llamada (callback) que hemos configurado. En nuestro caso GLUT sólo llamará a la función se registró como un callback display (RenderSceneCB) para darnos una chace para renderizar el frame.  

```
glClear(GL_COLOR_BUFFER_BIT);
glutSwapBuffers(); 
```

Lo único que hacemos en nuestra función de hacer es limpiar el framebuffer (con el color especificado anteriormente - pruebe a cambiar). La segunda llamada le dice a GLUT que intercambie los roles de backbuffer y frontbuffer. En la siguiente llamada de la función vamos a renderizar el frame en el frontbuffer y se mostrará el backbuffer. 