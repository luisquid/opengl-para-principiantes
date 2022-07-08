### Tutorial 9
## Interpolación

#### English: https://ogldev.org/www/tutorial09/tutorial09.html

[Click acá](https://www.youtube.com/watch?v=ZVgf_W-X8eM) para ver el video tutorial en inglés hecho por ogldev.

## Contexto

Este tutorial demuestra una parte muy importante del pipeline 3D - la interpolación que el rasterizador reraliza en variables que vienen del vertex shader. Como ya hemos visto, en orden de visualiar algo en pantalla necesitamos designar una de las variables output del VS como 'gl_Position'. Este es un vector4 que contiene las coordenadas homogéneas del vértice. Los componentes XYZ de ese vector se dividen por el componente W (un proceso que se llama división de perspectiva y que veremos en su tutorial correspondiente) y cualquier componente que vaya fuera de la caja normalizada ([-1,1]) será cortado. El resultado se transforma a coordenadas del espacio de pantalla y luego el triángulo (o cualquier otro tipo de primitiva compatible) se renderizará en la pantalla por el renderizador. 

El renderizador realiza la interpolación entre los vértcies de los tres triángulos (ya sea yendo línea por línea o cualquier otra técnica) y "visita" cada pixel dentro del triángulo mediante la ejecución del fragment shader. El fragment shader se espera que regrese uncolor de pixel que el renderizador posicionará en el búfer de color para visualizarse (después de pasar un algunas pruebas adicionales como el depth test, etc). Cualquier otra variable que salga del vertex shader no deberá pasar por los pasos previamente mencionados. Si el fragment shader no pide explícitamente dicha variable (y puedes mezclar multiples fragment shaders con el mismo vertex shader) entonces una optimización común de drivers ocurrirá para dejar cualquier instrucción en el VS que solo afecte esta variable (para el shader program en particular que este par de VS y FS). Sin embargo, si el FS usa dicha variable el rasterizador interpolará durante la rasterización y cada invocación del FS se le dará un valor interpolado que corresponda con esa ubicación específica.  Esto usualmente significa que los valores por pixeles que existen a lado de cada uno serán un poco diferentes (aunque conforme el triángulo se aleje más de la cámara esto será menos probable que ocurra).

Dos variables comunes de las cuales depende esta interpoliación son la normal del triángulo y las coordenadas de textura. La normal del vértice se calcula usualmente con el promedio entre las normales del triángulo de todos los triángulos que incluyen el vértice. Si este objeto no está completamente plano esto significa que las tres normales de los vértices del triángulo serán diferentes entre ellas. En este caso dependemos en la interpolación para calcular la normal específica de cada pixel. La normal se usa en los cálculos de iluminación para poder generar una representación de efectos de luz más creíble. El caso para las coordenadas de la textura es muy similar. Estas coordenadas son parte del modelo y se especifican por vértice. En orden de "cubrir" el triángulo con una textura necesitamos realizar una operación de sampleo por cada pixel, y espeficiar las coordenadas de textura correctas para dicho pixel. Estas coordenadas son el resultado de la interpolación. 

En este tutorial veremos los efectos de la interpolación al interpolar entre diferentes colores en la cara del triángulo. Generaremos el color en el VS. Una forma un poco más tediosa sería recibir el color del búfer de vértices. Usualmente no le damos colores al búfer de vértices. Le damos coordenadas de texturas y mostramos un color de la textura. Este color será después procesado por los cálculos de iluminación. 

## El Código Paso a Paso

`out vec4 Color;`

Los parámetros que se pasan entre las etapas del pipeline deben ser declaradas usando la palabra reservada 'out' y estar declaradas en el scope global del shadre. El color es un vector4 porque los componentes XYZ llevan los valores RGB (respectivamente) y W lleva el valor alfa (transparencia).

`Color = vec4(clamp(Position, 0.0, 1.0), 1.0);`

El color en el pipeline de gráficos es usualmente representado usando un valor flotante en el rango de [0.0, 1.0]. El valor es después mapeado a un entero de 0 a 255 por cada canal de color (haciendo un total de 16M de colores). Asignamos el valor del color del vértice como una función de la posición del vértice. Primero, usaremos la función llamada clamp() para asegurarnos que los valores no vayan más allá de nuestro rango 0.0 - 1.0. La razón de hacer esto es que vértice de la parte inferior izquierda del triángulo está posicionado en -1,-1. Si tomamos estos valores para que el renderizador haga la interpolación tal y como estan, llegará un punto en el que X y Y pasarán 0 y no veremos nada porque todos los valores que sean menores o iguales a cero se renderizarán como negro. Esto significa que la mitad de la arista en cada dirección será negra antes de que el color pase cero y eso causará que veamos algo significativo en la cara. Al restringir estos valores nos aseguramos de que solo la parte inferior izquierda más lejana sea negra, pero conforme nos alejamos de este punto el color se vuelve más brillante. Intenta jugar con la función clamp - remuevela completamete o cambiando sus valores para ver que efectos tienen en el triángulo.

El resultado de la función clamp no irá directamente a la variable output porque es una variable de tipo vector4 mientras que la posición es de tipo vector3 (la función clamp no cambia el número de componentes, solo sus valores). Desde el punto de vista de GLSL no hay una conversión por defecto aquí y por eso mismo debemos de hacer la conversión explicita. Hacemos eso cambiando la notación 'vec4(vec3, W)' lo cual crea un vector4 al concatenar el valor de W al vector3. En nuestro caso usamos 1.0 porque este valor va al componente alfa del color y queremos que el pixel se vea enteramente. 

`in vec4 Color;`

El lado opuesto del color resultante en el VS es el color de entrada en el FS. Esta variable se interpola por el renderizador para que cada FS sea ejecutado con un color diferente. 

`FragColor = Color;`

Usamos el color interpolado como color de fragmento sin más cambios, y con eso terminamos este tutorial. 