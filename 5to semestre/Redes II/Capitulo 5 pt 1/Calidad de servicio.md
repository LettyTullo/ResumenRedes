La **Calidad de Servicio (QoS)** se define como un conjunto de mecanismos y técnicas diseñados para proporcionar garantías de rendimiento superiores al modelo de "mejor esfuerzo" (_best effort_) convencional de Internet,. Su objetivo principal es satisfacer los requisitos específicos de diferentes aplicaciones para asegurar una experiencia de usuario óptima,.
# Parámetros clave de la QoS
Las necesidades de cada **flujo** (tráfico de un origen a un destino específico) se miden mediante cuatro parámetros fundamentales,:
- **Ancho de banda:** La capacidad de transmisión necesaria para el flujo,.
- **Retardo (Latencia):** El tiempo que tardan los paquetes en atravesar la red desde el origen al destino,.
- **Variación del retardo (****Jitter****):** La desviación estándar en los tiempos de llegada de los paquetes; es crítica para aplicaciones de audio y video,.
- **Pérdida de paquetes:** La tasa de paquetes que no llegan a su destino, lo cual puede degradar severamente el rendimiento,.
# Aspectos para su implementación
Para garantizar la calidad de servicio, la red debe abordar cuatro retos técnicos,:
1. **Requerimientos de la aplicación:** Identificar qué necesita cada tipo de tráfico (ej. la telefonía es sensible al retardo pero no requiere mucho ancho de banda),.
2. **Regulación del tráfico (Modelado):** Controlar la tasa promedio y las ráfagas de los flujos que entran a la red mediante algoritmos como la **cubeta con goteo** (_leaky bucket_) o la **cubeta con tokens**,,.
3. **Reserva de recursos:** Asignar ancho de banda, espacio de búfer y ciclos de CPU en los enrutadores para flujos específicos,.
4. **Control de admisión:** Decidir si la red puede aceptar de forma segura más tráfico sin comprometer las garantías ya existentes\
## Modelado de trafico 
El **modelado de tráfico** (o _traffic shaping_) es una técnica de la capa de red diseñada para regular la tasa promedio y la intensidad de las ráfagas de un flujo de datos que entra a la red. Su funcionamiento se basa en un acuerdo previo entre el cliente y el proveedor sobre el patrón de tráfico que se va a transmitir.
>[!info] Conceptos clave
>- **Acuerdo de Nivel de Servicio (SLA):** Es el contrato donde el usuario y la red acuerdan la "forma" o patrón del tráfico. El proveedor se compromete a entregar los paquetes a tiempo siempre que el cliente cumpla con este contrato.
>- **Vigilancia del Tráfico (Traffic Policing):** Es el proceso mediante el cual el proveedor supervisa el flujo para verificar que el cliente cumple con el SLA. Si el tráfico excede lo pactado, la red puede descartar los paquetes excedentes o reducir su prioridad.

# Algoritmos Principales
#### 1. Cubeta con Goteo (Leaky Bucket)
Este algoritmo fuerza una **tasa de salida constante**, independientemente de la velocidad a la que lleguen los datos desde el host.

>[!danger]- **Funcionamiento:**
> Imagine un cubo con un pequeño agujero en el fondo. El agua (paquetes) puede entrar a ráfagas, pero sale por el agujero a una tasa constante \(R\).
>- **Capacidad y Descarte:** El cubo tiene una capacidad física \(B\) (un búfer). Si el host envía una ráfaga que llena el cubo, cualquier dato adicional que llegue se "derrama" por los lados y se pierde (se descarta).
>- **Efecto:** Convierte un tráfico irregular o con ráfagas en un flujo de salida uniforme, eliminando completamente las ráfagas en la red.
#### 2. Cubeta con Tokens (Token Bucket)
A diferencia del anterior, este algoritmo permite un cierto grado de **tráfico en ráfagas** mientras mantiene una tasa promedio controlada.

>[!danger] Funcionamiento
>- **Generación de Tokens:** Un "grifo" llena el cubo con tokens (fichas) a una tasa constante de \(R\) tokens por segundo.
>- **Transmisión de Paquetes:** Para que un paquete pueda ser transmitido, debe "gastar" o sacar un token del cubo. Si el cubo tiene tokens acumulados, el host puede enviar una ráfaga de datos a la velocidad máxima del enlace hasta que los tokens se agoten.
>- **Espera:** Si el cubo está vacío, el paquete debe esperar en una cola hasta que se generen nuevos tokens.
>- **Capacidad de Ráfaga:** El tamaño del cubo \(B\) determina la ráfaga máxima que la red puede aceptar instantáneamente.

## Programacion de paquetes
 La **programación de paquetes** (o _packet scheduling_) se refiere a los algoritmos que utilizan los enrutadores para decidir qué paquete de su memoria intermedia (búfer) debe enviarse a continuación por una línea de salida. Su objetivo es distribuir de forma controlada los recursos del enrutador tanto entre los paquetes de un mismo flujo como entre diferentes flujos que compiten entre sí para garantizar la **Calidad de Servicio (QoS)**.
 
>[!important] Para ofrecer garantías de rendimiento, estos algoritmos gestionan la reserva de tres recursos críticos:
>1. **Ancho de banda:** Aseguran que no se asigne más tráfico a una línea de salida del que esta puede transportar.
>2. **Espacio de búfer:** Reservan memoria para absorber ráfagas de tráfico; si un flujo agota su búfer reservado, los paquetes excedentes se descartan.
>3. **Ciclos de CPU:** Garantizan que el procesador del enrutador tenga tiempo suficiente para procesar los paquetes de cada flujo.

# Algoritmos principales de programación
- **FIFO (First-In, First-Out) o FCFS (First-Come, First-Served):**
    - Es el método más sencillo; los paquetes se envían exactamente en el mismo orden en que llegaron.
    - **Desventaja:** Un flujo "agresivo" que envíe muchas ráfagas puede acaparar toda la capacidad, causando retardos o pérdida de paquetes en los demás flujos.
    - Cuando la cola está llena, el enrutador aplica el **descarte trasero** (_tail drop_), desechando cualquier paquete nuevo que llegue.
    
- **Encolamiento Justo (_Fair Queueing_):**
    - El enrutador mantiene colas separadas para cada flujo y las atiende de forma cíclica (**round-robin**).
    - Para evitar que los flujos con paquetes más grandes tengan ventaja, se utiliza una mejora que simula un servicio **byte por byte** basándose en "tiempos de finalización virtuales". Esto asegura que cada flujo reciba una fracción igual del ancho de banda.\
    
- **Encolamiento Justo Ponderado (_Weighted Fair Queueing - WFQ_):**
    - Es la variante más utilizada en arquitecturas de red modernas (como en los Servicios Diferenciados).
    - Permite asignar un **peso (\(W\))** a cada flujo para darle prioridad. Por ejemplo, a una cola de video se le puede dar más peso que a una de transferencia de archivos para que reciba más ancho de banda por cada ronda.
    - Utiliza la fórmula \(F_i = \max(A_i, F_{i-1}) + L_i/W\) para calcular el orden de salida, donde \(F\) es el tiempo de terminación, \(A\) la llegada y \(L\) la longitud del paquete.
## Control de admision
El **control de admisión** es una técnica utilizada en la capa de red para mantener a raya la **congestión** y garantizar la **Calidad de Servicio (QoS)**. Su principio fundamental es simple: la red **rechaza nuevas conexiones** o cargas si no tiene capacidad suficiente para transportarlas de forma segura sin comprometer el rendimiento de los flujos ya existentes.

Su funcionamiento se basa en:
- **Enfoque preventivo:** Se aplica en una escala de tiempo intermedia y es principalmente utilizado en redes de **circuitos virtuales**.
- **Decisión de entrada:** No se establece un nuevo circuito virtual a menos que la red confirme que puede soportar el tráfico adicional. Si la red está muy cargada, la solicitud de conexión falla, de forma similar a cuando un sistema telefónico no da tono de llamada por sobrecarga.
- **Combinación con enrutamiento:** Puede combinarse con el **enrutamiento consciente del tráfico** para buscar rutas alternativas que eviten los "puntos calientes" o nodos ya congestionados durante el proceso de configuración de la conexión.
![[Pasted image 20260816172639.png|374]]
## Servicios integrados
Los **Servicios Integrados** (frecuentemente abreviados como **IntServ**) son una arquitectura de red. Su objetivo principal es proporcionar garantías de **Calidad de Servicio (QoS)** específicas para cada flujo de datos individual, manejando tanto tráfico unicast como multicast.
# Modelo basado en flujos

A diferencia de otros modelos, IntServ se basa en la gestión de **flujos individuales**. Un flujo se define como un conjunto de paquetes que van desde un origen a un destino específico. En este esquema, cada aplicación (como una llamada de telefonía por Internet) obtiene sus propios recursos y garantías reservados de extremo a extremo.
# El protocolo RSVP (Resource reSerVation Protocol)

El mecanismo central para que los Servicios Integrados funcionen es el protocolo **RSVP**, el cual se encarga de realizar las reservas de recursos en la red. Su proceso operativo sigue estos pasos:

- **Solicitud del receptor:** El receptor del flujo envía un requerimiento de reserva hacia el emisor.
- **Reserva en cada salto:** Cada enrutador a lo largo de la ruta analiza el requerimiento y reserva los recursos necesarios (ancho de banda, espacio de búfer y ciclos de CPU).
- **Configuración completa:** Para que la reserva sea efectiva, debe establecerse con éxito en **todo el camino** entre emisor y receptor; de lo contrario, la reserva no se realiza.
- **Mantenimiento de estado:** Los routers deben mantener información sobre el estado de cada flujo activo en su memoria.
# Especificación del flujo
Para que la red pueda decidir si admite un nuevo flujo (control de admisión), el emisor debe proporcionar una **especificación del flujo** basada típicamente en los parámetros de una **cubeta con tokens** (tasa promedio y tamaño de ráfaga).
# Limitaciones y adopción

A pesar de su diseño robusto, las fuentes señalan que los Servicios Integrados tienen poca implementación práctica debido a varios inconvenientes:

- **Falta de escalabilidad:** Requerir que cada router mantenga el estado de cada flujo individual no escala bien cuando existen millones de flujos simultáneos en el núcleo de Internet.
- **Complejidad:** Los cambios necesarios en el código de los enrutadores y los intercambios de mensajes para configurar los flujos son muy complejos.
- **Vulnerabilidad:** Al mantener el estado internamente, el sistema es vulnerable si un enrutador se cae, ya que se pierde la información de las conexiones que pasaban por él.
## Servicios diferenciados
Los **Servicios Diferenciados** (frecuentemente abreviados como **DiffServ**) son una arquitectura de red (estandarizada en los RFC 2474 y 2475)

A diferencia de los Servicios Integrados (IntServ), que requieren reservas de recursos para cada conexión, DiffServ es mucho más simple de implementar y escalar, ya que las decisiones se toman de forma local en cada enrutador.
>[!important] Funcionamiento principal
>- **Basado en clases:** En lugar de gestionar cada llamada o flujo por separado, el tráfico se agrupa en categorías (como "voz", "video" o "datos normales"). Por ejemplo, en telefonía por Internet, todas las llamadas comparten los recursos reservados para la clase de voz.
>- **Campo de servicios diferenciados:** Se utiliza un campo específico en la cabecera de los paquetes **IPv4 e IPv6** para marcar la clase a la que pertenece el paquete. De los 8 bits disponibles, los 6 superiores se usan para la clase de servicio y los 2 inferiores para la notificación de congestión (ECN).
>- **Comportamiento por salto (PHB):** Los enrutadores aplican reglas de reenvío basadas únicamente en la marca del paquete que reciben, sin necesidad de conocer la ruta completa o configurar estados de flujo.
# Modelos de envío dentro de DiffServ
#### 1. Envío Expedito (Expedited Forwarding - EF)

Es la clase más sencilla y está diseñada para aplicaciones que requieren baja pérdida, bajo retardo y baja fluctuación (_jitter_), como la **VoIP**.

- Los paquetes marcados como "expeditos" se envían con preferencia absoluta sobre el tráfico regular.
- Simula un "tubo" dedicado donde el tráfico ve la red como si estuviera descargada.
#### 2. Envío Asegurado (Assured Forwarding - AF)

Es un esquema más elaborado (RFC 2597) que define niveles de prioridad y descarte.

- **12 Clases de servicio:** Se definen **cuatro clases de prioridad** (frecuentemente llamadas Oro, Plata, Bronce y Estándar) y **tres probabilidades de descarte** (baja, media y alta).
- **Mecanismo técnico:** Un clasificador identifica el tráfico; luego, un regulador (_policer_) marca los paquetes que exceden las ráfagas permitidas con una mayor probabilidad de descarte. Finalmente, el router utiliza algoritmos como **WFQ (Weighted Fair Queueing)** para asignar anchos de banda distintos a cada prioridad.
[[1 Internetworking]]
