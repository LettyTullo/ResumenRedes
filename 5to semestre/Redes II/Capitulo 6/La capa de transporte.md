La capa de transporte es el nivel encargado de realizar el **transporte de datos de extremo a extremo** (es decir, **de proceso a proceso**) desde una máquina de origen a una máquina de destino. Su función principal es tomar los datos de la capa de aplicación, dividirlos en unidades más pequeñas llamadas **segmentos** y entregarlos de manera eficiente al receptor.
#### La entidad de transporte

El trabajo físico y lógico de esta capa es gestionado por la **entidad de transporte**. Este componente de software o hardware puede residir en diversos puntos del sistema:

- En el núcleo (_kernel_) del sistema operativo.
- Como parte de una biblioteca de enlace de la aplicación de red.
- En un proceso de usuario independiente.
- Directamente en la tarjeta de interfaz de red (NIC).
#### ¿Por qué es importante? (La necesidad de su existencia)

A menudo surge la duda: si la capa de red ya se encarga de llevar paquetes de una máquina a otra, ¿por qué necesitamos una capa adicional de transporte? La respuesta es crucial y radica en la **propiedad del control**:
- **El dilema del control externo:** Los enrutadores físicos de la red son operados y administrados por terceros (como los ISPs). Si la capa de red pierde paquetes, se satura o se interrumpe bruscamente, los usuarios no tienen control sobre el hardware para solucionarlo.
- **Actúa como un escudo protector:** La capa de transporte se ejecuta completamente en las máquinas de los usuarios (hosts de origen y destino). Esto le permite actuar como un filtro que **aísla y protege a las aplicaciones de las imperfecciones e inestabilidades de la red subyacente**.
- **Aislamiento tecnológico:** Oculta las complejidades, variaciones y cambios de las tecnologías físicas de la red (como transitar entre Ethernet, Wi-Fi o WiMAX) detrás de un conjunto estandarizado de llamadas. Gracias a esto, un programador puede escribir una aplicación utilizando una API estándar (como los sockets de Berkeley) y tener la garantía de que funcionará de igual manera sobre cualquier red física sin reescribir el código.
# ¿Qué servicios ofrece la capa de transporte?

Para facilitar la programación de aplicaciones robustas, la capa de transporte proporciona una serie de servicios y abstracciones esenciales:
#### A. Tipos de Servicio: Orientado y No Orientado a la Conexión

- **Servicio Orientado a la Conexión (ej. TCP):** Está diseñado para modelar una "conexión perfecta". Oculta las pérdidas y retrasos de la red para ofrecer a la aplicación la abstracción de una **tubería de bits fiable al 100%**, donde los datos introducidos en un extremo salen exactamente en el mismo orden e íntegros en el otro extremo. Sigue tres fases estrictas: establecimiento, transferencia de datos y liberación de la conexión.
- **Servicio Sin Conexión (ej. UDP):** No realiza configuraciones previas. Simplemente encapsula los paquetes IP añadiendo una pequeña cabecera para entregar datagramas de manera rápida, útil en aplicaciones cliente-servidor sencillas (como búsquedas DNS) o transmisiones multimedia en tiempo real donde un pequeño porcentaje de pérdida de paquetes es tolerable frente al costo de retardo de las retransmisiones.
#### B. Direccionamiento (TSAPs / Puertos)

Dado que un ordenador suele poseer una única dirección de red (NSAP o dirección IP), la capa de transporte introduce los **TSAP (Transport Service Access Points)**, conocidos en la práctica como **números de puerto**. Estos puertos permiten multiplexar la red para que múltiples procesos y aplicaciones en ejecución dentro de la misma máquina puedan enviar y recibir tráfico simultáneamente de forma diferenciada.
#### C. Control de Errores y Confiabilidad Mejorada

A diferencia del control de errores de la capa de enlace (que solo protege un tramo físico de cable), la capa de transporte realiza una **comprobación de integridad de extremo a extremo** para proteger todo el camino recorrido. Si un paquete se corrompe dentro de la memoria de un router intermedio, las capas físicas no lo notarán, pero la capa de transporte detectará el fallo mediante sumas de comprobación (_checksum_) y solicitará la retransmisión (ARQ) de manera automática.
#### D. Control de Flujo y Buffers

La capa de transporte evita que un emisor rápido sature a un receptor lento que carece de memoria temporal para procesar los datos. Para ello, utiliza mecanismos de **ventanas deslizantes dinámicas**, donde el receptor avisa activamente al emisor de cuántos bytes tiene de espacio en sus búferes (_buffers_) para recibir más información antes de verse obligado a pausar la transmisión.
#### E. Control de Congestión

Dado que los routers intermedios de la red pueden congestionarse si reciben demasiados paquetes con excesiva rapidez, la capa de transporte asume la responsabilidad de **regular el ritmo al que inyecta datos a la red**. Analizando pérdidas implícitas o marcas de notificación de los routers (ECN), disminuye temporalmente su velocidad de envío para prevenir un colapso generalizado de la red.

## Primitivas de servicios
Las **primitivas de servicio de la capa de transporte** son las llamadas a procedimientos o funciones de software que permiten a los programas de aplicación (procesos de usuario) interactuar con la entidad de transporte para establecer comunicación a través de la red.

A diferencia de las primitivas de la capa de red, que modelan una transmisión con todas sus imperfecciones, las primitivas de transporte orientadas a la conexión tienen como objetivo fundamental **ocultar los errores de la red** y proporcionar a los programadores la abstracción de una **conexión perfecta, fiable y fácil de usar**.

#### 1. Primitivas de un Servicio de Transporte Simple
Este modelo teórico ilustra cómo interactúan un cliente y un servidor a través de 5 primitivas básicas:

|Primitiva|Paquete enviado|Significado|
|---|---|---|
|**LISTEN**|(Ninguno)|Bloquea el proceso hasta que algún host remoto intenta conectarse.|
|**CONNECT**|`CONNECTION REQ.`|Intenta activamente establecer una conexión con un destino.|
|**SEND**|`DATA`|Envía información (datos de usuario) a través de la conexión.|
|**RECEIVE**|(Ninguno)|Se bloquea a la espera de que llegue un paquete de datos.|
|**DISCONNECT**|`DISCONNECTION REQ.`|Solicita la liberación o cierre de la conexión.|

>[!example] Flujo de interacción Cliente-Servidor en este modelo:
>1. **Inicio del Servidor:** El servidor se ejecuta e invoca **LISTEN**, quedando en un estado suspendido esperando peticiones.
>2. **Solicitud del Cliente:** El cliente ejecuta **CONNECT**, lo que hace que su entidad de transporte envíe un segmento de solicitud de conexión (`CONNECTION REQUEST`).
>3. **Establecimiento:** La recepción del segmento despierta al servidor. Si este acepta, envía de vuelta un segmento de conexión aceptada (`CONNECTION ACCEPTED`), desbloqueando al cliente. La conexión queda formalmente establecida.
>4. **Transferencia de Datos:** Una vez conectados, ambos procesos pueden intercambiar datos de forma bidireccional invocando llamadas consecutivas a **SEND** y **RECEIVE**.
>5. **Liberación:** Cuando terminan, ejecutan **DISCONNECT** para liberar los recursos asignados a esa conexión en las tablas de las entidades de transporte

#### Primitivas Reales en Internet: Sockets de Berkeley
En la práctica, las aplicaciones de Internet que corren sobre TCP/IP no utilizan el modelo simple anterior, sino la **API de Sockets de Berkeley** (estandarizada para UNIX en 1983). Las primitivas que un programador utiliza para la gestión de conexiones TCP son las siguientes:

##### A. Primitivas para la Inicialización (Servidor y Cliente)
- **SOCKET:** Crea un nuevo punto terminal (endpoint) de comunicación. Reserva espacio en las tablas de la entidad de transporte y devuelve un descriptor de archivo (similar a abrir un archivo en disco). En este punto, el socket aún no tiene una dirección asociada.
- **BIND (Exclusivo del Servidor):** Asocia una dirección de red local (IP y número de puerto o TSAP) al socket creado. Esto permite que los clientes remotos sepan exactamente a dónde dirigir sus solicitudes de conexión. El cliente no suele requerir `BIND` porque su puerto local se asigna dinámicamente.
##### B. Primitivas para el Establecimiento de Conexiones
- **LISTEN (Servidor):** A diferencia de la llamada teórica, en los sockets **LISTEN** no es bloqueante. Su función es anunciar de forma pasiva que el servidor está dispuesto a recibir conexiones y define el tamaño máximo de la cola para almacenar solicitudes que lleguen simultáneamente.
- **ACCEPT (Servidor):** Esta sí es una llamada bloqueante. El servidor se suspende hasta que llega una solicitud de conexión activa. Cuando llega, la entidad de transporte **crea automáticamente un nuevo socket** con las mismas propiedades que el original y devuelve su descriptor. Esto permite al servidor bifurcar un hilo para atender al cliente en el nuevo socket y volver a escuchar en el socket original.
- **CONNECT (Cliente):** Bloquea activamente al proceso cliente e inicia el apretón de manos de tres vías (_three-way handshake_) de TCP enviando un segmento con el bit `SYN` activado. Se desbloquea únicamente cuando la conexión se establece con éxito.
##### C. Primitivas para el Intercambio de Datos y Cierre
- **SEND:** Envía flujo de bytes a través de la conexión full-duplex. (También se puede utilizar la llamada estándar del sistema UNIX `WRITE`).
- **RECEIVE:** Recibe los datos entrantes de la conexión. (Equivale a la llamada estándar del sistema `READ`).
- **CLOSE:** Libera la conexión de manera simétrica. TCP requiere que ambos extremos de la conexión ejecuten **CLOSE** de forma independiente (mediante el intercambio de segmentos con el bit `FIN`) para que la conexión se considere completamente cerrada y los recursos de memoria se liberen en los hosts.

>[!success] ¿Por qué es tan importante esta estructura?

La separación de estas primitivas garantiza el **aislamiento tecnológico**. Un desarrollador puede programar una aplicación compleja asumiendo que la red es un canal perfecto gracias a que las primitivas de la capa de transporte gestionan de forma totalmente invisible para el usuario detalles críticos como el control de errores (retransmisiones ARQ), control de flujo (búferes dinámicos) y el control de congestión de la red.