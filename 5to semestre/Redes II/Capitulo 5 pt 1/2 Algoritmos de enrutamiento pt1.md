Los algoritmos de enrutamiento son la parte del software de la capa de red encargada de decidir por qué línea de salida debe transmitirse un paquete que llega a un enrutador. 
# Propiedades 
Independientemente de si las rutas se eligen para cada paquete por separado o solo al establecer una conexión, un algoritmo de enrutamiento debe poseer las siguientes propiedades:

>[!example]
>- **Corrección y Simplicidad:** El algoritmo debe encontrar rutas válidas de forma sencilla.
>- **Robustez:** Debe ser capaz de lidiar con cambios en la topología (como fallos en enrutadores o líneas) y variaciones en el tráfico sin necesidad de reiniciar toda la red.
>- **Estabilidad:** El algoritmo debe alcanzar un estado de equilibrio (convergencia) y mantenerse en él.
>- **Equidad:** Debe buscar un balance para que las conexiones individuales reciban un trato justo en la red.
>- **Eficiencia:** Debe optimizar el uso de los recursos globales de la red.
# Objetivos de los algoritmos

El objetivo principal es **descubrir caminos** en la red para transportar paquetes desde el origen al destino. De manera más específica, los algoritmos suelen buscar:

- **Minimizar la distancia** que debe recorrer el paquete o el **número de saltos** necesarios para llegar al destino.
- **Mejorar el retardo** y reducir el **ancho de banda consumido** por cada paquete, lo que a su vez mejora la velocidad de transferencia y el rendimiento global de la red.
- Evitar la sobrecarga de ciertas líneas o routers mientras otros permanecen sin utilizar
## El principio de la optimizacion
El **principio de optimización** (o principio de optimalidad) es una afirmación general sobre las rutas óptimas que sirve como base para casi todos los algoritmos de enrutamiento, independientemente de la topología o el tráfico de la red.
# 1. La lógica principal
El principio establece que si un enrutador **J** se encuentra en el camino óptimo entre el enrutador **I** y el enrutador **K**, entonces el camino óptimo desde **J** hasta **K** también sigue esa misma ruta.

>[!info] ¿Por qué es así?
>Imagina que la ruta de **I** a **K** pasa por **J**. Si existiera un camino mejor (más corto o rápido) para ir de **J** a **K**, podrías "pegar" ese nuevo camino a la primera parte (de **I** a **J**) y tendrías una ruta mejor para ir de **I** a **K**. Esto contradice la idea original de que el primer camino era el "óptimo". Por lo tanto, cualquier parte de un camino óptimo debe ser, por definición, óptima también.
# 2. El Árbol Sumidero (Sink Tree)
Como consecuencia directa de este principio, si juntas todos los mejores caminos desde todos los orígenes hacia un destino específico, el resultado es una estructura llamada **árbol sumidero**.

- **Forma de árbol:** Se llama así porque todos los caminos convergen en el enrutador de destino (la raíz del árbol).
- **Sin bucles:** Una propiedad fundamental de los árboles es que **no contienen ciclos o bucles**. Esto garantiza que un paquete no se quede dando vueltas infinitamente y llegue a su destino en un número finito de saltos.
- **Métricas:** El concepto de "mejor" camino depende de lo que el algoritmo quiera optimizar: puede ser el menor número de saltos, la distancia geográfica, el menor retardo o el mayor ancho de banda.
# 3. ¿Por qué es importante?
El objetivo final de cualquier algoritmo de enrutamiento (como Dijkstra o Bellman-Ford) es **descubrir y utilizar estos árboles sumideros** para cada enrutador de la red.
Aunque en la práctica los fallos de routers o enlaces pueden hacer que diferentes nodos tengan ideas distintas sobre la topología, el principio de optimización proporciona el estándar o la "referencia" ideal para medir qué tan bien está funcionando un algoritmo de enrutamiento
## Algoritmo de Dijkstra 
Para aplicar el algoritmo, la red se representa como un grafo donde cada nodo es un enrutador y cada arista es un enlace de comunicación. A cada enlace se le asigna un **peso o costo no negativo**, que puede representar la distancia geográfica, el retardo medio, el ancho de banda o el tráfico promedio.
# Tipos de etiquetas
El algoritmo utiliza etiquetas en los nodos para rastrear el progreso. Estas etiquetas contienen la distancia desde el origen y el nodo predecesor (para poder reconstruir la ruta al final). Existen dos estados para estas etiquetas:

- **Provisionales**: Se asignan inicialmente a los nodos cuando se descubre un camino, pero aún es posible encontrar uno mejor.
- **Permanentes**: Una vez que el algoritmo confirma que una etiqueta representa el camino más corto posible desde el origen hasta ese nodo, la marca como permanente y su valor ya nunca cambia.

>[!example] Funcionamiento
>1. **Inicialización**: Se marca el nodo de origen como **permanente** con una distancia de 0. Todos los demás nodos se marcan como **provisionales** con una distancia de **infinito** y sin predecesor.
>2. **Examen de vecinos (Relajación)**: El algoritmo toma el nodo que acaba de hacerse permanente (llamado "nodo de trabajo") y examina a todos sus vecinos que aún sean provisionales.
>3. **Actualización de costos**: Para cada vecino, se calcula la distancia desde el origen sumando la etiqueta del nodo de trabajo y el costo del enlace hacia ese vecino. Si este nuevo valor es **menor** que la etiqueta actual del vecino, se actualiza su etiqueta con el nuevo valor y se marca al nodo de trabajo como su nuevo predecesor.
>4. **Selección del mínimo**: Tras examinar a los vecinos, el algoritmo busca entre **todos** los nodos provisionales del grafo aquel que tenga la **etiqueta de distancia más pequeña**.
>5. **Cierre de ciclo**: Ese nodo con la distancia mínima se convierte en **permanente** y pasa a ser el nuevo nodo de trabajo para la siguiente ronda.
>6. **Finalización**: Los pasos 2 a 5 se repiten hasta que el nodo de destino se vuelve permanente o todos los nodos han sido procesados.
La **inundación** (o _flooding_) es un algoritmo de enrutamiento estático y sencillo cuyo objetivo es hacer llegar un paquete a **todos los nodos** de la red.

## Inundacion
Bajo esta técnica, cada vez que un enrutador recibe un paquete entrante, lo retransmite por **todas sus líneas de salida**, excepto por aquella por la que el paquete llegó originalmente.

Su principal problema es que genera una **gran cantidad de paquetes duplicados**, lo que puede saturar la red o incluso crear bucles infinitos. Para evitar esto, se utilizan dos métodos de control:

1. **Límite de saltos (TTL):** Se incluye un contador en la cabecera de cada paquete que disminuye en uno en cada enrutador. Cuando el contador llega a cero, el paquete se descarta.
2. **Números de secuencia:** El enrutador de origen asigna un número de secuencia a cada paquete. Los enrutadores intermedios mantienen una lista de los paquetes que ya han procesado (identificados por origen y secuencia); si llega un paquete que ya está en la lista, no se vuelve a inundar.
# Ventajas y aplicaciones
A pesar de su ineficiencia en el uso de ancho de banda, tiene propiedades valiosas:
- **Extrema robustez:** Es ideal para aplicaciones militares o redes donde muchos nodos pueden fallar simultáneamente, ya que si existe un camino al destino, la inundación lo encontrará.
- **Métrica de camino más corto:** Como explora todos los caminos posibles en paralelo, garantiza que la primera copia en llegar al destino lo haga por la ruta más corta (menor retardo).
- **Simplicidad:** Los routers solo necesitan conocer a sus vecinos directos, no requieren una imagen completa de la topología.
- **Redes inalámbricas:** En estas redes, un mensaje transmitido por una estación es recibido naturalmente por todas las demás en su alcance, lo que facilita este proceso.
## Enrutamiento por vector de distancia
El **enrutamiento por vector de distancia** o (Bellman-Ford) es un algoritmo de enrutamiento dinámico en el que cada enrutador mantiene una tabla (un vector) que registra la mejor distancia conocida a cada destino y qué enlace debe utilizar para llegar allí.
# Funcionamiento básico
- **Intercambio local:** A diferencia de otros algoritmos, los enrutadores que utilizan vector de distancia solo intercambian información con sus **vecinos directos**.
- **Contenido de la tabla:** Cada entrada de la tabla de enrutamiento consta de dos partes: la línea de salida preferida para ese destino y una estimación de la distancia (métrica) hacia él.
- **Proceso de actualización:** Periódicamente, cada enrutador envía a sus vecinos una lista de sus distancias estimadas a todos los destinos. Cuando un enrutador recibe un vector de un vecino \(X\), y sabe que el retraso hacia \(X\) es de \(m\) unidades, calcula que puede llegar a cualquier destino \(i\) a través de \(X\) con una distancia total de \(X_i + m\). Si este nuevo cálculo resulta ser menor que la mejor distancia que ya tenía registrada, actualiza su tabla.

![[Screenshot 2026-08-14 125441.png|432]]
  
Para construir su nueva tabla, el Router J necesita combinar tres datos:
1. **Sus vecinos directos:** Mirando el mapa (_Network_), los vecinos conectados directamente a **J** son **A**, **I**, **H** y **K**.
2. **El retardo hacia sus vecinos:** J mide el tiempo de respuesta actual hacia cada vecino directo:
    - Retardo $J \to A = 8$
    - Retardo $J \to I = 10$
    - Retardo $J \to H = 12$
    - Retardo $J \to K = 6$
3. **Los vectores recibidos:** Cada vecino le envía a J una lista (vector) con los tiempos que ellos tardan en llegar a **todos** los nodos de la red (columnas **A, I, H, K**).
>[!info] ¿Cómo calcula J su nueva tabla?
Para cada destino posible en la red ($A, B, C, \dots, L$), Router J prueba pasar por cada uno de sus 4 vecinos y calcula el costo total usando la fórmula:
$$\text{Costo Total} = \text{Retardo de J al vecino} + \text{Retardo del vecino al destino final}$$
Luego, **elige el menor valor** y guarda por qué línea (interfaz) debe enviar el paquete.

>[!succes] Ejemplo
>1. Para enviar un paquete al destino **C**:
>- **Vía A:** $8 \text{ (de J a A)} + 25 \text{ (de A a C)} = 33$
>**Vía I:** $10 \text{ (de J a I)} + 18 \text{ (de I a C)} = \mathbf{28}$ $\leftarrow$ **¡Mínimo!**
>- **Vía H:** $12 \text{ (de J a H)} + 19 \text{ (de H a C)} = 31$    
>- **Vía K:** $6 \text{ (de J a K)} + 36 \text{ (de K a C)} = 42$
> **Resultado en la tabla de J para C:** Guarda el tiempo estimado **28** y la línea de salida **I**.

# El problema del conteo infinito
El **problema del conteo al infinito** es un inconveniente fundamental de los algoritmos de **enrutamiento por vector de distancia** (como RIP) que ocurre cuando el algoritmo reacciona con lentitud ante las "malas noticias", como la caída de un enlace o un enrutador.

**¿Por qué ocurre?**
El núcleo del problema es que, en el enrutamiento por vector de distancia, cuando un enrutador le informa a un vecino que tiene una ruta hacia un destino, el vecino **no tiene forma de saber si él mismo forma parte de esa ruta**

![[Pasted image 20260815150732.png|486]]

>[!example] Funcionamiento
>**Gráfico (a): Propagación de "Buenas Noticias"**
Representa el proceso cuando el nodo A entra en funcionamiento y la red aprende rápidamente a llegar a él:
>- **Al principio:** Los nodos B, C, D y E desconocen cómo llegar a A, por lo que sus distancias son infinitas ($\infty, \infty, \infty, \infty$).  
>- **1er intercambio:** B se conecta directamente a A y detecta que su distancia es de $1$ salto.  
>- **2do intercambio:** C recibe el vector de B. Sabiendo que B llega a A en 1 salto, C calcula que puede llegar a A pasando por B en $1 + 1 = 2$ saltos.  
>- **3er intercambio:** D recibe la actualización de C y calcula su costo como $2 + 1 = 3$ saltos.  
>- **4to intercambio:** E escucha a D y fija su distancia en $3 + 1 = 4$ saltos.  
> **Conclusión del caso (a):** La información positiva se propaga a razón de $1$ salto por intercambio. La red converge totalmente en solo $N-1$ pasos
> 

>[!danger] **Gráfico (b): Propagación de "Malas Noticias" (Conteo al infinito)**
Ilustra la falla en el razonamiento de los routers cuando **el nodo A cae o el enlace A-B se rompe**:
>- **Al principio:** La red está convergida con las distancias reales hacia A ($B=1, C=2, D=3, E=4$).
>- **1er intercambio:** B detecta que se rompió la conexión directa con A. Sin embargo, consulta el último vector recibido de C (donde C decía estar a distancia $2$ de A). B piensa erróneamente: _"Si C llega a A en 2 saltos, yo puedo ir a A a través de C en $2 + 1 = 3$ saltos"_. B actualiza su valor a **3**.
>- **2do intercambio:** C recibe la tabla de B. C solía llegar a A pasando por B, pero ahora ve que B dice estar a distancia 3, por lo que C calcula $3 + 1 = \mathbf{4}$.  
>- **3er intercambio:** B observa que la distancia de C subió a 4, así que B incrementa su valor a $4 + 1 = \mathbf{5}$. Al mismo tiempo, D nota que C subió a 4 y actualiza su costo a $4 + 1 = \mathbf{5}$.  
>- **4to intercambio:** C ve a B en 5 y recalcula $5 + 1 = \mathbf{6}$. E ve a D en 5 y recalcula $5 + 1 = \mathbf{6}$.  
>- **5to e intercambios posteriores:** B y D incrementan su valor a **7**, en el siguiente paso C y E suben a **8**, y así sucesivamente.

# Detalles técnicos y límites
- **Valor de infinito:** Debido a que los enrutadores deben "abrirse camino" hasta alcanzar el infinito para darse cuenta de que el destino es inalcanzable, se define un valor máximo para limitar este proceso. En el protocolo **RIP**, el valor de "infinito" es **16**.
- **Convergencia lenta:** Mientras que las "buenas noticias" (nuevos enlaces activos) se propagan rápidamente (un salto por intercambio), las malas noticias pueden tardar muchos intercambios en procesarse totalmente, lo que degrada el rendimiento de la red
# Algunos mecanismos de solucion
Para mitigar los fallos en el **enrutamiento por vector de distancia**, específicamente el problema del **conteo al infinito**, se han desarrollado diversos mecanismos técnicos que buscan acelerar la convergencia cuando ocurren "malas noticias" en la red.

Los principales mecanismos de solución mencionados en las fuentes son los siguientes:
- **Horizonte dividido (Split Horizon):** Este mecanismo prohíbe que un enrutador anuncie una ruta de regreso a través de la misma interfaz por la cual aprendió originalmente dicha ruta. Esto ayuda a evitar bucles de enrutamiento simples entre dos nodos adyacentes.
- **Envenenamiento de rutas (Route Poisoning):** Cuando un enrutador detecta que una ruta local ha fallado, en lugar de simplemente eliminarla, la mantiene de forma activa en sus actualizaciones pero le asigna inmediatamente el valor de **"infinito"**. Esto informa de manera explícita y rápida a los vecinos que el destino ya no es alcanzable.
- **Envenenamiento en reversa (Poison Reverse):** Es una variante o mejora del horizonte dividido. En este caso, el enrutador receptor envía una actualización de vuelta por la misma interfaz por la que aprendió la ruta, pero marcando la distancia como "infinito" para asegurar que no se creen rutas cíclicas.

Es importante notar que, a pesar de estos mecanismos, estos no siempre funcionan perfectamente en la práctica, ya que el problema fundamental persiste: cuando un enrutador recibe una ruta de un vecino, no tiene forma de saber con certeza si él mismo forma parte de ese camino anunciado.
 [[3 Algoritmos de enrutamiento pt2]]