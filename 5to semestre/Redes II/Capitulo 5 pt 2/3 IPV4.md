El protocolo **IPv4 (Internet Protocol versión 4)** es el "pegamento" que mantiene unida a la Internet, encargándose de transportar paquetes de datos desde un origen a un destino de la manera más eficiente posible (**"mejor esfuerzo"**), sin importar cuántas redes intermedias deba atravesar.
##  El Formato del Datagrama IPv4

![[Pasted image 20260512150703.png|605]]
La cabecera tiene una **parte fija de 20 bytes** y una parte opcional de longitud variable.

>[!info] Explicacion de cada campo 
>- **Versión (4 bits):** Indica la versión del protocolo (en este caso, 4).
>- **IHL (4 bits):** Indica la longitud de la cabecera en palabras de **32 bits**. El valor mínimo es 5 (20 bytes) y el máximo 15 (60 bytes).
>- **Servicios Diferenciados (8 bits):** Se utiliza para distinguir la clase de servicio de los paquetes (QoS) y para señalar indicaciones de congestión.
>- **Longitud Total (16 bits):** Incluye la cabecera y los datos, con un máximo de **65,535 bytes**.
>- **Identificación (16 bits):** Permite al host de destino identificar a qué datagrama pertenecen los fragmentos recibidos.
>- **Flags (DF y MF):**
>	- **DF (Don't Fragment):** Ordena a los routers no fragmentar el paquete.
>	- **MF (More Fragments):** Indica si hay más fragmentos después del actual.
>- **Desplazamiento de Fragmento (13 bits):** Indica en qué parte del datagrama original se sitúa el fragmento.
>- **Tiempo de Vida (TTL) (8 bits):** Contador que se reduce en cada salto (router) para evitar que los paquetes circulen infinitamente; si llega a cero, el paquete se descarta.
>- **Protocolo (8 bits):** Indica a qué proceso de la capa de transporte (como **TCP** o **UDP**) debe entregarse el paquete en el destino.
>- **Suma de Comprobación (Checksum) (16 bits):** Verifica la integridad de la cabecera.
>- **Direcciones de Origen y Destino (32 bits cada una):** Indican las direcciones IP de las interfaces de red del emisor y receptor.
>- **Opciones (Variable):** Diseñado para permitir pruebas, seguridad o enrutamiento específico, aunque hoy en día muchos routers las ignoran.

## Direccionamiento IPv4
Una dirección IPv4 es una etiqueta lógica de **32 bits** que define la conexión de un dispositivo a una red (un router, un servidor u host).

- **Notación:** Se escribe comúnmente en formato **decimal con puntos**, dividiendo los 32 bits en 4 octetos (ej. `128.208.2.151`).
- **Jerarquía:** Cada dirección consta de un **Número de RED** y un **Número de HOST**. Esto permite a los routers encaminar basándose solo en el prefijo de red, reduciendo el tamaño de las tablas de rutas.
- **Prefijos y Máscaras:**
    - **CIDR(Enrutamiento Entre Dominios sin Clases):** Es el sistema estándar actual para la asignación de direcciones IP y el enrutamiento de paquetes en Internet. Usa una barra seguida de la longitud del prefijo (ej. `/24`).

>[!danger] ¿Qué es un Prefijo?
>Un **prefijo** es un bloque contiguo de espacio de direcciones IP donde todos los dispositivos comparten los mismos bits iniciales (la parte de red).
> **Notación CIDR:** Los bloques de direcciones se definen como `x.y.z.t /n`, donde `x.y.z.t` es una de las direcciones del bloque y **/n** indica la longitud del prefijo en bits. Por ejemplo, `/24` indica que los primeros 24 bits identifican a la red.

>[!example] Que es una Mascara?
> Dado que la longitud de la parte de red no se puede adivinar solo mirando la dirección IP, se utiliza la **máscara de subred** para indicar dónde termina la red y dónde empieza el host.
>- **Estructura binaria:** Es una secuencia de 32 bits que contiene **"1"** en todas las posiciones correspondientes a la parte de red (prefijo) y **"0"** en las posiciones de la parte de host.
>- **Relación visual:** Si un prefijo tiene una longitud **L**, su máscara tendrá exactamente **L** unos seguidos de **32–L** ceros.
>- **Conversión a decimal:** Las máscaras también se expresan en formato decimal con puntos. Por ejemplo, un prefijo **/20** (20 unos seguidos de 12 ceros) equivale a la máscara **255.255.240.0**
# Tipos de Direcciones y Reglas Especiales

Dentro de un bloque de direcciones, existen valores con funciones específicas:

- **Dirección de Red:** Es la dirección más baja del bloque (todos los bits de host en "0") y sirve para identificar el segmento de red; **no se puede asignar** a ningún host.
- **Dirección de Broadcast (Difusión):** Es la dirección más alta del bloque (todos los bits de host en "1") y se usa para enviar un paquete a **todos** los dispositivos de esa red simultáneamente.
>[!amarillo] Maximo numero de hosts en un bloque (red o subred)
>$$Host = 2^k- 2$$
k =( 32 - prefijo)
- **Direcciones Especiales:**
    - **0.0.0.0:** Utilizada por los hosts durante el proceso de arranque.
    - **127.x.x.x (Loopback):** Reservada para pruebas internas del propio host; los paquetes enviados aquí nunca salen al cable físico.
    - **Direcciones Privadas:** Rangos (como `10.0.0.0/8` hasta `10.255.255.255/8` , `172.16.0.0/12` hasta `172.31.255.255/12`o `192.168.0.0/16` hasta `192.168.255.255/16`) reservados para redes locales que no son visibles directamente en Internet y requieren **NAT** para navegar.
## Subnetting (Division en Subredes)
La **división en subredes** es una técnica que permite fragmentar un bloque de direcciones IP grande en varias redes más pequeñas e independientes para uso interno, mientras que para el resto de Internet siguen pareciendo una única red.

>[!info] La implementación de subredes responde a varias necesidades técnicas y administrativas:
>- **Reducción del tráfico de difusión (broadcast):** Este es el motivo principal; al dividir una LAN grande, se limita el alcance de los mensajes de difusión, mejorando el rendimiento general.
>- **Organización jerárquica:** Permite que la red refleje la estructura de la organización (por ejemplo, dividiendo un bloque en subredes para los departamentos de Informática, Ingeniería y Arte).
>- **Seguridad y gestión:** Facilita la administración interna y permite aislar el tráfico entre diferentes grupos de usuarios, dificultando ataques entre colectivos.
>- **Eficiencia en el direccionamiento:** Evita el desperdicio de direcciones IP al adaptar el tamaño de cada subred a la cantidad real de hosts que necesita.

# Funcionamiento técnico
Para dividir una red, se toman bits que originalmente pertenecían a la **porción de host** y se utilizan para identificar la **subred**.

- **Transparencia externa:** Desde fuera de la organización, la red se ve como un único prefijo (por ejemplo, un `/16`). Los routers externos no necesitan conocer la estructura interna, lo que ayuda a que las tablas de enrutamiento globales no colapsen.
- **Uso de máscaras de subred:** Los routers internos utilizan la **máscara de subred** para procesar los paquetes. Cuando llega un paquete, el router realiza una operación lógica **AND** entre la dirección IP de destino y la máscara de la subred para determinar a qué segmento enviarlo.
- **Flexibilidad:** La división no tiene que ser uniforme. Un bloque se puede partir en trozos de diferentes tamaños (por ejemplo, una mitad como `/17`, un cuarto como `/18`, etc.) según las necesidades.
## Que es CIDR?

**CIDR** son las siglas de **Classless Inter-Domain Routing** (Enrutamiento Entre Dominios sin Clases). Es el sistema estándar actual para la asignación de direcciones IP y el enrutamiento de paquetes en Internet.
# 1. Propósito: Agregación de Rutas
El objetivo fundamental de CIDR es frenar el crecimiento explosivo de las **tablas de enrutamiento** en los routers troncales de Internet.

- Para lograrlo, utiliza la **agregación de rutas**, que permite juntar múltiples prefijos IP pequeños en uno solo más grande (a veces llamado **superred**).
- De esta forma, un router distante solo necesita una entrada en su tabla para representar a miles de hosts o redes pequeñas.
# 2. Funcionamiento y Notación

A diferencia del direccionamiento por clases antiguo (donde los bloques eran fijos como Clase A, B o C), CIDR permite que los prefijos tengan cualquier longitud.
- **Notación:** Se escribe como una dirección IP seguida de una barra y el número de bits de la red (ejemplo: `194.24.0.0/21`).
- **Flexibilidad:** Permite que un bloque de direcciones se divida o combine de forma mucho más eficiente, adaptándose a las necesidades reales de cada organización.
# 3. Regla del Prefijo más Largo Coincidente

Debido a que con CIDR un destino puede estar contenido en varios prefijos de diferentes tamaños dentro de la misma tabla de enrutamiento, los routers aplican la regla del **prefijo más largo coincidente** (_longest matching prefix_).

- Esto significa que si un paquete coincide con una entrada de máscara `/20` y otra de `/24`, el router elegirá la entrada `/24` por ser la más específica para ese destino.
## Direccionamiento por clase 
El **direccionamiento por clase** (_classful addressing_) fue el diseño original para la asignación de direcciones IP que precedió al sistema actual (CIDR). En este esquema, el espacio de direcciones IP se dividía en cinco categorías fijas, identificadas por los bits de mayor orden de la dirección.

>[!info] Las Cinco Clases de Direcciones
>- **Clase A (Inicia con bit `0`):** El rango decimal es **0 a 127**. Utiliza los primeros **8 bits** para la red y los **24 restantes** para el host. Permitía pocas redes (128) pero con muchísimos hosts (16 millones cada una).
>- **Clase B (Inicia con bits `10`):** El rango decimal es **128 a 191**. Utiliza los primeros **16 bits** para la red y los **16 restantes** para el host. Diseñada para organizaciones de tamaño medio, permitía 65,536 hosts por red.
>- **Clase C (Inicia con bits `110`):** El rango decimal es **192 a 223**. Utiliza los primeros **24 bits** para la red y solo **8 bits** para el host. Admitía más de 2 millones de redes, pero cada una limitada a solo 256 hosts.
>- **Clase D (Inicia con bits `1110`):** Rango **224 a 239**. Reservada exclusivamente para el tráfico de **multidifusión** (_multicast_).
>- **Clase E (Inicia con bits `1111`):** Rango **240 a 255**. Reservada para **uso futuro** y experimentación.
## Diferencias entre CIDR y Direccionamiento con clases

# 1. Flexibilidad de los Bloques (Fijo vs. Variable)
- **Direccionamiento por clases:** El espacio se dividía en categorías fijas (**Clase A, B y C**) con máscaras predefinidas de 8, 16 o 24 bits. No se podía tener un bloque de un tamaño intermedio.
- **CIDR:** Los bloques pueden tener **cualquier longitud de prefijo** (notación `/n`), lo que permite que el tamaño de la red se ajuste exactamente a las necesidades de la organización, evitando el desperdicio masivo de direcciones.
# 2. Uso de Direcciones 
- **Direccionamiento por clases:** Sufría por ser muy rígido: la Clase A era demasiado grande (16 millones de hosts), la Clase C demasiado pequeña (256 hosts) y la Clase B era el "término medio" tan solicitado que se agotó rápidamente.
- **CIDR:** Resolvió esto permitiendo la creación de **subredes** (dividir bloques) y **superredes** (combinar bloques), optimizando el uso del escaso espacio de direcciones IPv4.

# 3. Mecánica de Reenvío en los Routers

- **Direccionamiento por clases:** El reenvío era muy simple; el router solo miraba los primeros bits de la dirección para saber automáticamente cuántos bits correspondían a la red y buscaba en una tabla fija.
- **CIDR:** El reenvío es más complejo porque ya no hay clases fijas. El router debe utilizar la regla del **prefijo más largo coincidente** (_longest matching prefix_), buscando en su tabla la entrada que tenga la máscara más específica para el destino.

# 4. Escalabilidad y Tablas de Enrutamiento

- **Direccionamiento por clases:** Las tablas de enrutamiento globales estaban "explotando" porque cada red individual necesitaba su propia entrada.
- **CIDR:** Introdujo la **agregación de rutas**. Esto permite que un router anuncie un solo prefijo grande (ej. un `/19`) que represente a miles de redes pequeñas internas, reduciendo drásticamente el tamaño de las tablas de rutas en el núcleo de Internet.
## NAT 
**NAT (Network Address Translation)**, o Traducción de Direcciones de Red, es una tecnología que permite el intercambio de direcciones IP y/o puertos en los paquetes que entran y salen de una red.

>[!danger]  Objetivo Principal
El propósito fundamental de NAT es **paliar la escasez de direcciones IPv4**. Dado que las direcciones de 32 bits son limitadas y se han agotado prácticamente en todo el mundo, NAT permite que una organización o un hogar utilice un bloque de **direcciones IP privadas** internamente y se conecte a Internet usando una sola (o unas pocas) **dirección IP pública**.

# Funcionamiento Básico
NAT opera generalmente en un dispositivo de interconexión (como un router o cortafuegos) situado en el límite entre la red interna del cliente y el ISP.

>[!example] Hay dos tipos de funcionamiento
> - **Hacia fuera:** Cuando un host interno envía un paquete a Internet, la caja NAT traduce su dirección de origen privada a la dirección IP pública compartida.
> - **Hacia dentro:** Cuando llega una respuesta desde Internet, el dispositivo NAT debe identificar a qué host de la red interna corresponde ese tráfico para realizar la traducción inversa.

Para que esto funcione internamente, se utilizan rangos de **direcciones privadas** que no son válidas en la Internet pública:
- `10.0.0.0` a `10.255.255.255`
- `172.16.0.0` a `172.31.255.255`
- `192.168.0.0` a `192.168.255.255`
# Tipos de NAT

>[!tip] Existen tres tipos de NAT
>- **NAT Estática:** Se realiza un mapeo fijo de uno a uno, donde una dirección IP privada siempre se traduce a una dirección IP pública específica.
>- **NAT Dinámica:** Se dispone de un conjunto (_pool_) de direcciones IP públicas. Cuando un host interno requiere conexión, el router le asigna dinámicamente una dirección pública que no esté siendo utilizada en ese momento.
>- **PAT (Port Address Translation) o NAT con sobrecarga:** Es el tipo más común (usado en hogares). Permite mapear múltiples direcciones privadas a través de una **única dirección pública** utilizando los números de **puerto TCP/UDP** para distinguir las conexiones. El router guarda una tabla que asocia la IP privada y el puerto de origen con la IP pública y un puerto asignado al azar.

# Limitaciones y Objeciones

A pesar de su utilidad, NAT presenta varias desventajas técnicas señaladas por los puristas de redes:

- **Viola el modelo arquitectónico de IP:** Rompe la premisa de que cada dirección IP identifica de forma única a una sola máquina en el mundo.
- **Rompe la conectividad extremo a extremo:** Dificulta que un host externo inicie una conexión con uno interno si este no ha enviado un paquete primero (problema para servidores domésticos o juegos).
- **Viola la estratificación de protocolos:** Un dispositivo de capa de red (Capa 3) debe "espiar" y modificar campos de la capa de transporte (Capa 4), como los puertos.
- **Problemas con aplicaciones específicas:** Algunas aplicaciones como **FTP** insertan direcciones IP dentro del cuerpo del mensaje, lo que requiere que NAT sea "parcheado" para cada aplicación que no sea transparente al protocolo.
- El internet no se exige el uso de TCP o UDP
- Restriccion del numero de asignaciones a una sola direccion IP
## Estructura de los routers
La estructura de un router es fundamentalmente **similar a la de una computadora personal**, diseñada específicamente para realizar tareas de interconexión de redes en la capa de red (Capa 3) del modelo OSI.
# Componentes de un router

>[!warning] Un router utiliza diferentes tipos de memoria para gestionar su funcionamiento y almacenamiento:
>- **Memoria de Trabajo o RAM:** Es volátil y se encarga de guardar la **tabla de enrutamiento**, la configuración actual en ejecución (_running configuration_), las colas de paquetes en espera y el sistema operativo mientras el equipo está encendido.
>- **NVRAM (Non-volatile RAM):** Almacena la **configuración de inicio** (_start-up configuration_), la cual se carga en la memoria RAM cada vez que el equipo arranca. Generalmente una memoria flash de poca capacidad.
>- **Memoria Flash:** Es una memoria no volátil de mayor capacidad que guarda la **imagen del Sistema Operativo (IOS)**. Puede retener varias versiones del software.
>- **Memoria ROM:** Contiene el programa de arranque o **bootstrap**, el software básico de inicio, pruebas de diagnóstico de hardware y rutinas de recuperación.

# Interfaces y Puertos
Los routers poseen diversos puntos de conexión física para interactuar con distintas tecnologías de red:

- **Interfaces de LAN:** Utilizadas para conectar el router a redes locales (típicamente puertos Ethernet).
- **Interfaces de WAN:** Permiten la conexión con redes de área amplia proporcionadas por los ISP (como enlaces seriales, fibra óptica o DSL).
- **Puertos de Administración:** Incluyen el **Puerto de Consola** (puerto serial no destinado al tráfico de red) y el puerto AUX para configurar el equipo de forma local o remota.
## Configuración de la línea serial

Para que dos dispositivos se comuniquen correctamente a través de una línea serial, ambos deben estar configurados con los mismos parámetros. Según las fuentes, los parámetros principales son:
>[!success] 
>1. **Tasa de datos (velocidad):** Se mide en bits por segundo (bps) o baudios. Los valores comunes son **4800, 9600 o 19200 bps**..
>2. **Bits por carácter:** Define cuántos bits se utilizan para representar un carácter ASCII, siendo habituales **5, 7 u 8 bits**.
>3. **Bits de parada:** Son bits adicionales que indican el final de la transmisión de un carácter. Pueden configurarse **1, 1.5 o 2 bits**
>4. **Paridad:** Un mecanismo básico de detección de errores. Las opciones son **Sin paridad (No), Par o Impar**.
>5. **Control de flujo:** Evita que el receptor se sature. Puede ser:
>- **Por Software:** Utiliza caracteres ASCII especiales como **XON** (carácter 17) y **XOFF** (carácter 19).
>- **Por Hardware:** Utiliza señales físicas como **RTS/CTS** (Request To Send / Clear To Send).

Esta configuración es la que se utiliza típicamente para acceder a la **consola de un router** o configurar **interfaces de WAN**.

## Transmisión asíncrona
La **transmisión asíncrona** (o serial) se caracteriza porque los datos se envían en pequeños grupos (generalmente caracteres) y el **intervalo de tiempo entre ellos es impredecible**.

>[!warning] Los aspectos clave de su funcionamiento son:
>- **Estructura del carácter:** Cada bloque de datos está rodeado por bits de control. Comienza con un **bit de inicio (start bit)**, seguido de los **bits de datos** (5 a 8), un **bit de paridad** opcional y finaliza con uno o más **bits de parada**.
>- **Sincronización:** A diferencia de la transmisión síncrona (donde se envía un flujo continuo de bits con un reloj constante como en SONET), en la asíncrona la línea puede permanecer en **estado inactivo (idle)** entre caracteres.
>- **Contexto ATM:** El término "asíncrono" también se aplica al protocolo **ATM (Asynchronous Transfer Mode)**, donde significa que las celdas de información solo se envían cuando hay datos reales que transportar, en lugar de ocupar el canal constantemente.
>- **Errores de temporización:** Un factor crítico es que el receptor debe muestrear la señal en momentos precisos; si hay una diferencia entre el reloj del emisor y el del receptor, pueden ocurrir errores en la interpretación de los bits.
## Tabla de enrutamiento
La **tabla de enrutamiento** es una base de datos interna o "mapa" que reside en la memoria RAM de un router y que contiene la información necesaria para decidir por qué interfaz de salida debe reenviar un paquete hacia su destino final.
# Función y proceso de reenvío

El router utiliza esta tabla cada vez que llega un paquete IP. El proceso, conocido como **reenvío (forwarding)**, consiste en extraer la dirección IP de destino del paquete y buscar en la tabla la línea de salida que corresponda a esa dirección.
# Estructura de una entrada de la tabla
Cada entrada de la tabla es típicamente un conjunto de datos que incluye:

- **Red de destino:** El prefijo o dirección de la red a la que se desea llegar.
- **Máscara de subred:** Define qué parte de la dirección es red y qué parte es host.
- **Interfaz:** El puerto físico del router (ej. GI0/0, Ethernet 1) por donde debe salir el paquete.
- **IP del siguiente salto (Next Hop):** La dirección IP del próximo router en el camino si el destino no está conectado directamente.
- **Métrica:** Un valor que indica el "costo" de la ruta (basado en saltos, retardo o ancho de banda). Si hay dos rutas al mismo destino, se elige la de menor métrica.
# Tipos de entradas en la tabla

Las rutas en la tabla pueden originarse de tres maneras:

1. **Directamente conectadas:** Se agregan automáticamente cuando se configura una interfaz del router y se activa. Son las redes a las que el router tiene acceso físico inmediato.
2. **Rutas estáticas:** Son introducidas manualmente por un administrador de red. Son útiles cuando el camino es obvio y no cambia, pero no se adaptan automáticamente a fallos en la red.
3. **Rutas dinámicas:** Se generan mediante **protocolos de enrutamiento** (como **OSPF, RIP, BGP o EIGRP**) que intercambian información con otros routers para aprender la topología de la red y calcular los caminos más cortos o eficientes de forma automática.
4. **Gateway: La puerta de enlace predeterminada (o _default gateway_) es la interfaz de un **router** que está conectada al mismo segmento de red o subred que un host. Su función principal es actuar como el punto de salida para que los dispositivos de una red local puedan comunicarse con destinos que se encuentran **fuera** de su propio segmento

[[4 IPV6]]
