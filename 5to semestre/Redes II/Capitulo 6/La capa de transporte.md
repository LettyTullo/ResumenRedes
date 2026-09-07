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
