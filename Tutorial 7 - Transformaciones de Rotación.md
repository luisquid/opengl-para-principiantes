### Tutorial 7
## Transformaciones de Rotación

#### English: https://ogldev.org/www/tutorial07/tutorial07.html


## Contexto
La siguiente transformación en nuestra lista es la de rotación, en la cual dado un ángulo y un punto queremos rotar el punto alrededor de uno de los ejes. Siempre cambiaremos dos de las tres coordenadas X, Y y Z y dejaremos el tercer componente sin modificar. Esto significa que el camino a seguir en la rotación seguira uno de los tres planos mayores: XY (cuando giramos alrederdor de Z), YZ (cuando giramos alrededor de X) y XZ (cuando giramos alrededor de Y). Hay transformaciones de rotación más complejas que nos permitirán rotar alrededor de un vector arbitrario, pero no las necesitamos aún en este punto. 

Definamos el problema en términos generales. Consideremos el siguiente diagrama: 


Queremos movernos 

Queremos movernos alrededor del círculo de (x1,y1) a (x2,y2). En otras palabras, queremos rotar (x1,y1) por el ángulo a2. Asumamos que el radio del círculo es 1. Esto significaría lo siguiente:

Usaremos las siguientes identidades trigonométricas para desarrollar x2 y y2:

Usando lo descrito anteriormente podemos escribir:

En el diagrama de arriba vemos el plano XY y Z apuntando a la página. Si X&Y son parte de un vector4 entonces la ecuación descrita arriba peude ser escrita en forma de matriz (sin afectar Z&W):

Si quereos crear rotaciones para los planos YZ (alrededor del eje X) y XZ (alrededor del eje Y) entonces las ecuaciones son básicamente las mismas pero la matriz es acomodada un poco diferente. Aquí está la matriz de rotación alrededor del eje Y:

Y la rotación de la matriz alrededor del eje X:


## El Código Paso a Paso

Los cambios en el código son pocos en este tutorial. Solamente cambiamos el contenido de una matriz de transformación en el código. 

```
World.m[0][0]=cosf(Scale); World.m[0][1]=-sinf(Scale); World.m[0][2]=0.0f; World.m[0][3]=0.0f;
World.m[1][0]=sinf(Scale); World.m[1][1]=cosf(Scale);  World.m[1][2]=0.0f; World.m[1][3]=0.0f;
World.m[2][0]=0.0f;        World.m[2][1]=0.0f;         World.m[2][2]=1.0f; World.m[2][3]=0.0f;
World.m[3][0]=0.0f;        World.m[3][1]=0.0f;         World.m[3][2]=0.0f; World.m[3][3]=1.0f;
```

Como podemos ver, podemos rotar alrededor del eje Z. Podríamos tratar otras rotaciones, pero creo que en este punto sin tratar de hacer verdadera proyección de 3D a 2D, las otras rotaciones se ven un tanto extrañas. Completaremos estas transformaciones en una clase de transformación del pipeline en los siguiente tutoriales.

Para más información sobre este tema podemos seguir el siguiente [video tutorial de Frahaan Hussain](https://www.youtube.com/watch?v=aJRrgka4dpU&list=PLRtjMdoYXLf6zUMDJVRZYV-6g6n62vet8&index=11).

