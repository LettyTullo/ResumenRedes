La versión 6 del Protocolo de Internet (**IPv6**) fue diseñada específicamente para resolver el inminente problema del agotamiento de direcciones de su predecesora, IPv4. Aprobada inicialmente en 1993 como una variación del protocolo propuesto **SIPP** (_Simple Internet Protocol Plus_), se consolidó como estándar de Internet en 1998. Aunque **no es compatible de forma directa con IPv4**, sí lo es con los demás protocolos auxiliares de la capa de transporte, de red y de aplicación (como TCP, UDP, ICMP, IGMP, OSPF, BGP y DNS) tras realizar pequeñas modificaciones.

>[!tip] Ventajas Clave de IPv6
>1. **Espacio masivo de direccionamiento:** Utiliza campos de dirección de 128 bits de longitud fija.
>2. **Simplificación de la cabecera fija:** Su formato se reduce a solo **7 campos obligatorios** (en comparación con los 13 de IPv4), lo que agiliza significativamente el procesamiento en los routers y reduce el retardo.
>3. **Soporte optimizado para opciones:** Los campos que antes eran obligatorios ahora se manejan mediante cabeceras de extensión que los routers pueden ignorar si no les competen.
>4. **Seguridad integrada nativamente:** Diseñada desde el inicio con soporte para autenticación y privacidad de datos.
>5. **Capacidades avanzadas de Calidad de Servicio (QoS):** Introduce el concepto de etiquetas de flujo para garantizar recursos en aplicaciones de tiempo real.

## Estructura de la Cabecera Fija (40 bytes)

![[Pasted image 20260830162547.png|458]]

>[!success] La cabecera obligatoria de IPv6 mide exactamente **40 bytes** y consta de los siguientes campos:
>- **Versión (4 bits):** Contiene el valor **6** para identificar de forma unívoca el protocolo.
>- **Servicios Diferenciados (8 bits):** Se comporta de la misma forma que en IPv4; utiliza los 6 bits superiores para clasificar el tráfico (QoS basada en clases) y los 2 bits inferiores para la Notificación Explícita de Congestión (ECN).
>- **Etiqueta de flujo (_Flow label_ - 20 bits):** Se utiliza para identificar grupos de paquetes que pertenecen a un flujo de datos continuo con requerimientos específicos de QoS (por ejemplo, transmisiones de video en tiempo real). Permite que los routers consulten rápidamente una tabla interna para decidir el trato prioritario que se le debe dar al paquete sin tener que procesar detalladamente su cabecera. Un flujo se designa mediante la combinación de la dirección de origen, dirección de destino y este identificador.
>- **Longitud de la carga útil (_Payload length_ - 16 bits):** Indica cuántos bytes siguen inmediatamente después de la cabecera fija de 40 bytes. A diferencia del campo _Total Length_ de IPv4, en IPv6 **los 40 bytes del encabezado obligatorio no se suman en este campo**, lo que permite cargas útiles de hasta 65,535 bytes.
>- **Siguiente encabezamiento (_Next header_ - 8 bits):** Reemplaza al campo de "Protocolo" de IPv4. Indica cuál es la cabecera de extensión opcional que sigue a la cabecera fija o, en caso de no haber extensiones, identifica el protocolo de la capa de transporte superior (como TCP o UDP) al que se debe entregar la carga útil.
>- **Límite de salto (_Hop limit_ - 8 bits):** Reemplaza al campo _Time to Live_ (TTL) de IPv4. Este valor se decrementa en 1 en cada router que atraviesa y sirve para descartar el paquete si llega a 0, evitando que los datagramas queden en bucles infinitos.
>- **Direcciones de origen y destino (16 bytes / 128 bits cada una):** Almacenan de forma fija las direcciones IP completas del emisor y el receptor final.
## Direccionamiento en IPv6

El tamaño de 128 bits genera un espacio de direcciones extraordinariamente grande (más de $3 x10^{38})$ direcciones disponibles).

- **Representación hexadecimal:** Las direcciones se escriben como **8 grupos de 4 dígitos hexadecimales** separados por dos puntos.
- **Abreviación de ceros:** Para simplificar la visualización de direcciones largas, los ceros consecutivos de uno o varios grupos se pueden omitir y reemplazar por un par de dos puntos (`::`) una única vez en toda la dirección. Por ejemplo, la dirección `8000:0000:0000:0000:0123:4567:89AB:CDEF` se representa de forma compacta como `8000::123:4567:89AB:CDEF`.
- **Direcciones IPv4 embebidas:** Durante la transición, se pueden representar direcciones IPv4 tradicionales usando dos puntos iniciales y la notación decimal con puntos (por ejemplo, `::192.31.20.46`).
## Campos Eliminados de IPv4 y Mejoras de Rendimiento

Para acelerar el procesamiento de paquetes, IPv6 eliminó varios campos de la cabecera IPv4:

- **Eliminación de la longitud de cabecera (_IHL_):** Debido a que la cabecera fija de IPv6 es siempre de 40 bytes, ya no es necesario un campo de longitud dinámica.
- **Eliminación de la Suma de Comprobación (_Checksum_):** Su cálculo dinámico en cada salto reducía drásticamente el rendimiento. Dado que los medios de transmisión actuales son altamente fiables y que las capas superiores (como TCP y UDP) y las inferiores (capa de enlace) ya efectúan sumas de comprobación, se consideró que no valía la pena pagar el costo de rendimiento de otra suma de comprobación adicional.
- **Eliminación de la fragmentación en ruta:** En IPv6, **los enrutadores intermedios tienen estrictamente prohibido fragmentar paquetes en tránsito**. El tamaño mínimo de paquete que todos los routers de la ruta deben admitir se aumentó a 1280 bytes.
- **Fragmentación basada en hosts:** Para evitar fragmentar en tránsito, los hosts aplican obligatoriamente el **descubrimiento de la MTU de la ruta (_Path MTU Discovery_)**. Si un paquete es demasiado grande para un enlace intermedio, el router correspondiente lo descarta y devuelve un mensaje de error ICMPv6 notificando el tamaño de MTU admitido, de modo que el host de origen reajuste o fragmente sus paquetes futuros hacia ese destino de forma directa.
## Cabeceras de Extensión (Opcionales)

Para mantener la eficiencia de la cabecera fija, las características que solo se requieren ocasionalmente se agregaron en cabeceras de extensión opcionales. Estas se colocan directamente después de la cabecera fija en el orden sugerido de transmisión. Existen 6 tipos principales:
![[Pasted image 20260830171232.png|494]]

1. **Opciones salto a salto (_Hop-by-hop options_):** Información miscelánea que deben examinar de forma obligatoria todos los routers de la ruta. Su aplicación principal es el soporte de **jumbogramas** (paquetes con cargas útiles que superan los 64 KB para aplicaciones de supercomputación), configurando el campo de longitud de carga útil fija a 0.
2. **Opciones de destino (_Destination options_):** Contiene campos que solo deben ser procesados e interpretados por el host de destino final.
3. **Enrutamiento (_Routing_):** Almacena una lista de enrutadores específicos que el paquete debe visitar en orden en su camino al destino (similar al enrutamiento suelto de origen de IPv4).
4. **Fragmentación (_Fragmentation_):** Contiene la información de desplazamiento, el identificador de datagrama y el bit de finalización necesarios para que el destino final reensamble el paquete si el host emisor tuvo que fragmentarlo debido al Path MTU.
5. **Autenticación (_Authentication_):** Proporciona un mecanismo de firma digital para verificar de forma segura la identidad del remitente e integridad del datagrama.
6. **Carga útil de seguridad cifrada (_Encrypted security payload_):** Permite encriptar el contenido del paquete de manera que solo el receptor autorizado pueda descifrarlo y leerlo.

## Protocolo ICMP (Internet Control Message Protocol)
El protocolo **ICMP es un subprotocolo fundamental de la capa de red que actúa como el mecanismo de **control y notificación de errores** para el Protocolo de Internet (IP).

A diferencia de IP, que se encarga del transporte de datos con el "mejor esfuerzo", ICMP permite que los routers y hosts informen sobre problemas encontrados durante el procesamiento de los paquetes.
# Funcionamiento y Encapsulamiento
Aunque opera en la capa de red (Capa 3), el protocolo ICMP no envía sus mensajes directamente sobre la capa de enlace. En su lugar:

- Un mensaje ICMP se entrega a la capa **IP**, la cual lo **encapsula** dentro de un datagrama IP estándar como si fueran datos normales antes de transmitirlo.
- Cada mensaje contiene una **cabecera** con tres campos fijos de 4 bytes en total: **Tipo** (8 bits), **Código** (8 bits) y **Suma de comprobación** (16 bits), seguidos de datos que dependen del tipo de mensaje.
![[Pasted image 20260516191305.png|573]]
# Principales Tipos de Mensajes ICMP

Existen diversos tipos de mensajes definidos para diferentes diagnósticos y situaciones de error:
1. **Destino inalcanzable (Destination Unreachable):** Se genera cuando un paquete no puede ser entregado. Puede ser enviado por un router que no localiza el destino o por el propio host de destino si el protocolo de usuario no está disponible. También ocurre si un router necesita fragmentar un paquete pero el bit **DF (Don't Fragment)** está activado.
2. **Tiempo excedido (Time Exceeded):** Se envía cuando un paquete es descartado porque su campo **TTL (Time to Live)** ha llegado a cero. Esto suele indicar que el paquete está atrapado en un bucle de enrutamiento o que el valor inicial del TTL era demasiado bajo.
3. **Redirección (Redirect):** Un router utiliza este mensaje para informar a un host que existe una **ruta mejor** o más corta para llegar a un destino específico, enseñándole así sobre la "geografía" de la red.
4. **Eco y Respuesta a Eco (Echo Request/Reply):** Se utilizan para comprobar si una máquina está activa y es alcanzable en la red.

# Herramientas Basadas en ICMP

ICMP es la base técnica de dos de las herramientas de diagnóstico más utilizadas en redes:

- **Comando "ping":** Utiliza los mensajes **Echo Request** y **Echo Reply**. El emisor envía un "eco" y el receptor está obligado a devolverlo, confirmando que la comunicación entre ambos es posible.
- **Comando "traceroute" (o tracert):** Utiliza de forma ingeniosa el mensaje de **Tiempo excedido**. Envía una serie de paquetes con el TTL empezando en 1 y aumentando de uno en uno en cada intento. Cada router en el camino descarta el paquete cuando el TTL llega a cero y devuelve un mensaje ICMP de error, permitiendo al emisor identificar cada salto en la ruta hasta el destino final.
## Protocolo ARP (Address Resolution Protocol)
El protocolo **ARP (Address Resolution Protocol)** es un mecanismo fundamental de la capa de red que se encarga de **mapear(resolucion) direcciones lógicas (IP) a direcciones físicas (MAC)** dentro de una red de área local.
# ¿Por qué es necesario el ARP?

Aunque las aplicaciones y el software de red utilizan direcciones IP para identificar los destinos, las tarjetas de red (NIC) de la capa de enlace de datos, como las de Ethernet, **no entienden las direcciones IP**. Las NIC solo pueden enviar y recibir tramas basándose en direcciones físicas de 48 bits (direcciones MAC). Por lo tanto, se necesita un "traductor" que convierta la dirección IP en una dirección MAC para que el paquete pueda viajar por el cable físico.
# Funcionamiento básico (Solicitud y Respuesta)
Cuando un host necesita enviar un paquete a una dirección IP específica dentro de su misma red:

1. **Solicitud ARP (Request):** El host emisor envía un paquete de **difusión (broadcast)** a toda la red preguntando: "¿Quién tiene la dirección IP X? Por favor, dime tu dirección MAC". Este mensaje incluye la IP y la MAC del emisor para que el destinatario sepa a quién responder.
2. **Respuesta ARP (Reply):** Todas las máquinas de la red reciben la pregunta, pero **solo aquella que posee la dirección IP especificada** responde con un mensaje enviando su dirección MAC.
3. **Encapsulamiento:** Una vez que el emisor recibe la MAC, puede encapsular el paquete IP en una trama Ethernet y enviarlo directamente al destino.

# La Tabla ARP (Caché)

Para evitar saturar la red con mensajes de difusión cada vez que se quiere enviar un dato, cada host mantiene una **Tabla ARP** en su memoria.

- Esta tabla relaciona temporalmente las direcciones IP con sus correspondientes direcciones MAC.
- Las entradas en esta tabla tienen un **tiempo de vida (TTL)** limitado (usualmente unos minutos) y expiran si no hay comunicación, obligando a realizar una nueva búsqueda ARP si se requiere contactar al equipo de nuevo.

# Casos especiales y variantes

- **Envío fuera de la red local:** Si el host destino no está en la misma subred, el emisor no realiza ARP por la IP del destino final. En su lugar, realiza ARP para obtener la dirección MAC de su **puerta de enlace predeterminada** (el router) y le envía el paquete a él para que lo encamine.
- **Proxy ARP:** Es una configuración donde un router responde a solicitudes ARP en nombre de hosts que se encuentran en otras redes, haciendo que el solicitante crea que el destino está en su propia red local.
- **ARP gratuito:** Ocurre cuando un host difunde su propia dirección IP al conectarse a la red. Sirve para actualizar las tablas ARP de otros hosts y para detectar si hay **direcciones IP duplicadas** en la red (si alguien responde, hay un conflicto).
## Protocolo DHCP 
El **DHCP** (Dynamic Host Configuration Protocol o Protocolo de Configuración Dinámica de Host) es un protocolo de la capa de red diseñado para la **asignación automática y dinámica de direcciones IP** y otros parámetros de configuración a los dispositivos que se conectan a una red.
# Propósito y necesidad

Aunque es posible configurar manualmente la dirección IP de cada ordenador, este proceso es tedioso y propenso a errores en redes grandes. El DHCP soluciona esto permitiendo que una red tenga un servidor encargado de gestionar un **conjunto de direcciones IP** y asignarlas a los equipos a medida que estos se activan, recuperándolas cuando dejan de usarse.

# Proceso de funcionamiento (4 pasos clave)

Cuando un dispositivo (cliente) se conecta a la red y no tiene una dirección IP, inicia un intercambio de mensajes con el servidor DHCP:

1. **DHCP DISCOVER:** El cliente envía un mensaje de difusión (**broadcast**) a toda la red para localizar servidores DHCP disponibles. En este mensaje, el cliente incluye su dirección física (**MAC**) para que el servidor pueda identificarlo.
2. **DHCP OFFER:** El servidor DHCP recibe la solicitud, selecciona una dirección IP libre de su lista y la envía al cliente en un mensaje de oferta.
3. **DHCP REQUEST:** Como puede haber más de un servidor DHCP en la red, el cliente envía este mensaje para aceptar formalmente una de las ofertas recibidas.
4. **DHCP ACK:** El servidor confirma la asignación con un acuse de recibo, indicando que la configuración está lista para ser usada.

_Nota: Si el servidor DHCP no está en la misma red local, los routers pueden configurarse para recibir estas difusiones y retransmitirlas hacia donde se encuentre el servidor._

[[5 MPLS y OSPF]]