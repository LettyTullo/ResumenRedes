Aborda la gestión de la carga en la red desde la perspectiva de la capa de transporte. Aunque la congestión ocurre físicamente en los enrutadores de la capa de red, es provocada por el volumen de datos que inyecta la capa de transporte. Por ello, el control de congestión es una **responsabilidad combinada** entre ambas capas, donde la única solución efectiva es que la entidad de transporte reduzca la velocidad a la que envía sus paquetes.
# Asignación deseable de ancho de banda (Sección 6.3.1)

El objetivo de un algoritmo de control de congestión no es solo evitar el colapso de la red, sino encontrar una asignación de ancho de banda para las entidades de transporte que cumpla con tres criterios fundamentales: **eficiencia**, **equidad** y **convergencia rápida**.

>[!info] Explicacion de cada uno:
>- **Eficiencia y Potencia:** Busca maximizar la tasa de entrega útil de paquetes (_goodput_) manteniendo el retardo bajo. Si la carga ofrecida se aproxima demasiado a la capacidad máxima de la red, las ráfagas de tráfico llenan los búferes de los routers y provocan pérdidas de paquetes y demoras exponenciales. Esto puede derivar en un **colapso de congestión**, donde los emisores retransmiten continuamente datos sin lograr trabajo útil.
>- **Equidad Máxima-Mínima (_Max-Min Fairness_):** Es el criterio utilizado para dividir el ancho de banda entre flujos que compiten por los mismos enlaces. Una asignación es justa en sentido máximo-mínimo si **no es posible aumentar el ancho de banda de un flujo sin reducir el de otro flujo que tenga una asignación igual o menor**. En la práctica, este criterio garantiza que ninguna conexión se quede sin ancho de banda (_starvation_).
>- **Convergencia:** Debido a que las conexiones se abren, se cierran y cambian su demanda en el tiempo, la asignación ideal es dinámica. El algoritmo de control debe **converger rápidamente** al punto de operación óptimo y rastrearlo de forma estable sin oscilar drásticamente alrededor de él.
# Regulación de la tasa de envío (Sección 6.3.2)

Para ajustar la velocidad de transmisión, la capa de transporte debe distinguir entre dos problemas que causan la pérdida de datos pero requieren soluciones opuestas:

1. **Control de flujo:** La velocidad se limita porque el **receptor es lento** y no tiene espacio en sus búferes.
2. **Control de congestión:** La velocidad se limita porque la **red intermedia es lenta** o está sobrecargada.
#### Señales de retroalimentación

Para saber cuándo frenar, la capa de transporte recopila señales de la red. Estas señales pueden ser explícitas (como bits **ECN** marcados por los routers en las cabeceras o mensajes de estrangulamiento) o implícitas (medición del aumento en el tiempo de ida y vuelta **RTT** o la detección de **pérdida de paquetes**).
#### La ley de control AIMD (_Additive Increase, Multiplicative Decrease_)

Chiu y Jain (1989) demostraron que ante señales de congestión binarias, la única ley de control que garantiza la **convergencia hacia una asignación equitativa y eficiente** es **AIMD**:

- **Incremento Aditivo (AI):** Cuando la red no está congestionada, los usuarios aumentan linealmente su tasa de envío a un ritmo suave (con una pendiente constante a 45° hacia la línea de eficiencia) para sondear el ancho de banda libre sin desestabilizar el sistema.
- **Decremento Multiplicativo (MD):** Al detectar congestión, los usuarios reducen drásticamente su tasa de envío en una proporción o porcentaje (apuntando al origen en el espacio de fase).
- Las alternativas como incrementar o reducir de forma puramente multiplicativa o aditiva (AIAD, MIMD, MIAD) no logran converger al punto óptimo de la red.
#### Aplicación en Internet y compatibilidad

En Internet, el protocolo **TCP implementa AIMD de manera indirecta** ajustando el tamaño de una **ventana de congestión (\(cwnd\))** mediante un reloj impulsado por los acuses de recibo (ACKs). El mecanismo presenta un sesgo natural: las conexiones con un RTT más corto reciben confirmaciones más rápido y hacen crecer su ventana con mayor velocidad que las conexiones lejanas. Debido a que TCP domina el tráfico en la red, se exige que cualquier nuevo protocolo de transporte sea **compatible con TCP (_TCP-friendly_)** para evitar que acapare el ancho de banda al competir con flujos TCP estándar.
# Cuestiones inalámbricas (Sección 6.3.3)

Los protocolos de control de congestión de la capa de transporte asumen tradicionalmente que cualquier pérdida de paquetes es el síntoma de un router saturado. Sin embargo, en los **enlaces inalámbricos**, la mayoría de las pérdidas se deben a **errores de transmisión de radio o ruidos en el canal** y no a congestión en la red.

Si un protocolo como TCP confunde un error de canal inalámbrico con congestión, reduce erróneamente su ventana de envío, lo que provoca una **degradación severa e innecesaria del rendimiento** en redes móviles o Wi-Fi.

## Seccion 6.4 UDP 
#  UDP: User Datagram Protocol (RFC 768)

UDP es un protocolo de transporte **sin conexión, no confiable y de funcionalidad mínima**. Su objetivo principal no es garantizar la entrega, sino proporcionar una interfaz liviana sobre el protocolo IP agregando la capacidad de **multiplexar y demultiplexar múltiples procesos** mediante el uso de puertos.
#### A. Estructura del Encabezado UDP (8 bytes)

Un segmento UDP consta de una **cabecera fija de 8 bytes** seguida de la carga útil de datos. El encabezado se divide en 4 campos de 16 bits (2 bytes cada uno):
![[Pasted image 20260924114144.png]]

1. **Puerto de Origen (16 bits):** Identifica el proceso o aplicación que envía el paquete. Se utiliza principalmente cuando el receptor necesita devolver una respuesta.
2. **Puerto de Destino (16 bits):** Identifica el proceso o aplicación receptora en la máquina de destino.
3. **Longitud UDP (16 bits):** Indica la longitud total del segmento en bytes, incluyendo los 8 bytes de cabecera y los datos. La longitud mínima es de 8 bytes y el máximo teórico es de 65.515 bytes (limitado por el tamaño máximo del paquete IP).
4. **Suma de Comprobación / Checksum (16 bits):** Campo opcional para verificar la integridad del segmento. Si no se calcula, se almacena como cero. Si detecta un error, el paquete se descarta sin enviar notificaciones.

#### B. La Pseudocabecera IP

Para calcular la suma de comprobación de manera más estricta, UDP incluye conceptualmente una **pseudocabecera IPv4** antes de los datos. Contiene:
![[Pasted image 20260924114209.png|637]]

- Dirección IP de origen (32 bits).
- Dirección IP de destino (32 bits).
- Un byte en cero y el número de protocolo (17 para UDP).
- La longitud del segmento UDP.

_Nota de diseño:_ Incluir la pseudocabecera permite detectar paquetes mal enrutados o entregados por error a una máquina equivocada, pero representa una **violación de la jerarquía de capas**, ya que la capa de transporte inspecciona campos pertenecientes a la capa de red (IP).

#### C. Lo que UDP NO hace

UDP transfiere toda la responsabilidad del control a las aplicaciones superiores:

- **Sin control de flujo:** No limita la velocidad de envío hacia el receptor.
- **Sin control de congestión:** No reduce la velocidad ante la saturación de los routers de la red.
- **Sin retransmisión ni ordenamiento:** No reenvía segmentos perdidos/defectuosos ni garantiza que lleguen en el orden en que se enviaron.
#### D. Casos de uso ideales
UDP se utiliza cuando la velocidad y la baja latencia son preferibles a la fiabilidad absoluta:
- **Aplicaciones Cliente-Servidor breves:** Como **DNS (Domain Name System)**, donde el cliente envía una consulta rápida de 1 paquete y espera 1 respuesta; si falla, simplemente vence un temporizador y reintenta.
- **Transmisión multimedia y juegos en línea:** Donde la pérdida de un paquete ocasional es preferible a sufrir la latencia de una retransmisión.
#### Ventajas y desventajas
**Ventajas**
- Simplicidad
- Velocidad
- Baja Sobrecarga 
**Desventajas**
- Falta de fiabilidad
- Ausencia de control de flujo y congestion
# Llamada a Procedimiento Remoto (RPC - Remote Procedure Call)
Propuesta por Birrell y Nelson (1984), **RPC** es una técnica que permite a un programa llamar a procedimientos o funciones ubicadas en máquinas remotas como si fueran llamadas locales ordinarias, ocultando los detalles del paso de mensajes de red.
#### A. El mecanismo de los _Stubs_ (Talones)
Para lograr la ilusión de una llamada local, RPC utiliza dos componentes cliente-servidor:

- **Stub del Cliente:** Procedimiento de biblioteca en la máquina cliente que sustituye al procedimiento remoto.
- **Stub del Servidor:** Procedimiento en la máquina remota que recibe el mensaje de red y llama al procedimiento real.

#### B. Pasos para ejecutar una RPC
![[Pasted image 20260924114636.png|527]]
1. El programa cliente hace una llamada local estándar al _stub_ del cliente.
2. El _stub_ del cliente empaqueta los parámetros en un formato neutro para la red (proceso denominado _**marshaling**_ o serialización) y realiza una llamada al sistema operativo.
3. El SO del cliente envía el mensaje por la red hacia el servidor.
4. El SO del servidor entrega el paquete entrante al _stub_ del servidor.
5. El _stub_ del servidor desempaqueta los parámetros (_unmarshaling_) y llama al procedimiento real en la CPU del servidor. La respuesta sigue la ruta inversa.
#### Ventajas de RPC
- Simplificación del desarrollo al abstraer la complejidad de complejidad de la red.
- Eficiencia en la comunicación entre sistemas distribuidos.
- Flexibilidad para trabajar en la nube y con microservicios.
#### Desafíos y limitaciones de RPC

Aunque es un modelo elegante, presenta complicaciones técnicas en la práctica:

- **Parámetros Puntero:** No se pueden pasar direcciones de memoria directamente porque el cliente y el servidor tienen espacios de direcciones virtuales distintos. A veces se simula usando "copia y restauración", pero falla con estructuras de datos complejas como grafos.
- **Tipado Débil (ej. C):** Dificultad para determinar el tamaño de un arreglo sin un parámetro explícito para poder marshalizarlo.
- **Variables Globales:** No se comparten entre máquinas distintas.
- **Operaciones no Idempotentes:** Una operación es _idempotente_ si se puede repetir varias veces sin causar efectos secundarios no deseados (ej. consultar el DNS). Si la operación no es idempotente (ej. realizar un pago o incrementar un contador), la pérdida de un ACK o mensaje puede provocar ejecuciones duplicadas peligrosas si se retransmite a ciegas por UDP.
# Protocolos de Transporte en Tiempo Real: RTP y RTCP

Para evitar que cada aplicación de streaming o telefonía reinvente su propio sistema sobre UDP, el IETF estandarizó **RTP** y **RTCP** (RFC 3550).

#### A. RTP (Real-time Transport Protocol)

RTP se ejecuta normalmente en el **espacio de usuario sobre UDP**. Su función principal es multiplexar varios flujos multimedia (audio, vídeo, texto) dentro de un único flujo de paquetes UDP.

- **No ofrece garantías:** No reserva ancho de banda ni garantiza entregas en tiempo real por sí solo; depende de las capacidades de la red.
- **Numeración de Secuencia:** Permite al receptor detectar si se han perdido paquetes o si llegaron fuera de orden. Si se pierde un paquete, no se retransmite (llegaría demasiado tarde), sino que la aplicación decide saltar un fotograma o interpolar el audio.
- **Marcas de Tiempo (Timestamps):** Registran el momento en que se tomó la primera muestra del paquete. Permiten al receptor desacoplar el momento de reproducción del momento de llegada del paquete, reduciendo el efecto del _**jitter**_ (variación del retardo).

##### Cabecera RTP
El diseño del encabezado se organiza en palabras de **32 bits** de la siguiente manera:
![[Pasted image 20260924115134.png|504]]
#### **Primera palabra de 32 bits:**

1. **Versión / Ver (2 bits):** Identifica la versión del protocolo RTP utilizada (la versión estándar actual es la 2).
2. **Relleno / P - Padding (1 bit):** Si se establece en `1`, indica que el paquete contiene bytes de relleno adicionales al final para ajustar el tamaño a un múltiplo de 4 bytes (32 bits). El último byte de relleno indica la cantidad exacta de bytes añadidos.
3. **Extensión / X (1 bit):** Si se activa en `1`, señala la presencia de una cabecera de extensión personalizada entre la cabecera fija y la carga útil de datos.
4. **Contador de contribuyentes / CC (4 bits):** Indica cuántos identificadores de fuentes colaboradoras (CSRC) siguen a la cabecera principal (de 0 a 15 identificadores).
5. **Marcador / M (1 bit):** Es un bit de interpretación específica para la aplicación. Se utiliza para señalar límites o eventos significativos en el flujo de medios; por ejemplo, el inicio de un fotograma de vídeo o el comienzo de un tramo de voz (_talkspurt_) tras un silencio en un canal de audio.
6. **Tipo de carga útil / Payload Type (7 bits):** Especifica el algoritmo o formato de codificación utilizado para los datos multimedia (por ejemplo, audio comprimido, MP3, etc.). Como cada paquete lleva este campo, la aplicación puede **cambiar el tipo de codificación dinámicamente** en mitad de una transmisión si la red se congestiona.
7. **Número de secuencia (16 bits):** Es un contador de 16 bits que se incrementa en `1` por cada paquete RTP transmitido. Permite al receptor **detectar paquetes perdidos** o reordenar aquellos que lleguen fuera de secuencia.
#### **Segunda y tercera palabras de 32 bits:**

8. **Marca de tiempo / Timestamp (32 bits):** Registra el instante exacto en que se tomó la primera muestra del paquete de datos. Sirve para que el receptor pueda **reproducir el contenido en el momento adecuado** y eliminar el efecto del _**jitter**_ (variación en el retardo de la red) mediante el uso de un búfer de reproducción.
9. **Identificador de la fuente de sincronización / SSRC (32 bits):** Es un número elegido de forma aleatoria que identifica unívocamente la **fuente del flujo multimedia** (por ejemplo, el micrófono o la cámara de un participante en una conferencia). Esto evita que múltiples flujos enviados a una misma dirección IP y puerto se confundan entre sí.
#### **Campos de tamaño variable (Opcionales):**

10. **Identificadores de fuentes colaboradoras / CSRC (0 a 15 palabras de 32 bits):** Se utiliza cuando en la sesión hay un **mezclador**. En una multiconferencia donde un mezclador combina las señales de audio de varios participantes en un único flujo, el mezclador se convierte en la fuente de sincronización (SSRC) e inserta en este campo la lista de los identificadores SSRC originales de cada uno de los participantes que contribuyeron a ese paquete.

#### B. RTCP (Real-time Transport Control Protocol)

Es el protocolo hermano de RTP. No transporta muestras de medios, sino que se encarga de:

1. **Retroalimentación de la calidad:** Informa sobre el retardo, la pérdida de paquetes, el _jitter_ y la congestión para que los códecs adapten su tasa de bits.
2. **Sincronización inter-flujo:** Mantiene sincronizados los flujos independientes de audio y vídeo cuando usan relojes distintos.
3. **Identificación:** Asocia nombres de usuario ASCII a las fuentes para mostrar en pantalla quién está hablando.
#### C. Control de Jitter y Búfer en el Receptor

Debido a que la red introduce demoras variables (_jitter_), el receptor almacena los paquetes en un **búfer de reproducción**.

- Un búfer más grande elimina las pausas o brechas en la reproducción pero incrementa el retardo (latencia).
- Las aplicaciones en vivo (videoconferencias) requieren búferes pequeños para mantener baja la latencia, mientras que las de _streaming_ bajo demanda pueden usar búferes grandes para máxima fluidez.
