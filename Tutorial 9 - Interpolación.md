### Tutorial 9
## Interpolación

#### English: https://ogldev.org/www/tutorial09/tutorial09.html

[Click acá](https://www.youtube.com/watch?v=ZVgf_W-X8eM) para ver el video tutorial en inglés hecho por ogldev.

## Contexto

Este tutorial demuestra una parte muy importante del pipeline 3D - la interpolación que el renderizador realiza en variables que vienen del vertex shader. Como ya hemos visto, en orden para mostrar algo en pantalla necesitamos designar uno de los outputs del VS como 'gl_Position'. Este es un vector4 que contiene las coordenadas homogeneas del vértice. Los componentes XYZ de un vector se dividen por el componente W (un proceso conocido como perspectiva divida, y que veremos más adelante en su respectivo tutorial) y cualquier componente que vaya fuera de la caja normalizada ([-1,1]) se verá recortado. El resultado es transformado a coordenadas del espacio de pantalla y luego el triángulo (o cualquier otro tipo de primitiva soportada) se renderizan en la pantalla por el renderizador. 

El renderizador realiza la interpolación entre los tres vértices del triángulo (ya sea yendo línea por línea o usando cualquier otra técnica) y "visita" cada pixel dentro del triángulo ejecutando el fragment shader. Se espera que el fragment shader regrese un color de pixel que el renderizador posicione en el búfer de color para mostrarse en pantalla (después de pasar por un par de pruebas adicionales como la prueba de profundidad, etc). Cualquier otra variable que salga del vertex shader no seguirá estos mismos pasos. Si el fragment shader no pide explícitamente la variable (y podamos mezclar y combinar multiples fragment shaders con el mismo vertex shader) entonces la optimización común del driver hará que ignoremos cualquier instrucción en el VS que solo afecte esta variable (para el programa de shader en particula que combine este par de VS y FS). Sin embargo, si el FS usa 

The rasterizer performs interpolation between the three triangle vertices (either going line by line or any other technique) and "visits" each pixel inside the triangle by executing the fragment shader. The fragment shader is expected to return a pixel color which the rasterizer places in the color buffer for display (after passing a few additional tests like depth test, etc). Any other variable which comes out of the vertex shader does not go through the steps above. If the fragment shader does not explicitly requests that variable (and you can mix and match multiple fragment shaders with the same vertex shader) then a common driver optimization will be to drop any instructions in the VS that only affect this variable (for that particular shader program that combines this VS and FS pair). However, if the FS does use that variable the rasterizer interpolates it during rasterization and each FS invocation is provided a the interpolated value that matches that specific location. This usually means that the values for pixels that are right next to each other will be a bit different (though as the triangle becomes further and further away from the camera that becomes less likely).

Existen dos variables muy comunes en las que nos podemos apoyar en esta interpolación. Estas variables son la normal del triángulo y las coordenadas de textura. La normal del vértice se calcula como el promedio entre las normales del triángulo de todos los triángulos que incluyan dicho vértice. 

Two very common variables that often rely on this interpolation are the triangle normal and texture coordinates. The vertex normal is usually calculated as the average between the triangle normals of all triangles that include that vertex. If that object is not completely flat this usually means that the three vertex normals of each triangle will be different from each other. In that case we rely on interpolation to calculate the specific normal at each pixel. That normal is used in lighting calculations in order to generate a more believable representation of lighting effects. The case for texture coordinates is similar. These coordinates are part of the model and are specified per vertex. In order to "cover" the triangle with a texture you need to perform the sample operation for each pixel and specify the correct texture coordinates for that pixel. These coordinates are the result of the interpolation.

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