### Tutorial 8
## Transformaciones de Escala

#### English: https://ogldev.org/www/tutorial08/tutorial08.html

[Click acá](https://www.youtube.com/watch?v=pLFXNmbDZk8) para ver el video tutorial hecho por ogldev.

## Contexto
La transformación de escala es bastante simple. Su propósito es incrementar o disminuir el tamaño de un objeto. Nosotros tal vez queramos hacer eso para, por ejemplo, hacer que un mismo modelo parezca diferente simplemente cambiando su tamaño, o cuando querramos cambiar el tamaño de un objeto al tamaño que tendría en el mundo. Para todo esto necesitamos escalar los vértices de posición la misma cantidad de unidades en los tres ejes. Sin embargo, a veces solo querremos escalar uno o dos ejes, causando que el modelo se vea más grueso o delgado. 

Desarrollar la matriz de transformación es bastante sencillo. Empezamos con la matriz de identidad y recordamos que la razón de multiplicarla por un vector nos da el mismo vector es porque cada '1' en la matriz es multiplicado por uno de los componentes del vector. Ninguno de los componentes se afecta entre si. Por esto, remplazando cualquiera de los '1' por otro valor causará que el objeto crezca en dicho eje si el valor es más grande que 1 o que disminuya en el eje si el valor es meno que 1. 

## El Código Paso a Paso

```
World.m[0][0]=sinf(Scale); World.m[0][1]=0.0f;        World.m[0][2]=0.0f;        World.m[0][3]=0.0f;
World.m[1][0]=0.0f;        World.m[1][1]=sinf(Scale); World.m[1][2]=0.0f;        World.m[1][3]=0.0f;
World.m[2][0]=0.0f;        World.m[2][1]=0.0f;        World.m[2][2]=sinf(Scale); World.m[2][3]=0.0f;
World.m[3][0]=0.0f;        World.m[3][1]=0.0f;        World.m[3][2]=0.0f;        World.m[3][3]=1.0f;
```

El único cambio del tutorial anterior es que remplazamos la matriz de transformacion global de acuerdo a lo descrito arriba. Como podemos ver, escalamos cada uno de los tres ejes por un número que va de -1 y 1. En el rango (0,1) el triángulo puede ser o muy pequeño o tener su tamaño original y cuando la diagonal es cero desaparecerá completamente. En el rango [-1,0) se ve igual solo que invertido porque el valor de la escala en la diagonal cambió el de signo la posición. 

Para más información sobre este tema podemos seguir el siguiente [video tutorial de Frahaan Hussain](https://www.youtube.com/watch?v=aJRrgka4dpU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=11).

