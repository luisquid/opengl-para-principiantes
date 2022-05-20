## Contexto
Este tutorial es bastante corto. Simplemente expandiremos el tutorial anterior para poder dibujar un triángulo. 

En este tutorial nos apoyaremos en la caja normalizada de nuevo. Los vértices visibles deben estar dentro de la caja para que la transformación de la vista pueda mapearlas a las coordenadas visibles de la ventana. Cuando vemos hacia bajo en el axis negativo de Z la caja se ve de esta forma: 

Punto (-1.0, -1.0) se mapea al lado inferior izquierdo de la ventana, (-1.0, 1.0) se mapea en la parte superior izquierda, y así sucesivamente. Si extiendes la posición de uno de los vértices fuera de esta caja el triángulo se vería cortado y solo podríamos ver parte de este mismo. 

## El código paso a paso

```
Vector3f Vertices[3];
Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
Vertices[1] = Vector3f(1.0f, -1.0f, 0.0f);
Vertices[2] = Vector3f(0.0f, 1.0f, 0.0f);
```

Extendimos el arreglo para que contenga tres vértices. 

`glDrawArrays(GL_TRIANGLES, 0, 3);`

Realizamos dos cambios a la función de dibujado: dibujamos triángulos en vez de puntos y dibujamos 3 vértices en vez de 1. 

Para más información sobre este tema puedes revisar el siguiente [video tutorial por Frahaan Hussain](https://www.youtube.com/watch?v=EIpxcNl2WJU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=8). 