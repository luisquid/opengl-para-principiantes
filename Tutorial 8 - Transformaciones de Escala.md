### Tutorial 8
## Transformaciones de Escala

#### English: https://ogldev.org/www/tutorial08/tutorial08.html

[Click acá](https://www.youtube.com/watch?v=pLFXNmbDZk8) para ver el video tutorial hecho por ogldev.

## Contexto
The scaling transformation is very simple. Its purpose is to either increase or decrease the size of the object. You may want to do that, for example, when you want to create some differentiation using the same model (large and small trees that are actually the same) or when you want to match the size of the object to its role in the world. For the above examples you would probably want to scale the vertices position in the same amount on all three axis. However, sometimes you may want to scale just one or two axis, causing the model to become "thicker" or "leaner".

Developing the transformation matrix is very simple. We start with the identity matrix and remember that the reason that multiplying it by a vector leave the vector unchanged is that each of the '1's in the diagonal is multiplied by one of the components in turn. None of the components can affect the other. Therefore, replacing any one of that '1's with another value will cause the object to increase on that axis if the other value is larger than 1 or decrease on that axis if the other value is smaller then one.


## El Código Paso a Paso

```
World.m[0][0]=sinf(Scale); World.m[0][1]=0.0f;        World.m[0][2]=0.0f;        World.m[0][3]=0.0f;
World.m[1][0]=0.0f;        World.m[1][1]=sinf(Scale); World.m[1][2]=0.0f;        World.m[1][3]=0.0f;
World.m[2][0]=0.0f;        World.m[2][1]=0.0f;        World.m[2][2]=sinf(Scale); World.m[2][3]=0.0f;
World.m[3][0]=0.0f;        World.m[3][1]=0.0f;        World.m[3][2]=0.0f;        World.m[3][3]=1.0f;
```

El único cambio del tutorial anterior es que remplazamos la matriz de transformacion global de acuerdo a lo descrito arriba. Como podemos ver, escalamos cada uno de los tres ejes por un número que va de -1 y 1. En el rango (0,1) el triángulo no estará ni cerca de estar 
The only change from the previous tutorial is that we replace the world transformation matrix according to the above description. As you can see, we scale each of the three axis by a number that swings between -1 and 1. In the range (0,1] the triangle is anywhere between being very tiny and its original size and when the diagonal is zero it disappears completely. In the range [-1,0) looks the same only reversed because the scaling value in the diagonal actually changed the sign of the position.

Para más información sobre este tema podemos seguir el siguiente [video tutorial de Frahaan Hussain](https://www.youtube.com/watch?v=aJRrgka4dpU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=11).

