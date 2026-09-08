Es la práctica de conectar dos o más redes físicas distintas e independientes para formar una sola red unificada y más grande, comúnmente llamada _internetwork_ o simplemente _internet_.

La necesidad de unir redes es fundamentalmente económica. Según la **Ley de Metcalfe**, el valor de una red es proporcional al cuadrado de su número de nodos $N^2$. Esto significa que combinar varias redes pequeñas para formar una inter-red incrementa exponencialmente su valor y su utilidad práctica al permitir la comunicación universal entre ellas.
# Overview de Internetworking (Interconexión de redes)
Esta sección ofrece una perspectiva de alto nivel sobre cómo conectar múltiples redes físicas e independientes para formar una sola inter-red unificada:

- Introduce el desafío de la **heterogeneidad**, explicando cómo las redes difieren en aspectos críticos como el direccionamiento, los tamaños máximos de paquete (MTU), la seguridad y los niveles de fiabilidad.
- Explica de manera general cómo una **capa común de indirección** (el protocolo IP) actúa como un formato universal que permite que los paquetes atraviesen de manera fluida redes tecnológicamente distintas (como 802.11, MPLS y Ethernet)
# ¿Cómo difieren las redes?

Lograr que múltiples redes trabajen juntas es extremadamente complejo porque la **heterogeneidad** es una característica permanente en la tecnología. Las redes individuales difieren drásticamente en la capa de red en aspectos como:

- **Servicios ofrecidos:** Algunas redes son sin conexión (como IP orientada a datagramas) y otras son orientadas a la conexión (como los Circuitos Virtuales, ATM o MPLS).
- **Direccionamiento:** Utilizan diferentes tamaños de dirección, formatos planos o jerarquías.
- **Capacidad de difusión:** Algunas redes soportan difusión (_broadcast_) o multidifusión (_multicast_) y otras no.
- **Tamaño del paquete (MTU):** Cada red tiene un tamaño máximo de paquete permitido para la transmisión.
- **Seguridad, Fiabilidad y QoS:** Poseen diferentes niveles de pérdida de paquetes, métodos de cifrado o mecanismos de calidad de servicio.
## Opciones de diseño para conectar redes heterogéneas
Para solucionar estas diferencias y permitir que los paquetes viajen a través de redes extranjeras, existen dos filosofías de diseño básicas:
# Opción A: Traducción mediante Pasarelas (_Gateways_)

Consiste en colocar dispositivos en los límites de las redes que traduzcan y conviertan directamente los paquetes de un formato de red al de la otra red.

- **Inconvenientes:** La conversión directa suele ser incompleta y estar condenada al fracaso si los protocolos son muy diferentes. Por ejemplo, las direcciones IPv6 de 128 bits no caben físicamente en un campo de dirección IPv4 de 32 bits, por mucho que se intente. De igual forma, traducir entre una red sin conexión y una orientada a la conexión es sumamente complejo y rara vez se intenta con éxito debido a estas dificultades.
# Opción B: Añadir una Capa Común (La Solución IP)
Propuesta por Vint Cerf y Bob Kahn en 1974, esta opción añade una **capa común de indirección** por encima de todas las redes heterogéneas. El protocolo **IP (Internet Protocol)** sirve como este formato universal de paquetes que todos los enrutadores reconocen.
- En este modelo, existe una diferencia crítica entre la **conmutación** y el **enrutamiento**:
    - **Switches y Puentes (Capa 2):** Conectan el mismo tipo de red transportando la trama completa basándose únicamente en la dirección física MAC.
    - **Routers (Capa 3):** Conectan redes diferentes; extraen el paquete IP de la trama física en cada límite, examinan la dirección IP común del paquete y eligen de forma dinámica la mejor ruta hacia el destino.
## Técnicas clave en Internetworking
	
Para que el tráfico fluya eficazmente entre las diferentes redes de una inter-red se requieren tres mecanismos esenciales: 
# A. Tunelización (_Tunneling_)
Es una tecnica que permite enviar paquetes de un protocolo específico a través de una red intermedia que utiliza un protocolo de tránsito diferente o incompatible.

Conceptualmente, consiste en **colocar un paquete completo de una red dentro de la carga útil (cuerpo) de otro paquete compatible con la red de tránsito**. 
#### ¿Cómo funciona la mecánica del Tunneling?

Imagine que un host ubicado en una red de París desea enviar datos a un host de Londres, y ambas oficinas operan con **IPv6**, pero la infraestructura de Internet que las conecta en medio solo soporta **IPv4**. El proceso ocurre de la siguiente manera:

1. **Construcción del paquete original:** El host emisor en París construye un paquete IPv6 dirigido al destino en Londres.
2. **Encapsulamiento en el borde:** El paquete llega al router multiprotocolo en el borde de la red de París. Este router, en lugar de intentar traducir la dirección IPv6 a IPv4 (lo cual es inviable debido a la diferencia de tamaño de dirección), **encapsula el paquete IPv6 original dentro del campo de datos de un nuevo paquete IPv4**.
3. **Tránsito por el túnel:** La cabecera IPv4 externa tiene como dirección de origen el router de París y como destino el router de Londres. Para todos los routers intermedios en la Internet IPv4, este es un paquete de datos IPv4 normal y corriente; la cabecera IPv6 interna permanece oculta y no afecta el reenvío.
4. **Desencapsulamiento:** Cuando el paquete llega al router multiprotocolo en Londres (el extremo del túnel), este remueve la cabecera IPv4 externa, recuperando el paquete IPv6 original intacto.
5. **Entrega final:** El router de Londres envía el paquete IPv6 directamente al host destino dentro de su red local.

>[!example] La analogía del Chunnel
Para entenderlo mejor, las fuentes proponen la analogía de una persona que conduce su coche de París a Londres:
>- Dentro de Francia, el coche se desplaza por sus propios medios.
>- Al llegar al Canal de la Mancha, el coche no puede circular por el agua ni por las vías del tren directamente, de modo que es **cargado en un vagón de tren de alta velocidad** (encapsulado).
>- El tren transporta el coche a través del túnel (_Chunnel_) como si fuera simple carga.
>- Al llegar al otro extremo en Inglaterra, el coche es descargado (desencapsulado) y continúa circulando de forma autónoma por las carreteras inglesas.

#### Aplicaciones principales del Tunneling
- **Transición tecnológica (ej. IPv6 sobre IPv4):** Permite implementar nuevas características o protocolos de red de forma progresiva sin tener que actualizar simultáneamente todos los routers del núcleo de Internet.
- **Redes Privadas Virtuales (VPN):** Se utiliza el tunneling para construir redes superpuestas (_overlay_) sobre la infraestructura pública de Internet. Al encapsular y cifrar todo el tráfico entre las oficinas corporativas, se simula una red privada de líneas alquiladas mucho más económica y flexible.
- **Modo Túnel en IPsec:** Utilizado comúnmente en las VPN de cortafuegos corporativos. En este modo, **todo el paquete IP original (incluyendo su cabecera de red)** se encapsula dentro de un paquete IP completamente nuevo. Al cifrar el paquete interno con protocolos como ESP, un atacante externo no solo es incapaz de leer el contenido, sino que tampoco puede realizar análisis de tráfico para deducir quién se comunica con quién dentro de la organización.
#### Desventajas y limitaciones del Tunneling
- **Aislamiento del tránsito:** Un paquete que viaja en un túnel no puede salir a mitad del camino; es decir, no puede interactuar con ninguno de los hosts de la red de tránsito intermedia por la que viaja encapsulado. Aunque para la seguridad de una VPN esto es una ventaja, limita la flexibilidad de comunicación general.
- **Sobrecarga (_Overhead_):** El modo túnel requiere añadir una cabecera de red adicional en cada paquete. Esto incrementa sustancialmente el tamaño total de los paquetes, lo que puede provocar que superen la MTU del enlace físico de tránsito y forzar una ineficiente fragmentación de paquetes en los routers intermedios.
# B. Enrutamiento a traves de multiples redes

Del enrutamiento a través de múltiples redes (también conocido como _internetwork routing_) es el proceso de dirigir paquetes de datos a través de una colección de redes físicamente distintas e independientes que están interconectadas. Estas redes individuales suelen denominarse **Sistemas Autónomos (AS)** o redes autónomas.

Para resolver las diferencias tecnológicas y comerciales entre estas redes heterogéneas, la arquitectura de Internet utiliza un esquema de **enrutamiento en dos niveles**.

>[!success] Las complicaciones
Enrutar paquetes cruzando fronteras de redes independientes introduce desafíos complejos que no existen dentro de una sola red homogénea:
>- **Múltiples algoritmos y métricas incompatibles:** Una red puede utilizar un protocolo de estado de enlace (como OSPF) y otra uno de vector de distancia (como RIP). Además, las métricas de costo no son comparables directamente; un operador puede definir sus pesos según el retardo de transmisión y otro según criterios monetarios.
>- **Privacidad e información sensible:** Los operadores de red (generalmente competidores comerciales) no desean exponer información interna sobre el estado de sus colas, topología o métricas financieras a otras redes ajenas.
>- **Escalabilidad:** Los algoritmos de enrutamiento tradicionales no escalan al tamaño masivo de una inter-red global como Internet.

>[!tip] La solución: Enrutamiento en Dos Niveles
Para abordar estos retos, se divide el trabajo de enrutamiento en dos jerarquías bien definidas:
>- **Protocolos Intradominio (IGP - Interior Gateway Protocol):** Son los protocolos que se ejecutan **dentro de cada red autónoma** de forma independiente. Cada operador es libre de configurar y optimizar internamente su red con el protocolo que prefiera (como OSPF o IS-IS).
 >- **Protocolos Interdominio (EGP - Exterior Gateway Protocol):** Son los protocolos utilizados para encontrar rutas **entre las distintas redes autónomas** que componen la inter-red. A diferencia de los IGP, aquí todos los operadores están obligados a utilizar el mismo estándar común. En Internet, el estándar exclusivo para esta tarea es el **BGP (Border Gateway Protocol)**.

# C. Fragmentacion de paquetes 
Cada red física en el camino de un paquete tiene una **Unidad de Transmisión Máxima (MTU)**, que es el tamaño máximo de paquete que puede transportar. 

>[!info] Los límites de la MTU se deben a factores como:
>1. **Hardware:** Límites físicos del medio (por ejemplo, Ethernet soporta un máximo de 1500 bytes y el estándar inalámbrico 802.11 soporta hasta 2272 bytes).
>2. **Sistemas Operativos:** Tamaño de los búferes internos (frecuentemente múltiplos de 512 bytes).
>3. **Protocolos:** El número de bits dedicados en la cabecera para especificar el campo de longitud del paquete.
>4. **Estándares internacionales** o el deseo de **reducir retransmisiones** causadas por errores (un paquete más pequeño tiene menos probabilidad de corromperse).

Cuando un paquete grande (por ejemplo, uno de 2272 bytes originado en una red Wi-Fi) tiene que cruzar una red con una MTU más pequeña (como Ethernet, con 1500 bytes), se produce un cuello de botella. La única opción para no descartar el paquete es **fragmentarlo** en trozos más pequeños.
# Fragmentación Transparente vs. No Transparente
#### A. Fragmentación Transparente

- **Mecánica:** Cuando un paquete llega a una red con una MTU pequeña, el router de entrada lo divide en fragmentos. Todos estos fragmentos se dirigen de forma obligatoria al **mismo router de salida de esa subred**, el cual tiene la tarea de recolectar todos los fragmentos, reconstruir el paquete original y enviarlo a la siguiente red como si nada hubiera pasado.
- **Problemas:**
    - Requiere que el router de salida sepa cuándo ha recibido todas las piezas (exige un campo de recuento o un bit de "fin de paquete").
    - Restringe el enrutamiento: obliga a que todos los fragmentos viajen por el mismo camino para salir por el mismo router, lo que puede saturar ciertos enlaces.
    - Genera un alto esfuerzo de procesamiento y almacenamiento temporal en los routers intermedios, lo cual puede ser un desperdicio si el paquete vuelve a pasar por otra red pequeña más adelante y debe fragmentarse de nuevo.

#### B. Fragmentación No Transparente (La solución usada por IP)

- **Mecánica:** Una vez que un router divide un paquete grande, **nunca se vuelve a ensamblar en el camino**. Cada fragmento se trata a partir de ese momento como un paquete independiente y completo en la capa de red. Los routers intermedios simplemente los reenvían.
- **Reensamblado final:** El proceso de volver a unir las piezas ocurre **únicamente en el host de destino final**.
- **Ventaja:** Los routers intermedios trabajan mucho menos, acelerando el tránsito global.
# Mecánica Detallada de la Fragmentación en IPv4

Para que el host de destino pueda reensamblar un paquete fragmentado de forma no transparente (incluso si los fragmentos llegan desordenados), la cabecera IPv4 utiliza tres campos clave:

>[!example]
>1. **Identificación (16 bits):** Es un número único asignado por el origen a cada datagrama. Todos los fragmentos derivados del mismo paquete original compartirán este mismo identificador para que el destino sepa que pertenecen al mismo grupo.
>2. **Bit MF (More Fragments - Más Fragmentos):** Se pone a **1** en todos los fragmentos excepto en el último, que se pone a **0**. Esto permite al host de destino saber cuándo ha llegado la última pieza.
>3. **Desplazamiento de fragmento (Fragment Offset - 13 bits):** Indica la posición exacta en bytes que ocupa la carga útil de ese fragmento con respecto al paquete original.
>- La unidad elemental de desplazamiento es de **8 bytes**. Por lo tanto, todos los fragmentos (excepto el último) deben contener un tamaño de datos que sea múltiplo de 8 bytes.

> **Ejemplo práctico de fragmentación IPv4:** Supongamos un paquete original de 10 bytes de datos con número de identificación 27. Si pasa por una red con una MTU que solo permite fragmentos de máximo 8 bytes de datos más la cabecera, se divide en dos:
> 
> - **Fragmento 1:** Lleva el identificador `27`, desplazamiento `0`, bit `MF = 1` y los primeros 8 bytes de datos (A, B, C, D, E, F, G, H).
> - **Fragmento 2:** Lleva el identificador `27`, desplazamiento `8` (en bytes/8 = 1 unidad de desplazamiento), bit `MF = 0` y los últimos 2 bytes de datos (I, J).

#### El grave problema de la fragmentación

A pesar de su utilidad, los investigadores de red (como Kent y Mogul) demostraron que **la fragmentación degrada severamente el rendimiento**. La razón principal es que IP no tiene mecanismos de recuperación de errores individuales por fragmento. **Si se pierde un solo fragmento de un paquete grande, todo el paquete original se pierde** y el emisor (generalmente controlado por TCP) se ve obligado a retransmitir todo el paquete completo, multiplicando el tráfico de la red innecesariamente.
# El Enfoque Moderno: Descubrimiento de la MTU de la Ruta (Path MTU Discovery)

Para evitar que los routers intermedios tengan que fragmentar paquetes, la Internet moderna utiliza el algoritmo de **Descubrimiento de la MTU de la ruta (PMTUD)**. Su objetivo es que el host emisor averigüe de antemano el tamaño del enlace más pequeño de todo el camino (la MTU del camino) y ajuste el tamaño de sus paquetes antes de enviarlos.

# Mecánica de funcionamiento en IPv4:

1. **Envío con DF=1:** El host de origen construye sus paquetes IP y activa el bit **DF (Don't Fragment - No Fragmentar)** en la cabecera.
2. **El descarte en el router:** Si un router intermedio recibe un paquete que supera la MTU de su siguiente enlace, al ver que tiene el bit `DF = 1`, tiene prohibido fragmentarlo.
3. **Notificación de error:** El router descarta el paquete y devuelve al host de origen un paquete de error de control **ICMP "Destination Unreachable"** (con el código de _fragmentación requerida pero bit DF activado_). Lo crucial es que este mensaje ICMP incluye el tamaño de la MTU de ese enlace restrictivo.
4. **Ajuste en el origen:** Al recibir el error ICMP, el host de origen reduce el tamaño de sus paquetes futuros hacia ese destino al tamaño de la MTU reportado e intenta el envío de nuevo.
5. **Iteración:** Si otro router más adelante tiene un límite aún menor, se repite el proceso hasta que los paquetes fluyen de extremo a extremo sin fragmentación intermedia.
# El enfoque estricto en IPv6:
En IPv6 se dio un paso más radical: **se eliminaron todos los campos de fragmentación de la cabecera fija**. **Los routers de IPv6 tienen prohibido fragmentar paquetes en tránsito**. Si un paquete IPv6 es demasiado grande para un enlace, el router obligatoriamente lo descarta y envía de vuelta un mensaje de error ICMPv6 "Packet Too Big" indicando la MTU. De este modo, IPv6 obliga a los hosts a realizar el descubrimiento de la MTU de la ruta de forma nativa. Si la fragmentación es absolutamente inevitable, solo el host emisor puede realizarla agregando una cabecera de extensión de fragmento especial antes de enviar el paquete.

>[!warning] Desventaja de PMTUD:
La única penalización de este mecanismo es el **retraso de inicio** (_startup delay_). Al principio de una conexión, puede tomar más de un viaje de ida y vuelta (RTT) simplemente sondear la ruta y ajustar el tamaño del paquete antes de que los datos reales empiecen a entregarse con éxito al destino.

[[2 SDN (Software Defined Networking)]]
