Aborda la gestión de la carga en la red desde la perspectiva de la capa de transporte. Aunque la congestión ocurre físicamente en los enrutadores de la capa de red, es provocada por el volumen de datos que inyecta la capa de transporte. Por ello, el control de congestión es una **responsabilidad combinada** entre ambas capas, donde la única solución efectiva es que la entidad de transporte reduzca la velocidad a la que envía sus paquetes.
# Asignación deseable de ancho de banda (Sección 6.3.1)

El objetivo de un algoritmo de control de congestión no es solo evitar el colapso de la red, sino encontrar una asignación de ancho de banda para las entidades de transporte que cumpla con tres criterios fundamentales: **eficiencia**, **equidad** y **convergencia rápida**.

>[!info] Explicacion de cada uno:
>- **Eficiencia y Potencia:** - Una asignación eficiente del ancho de banda entre las entidades de transporte utilizará toda la capacidad disponible de la red. Sin embargo, no será la sumatoria de todas las tasas de bits máximas asignadas ya que el tráfico es en ráfagas, y normalmente no transmiten todos al máximo al mismo tiempo.
 **Caudal útil (goodput):** Es la tasa a la que se entregan paquetes útiles a la aplicación de destino.
**Comportamiento:** A medida que la carga ofrecida aumenta, el _goodput_ crece proporcionalmente al inicio. Sin embargo, cuando la carga se aproxima a la capacidad de la red, los búferes de los enrutadores se llenan y comienzan a descartarse paquetes.
**Colapso por congestión:** Si un protocolo de transporte retransmite agresivamente paquetes retardados (pensando erróneamente que se perdieron), la red entra en un estado donde los emisores transmiten a gran velocidad pero casi ningún trabajo útil se completa (_goodput_ cae en picado).
**Retardo:** Se mantiene casi constante al inicio (retardo de propagación) y se dispara exponencialmente a medida que la carga roza la capacidad máxima
 **Potencia:** obtendremos el mejor desempeño de la red si asignamos ancho de banda hasta el punto en que el retardo empieza a aumentar con rapidez. Este punto está por debajo de la capacidad. Para identificarlo, Kleinrock (1979) propuso la métrica de potencia, en donde: POTENCIA = CARGA / RETARDO. En un principio la potencia aumentará con la carga ofrecida, mientras el retardo permanezca en un valor pequeño y aproximadamente constante, pero llegará a un máximo y caerá a medida que el retardo aumente con rapidez. La carga con la potencia más alta representa una carga eficiente para que la entidad de transporte la coloque en la red.
 
>[!example] Equidad Máxima-Mínima (_Max-Min Fairness_):
>Es el criterio utilizado para dividir el ancho de banda entre flujos que compiten por los mismos enlaces. Una asignación es justa en sentido máximo-mínimo si **no es posible aumentar el ancho de banda de un flujo sin reducir el de otro flujo que tenga una asignación igual o menor**. En la práctica, este criterio garantiza que ninguna conexión se quede sin ancho de banda (_starvation_).
>**La regla dice que no se debe mejorar a una a costa de otra que está igual o peor.**
 Esto busca ayudar a los más desfavorecidos y si sobra capacidad se asigna equitativamente respetando la idea anterior.
 Se requiere un conocimiento global de la red
 Es más importante en la práctica que ninguna conexión se quede sin ancho de banda a que todas reciban la misma cantidad de ancho de banda.
 
>[!success] Convergencia:
> El algoritmo de control de congestión debe converger rápidamente hacia una asignación equitativa y eficiente del ancho de banda.
La demanda de red cambia de forma dinámica (las conexiones entran y salen constantemente).
Debido a la variación en la demanda, el punto de operación ideal para la red varía con el tiempo.
El algoritmo debe converger rápidamente hacia el punto de operación ideal (justo y eficiente) y rastrearlo sin oscilaciones inestables.
 Si el algoritmo no es estable, puede fracasar al tratar de converger hacia el punto correcto en algunos casos, o incluso puede oscilar alrededor del punto correcto.
# Regulación de la tasa de envío (Sección 6.3.2)

Para ajustar la velocidad de transmisión, la capa de transporte debe distinguir entre dos problemas que causan la pérdida de datos pero requieren soluciones opuestas que deben ser realizadas por el EMISOR:

1. **Control de flujo:** La velocidad se limita porque el **receptor es lento** y no tiene espacio en sus búferes.
2. **Control de congestión:** La velocidad se limita porque la **red intermedia es lenta** o está sobrecargada.
#### Señales de retroalimentación

#### **Tipos de Señales de Retroalimentación:**

1. **Explícita y precisa:** El enrutador indica explícitamente al emisor la velocidad exacta a la que debe transmitir (ejemplo: _XCP - eXplicit Congestion Protocol_).
2. **Explícita e imprecisa:** El enrutador activa bits en las cabeceras de los paquetes para avisar de la congestión, pero no indica en cuánto debe reducirse la tasa (ejemplo: **ECN** - _Explicit Congestion Notification_).
3. **Implícita y precisa:** El emisor mide variaciones continuas en el retardo de ida y vuelta (RTT) para anticipar la congestión antes de que ocurran pérdidas (ejemplo: _FAST TCP_, _BBR_).
4. **Implícita e imprecisa:** El emisor deduce la congestión únicamente mediante la **pérdida de paquetes** (ejemplo: _TCP Tahoe_, _TCP Reno_, _TCP CUBIC, TCP con routers que aplican RED_).
Cuando se proporcione una señal de congestión, los emisores deben reducir sus tasas. La forma en que se deben aumentar o reducir las tasas se proporciona mediante una ley de control. Puede aumentar la tasa en forma aditiva (sumando una cantidad fija) o multiplicativa (sumando un porcentaje) y puede reducirla también de manera aditiva o multiplicativa
#### La ley de control AIMD (_Additive Increase, Multiplicative Decrease_)

Chiu y Jain (1989) demostraron que ante señales de congestión binarias, la única ley de control que garantiza la **convergencia hacia una asignación equitativa y eficiente** es **AIMD**:

- **Incremento Aditivo (AI):** Cuando la red no está congestionada, los usuarios aumentan linealmente su tasa de envío a un ritmo suave (con una pendiente constante a 45° hacia la línea de eficiencia) para sondear el ancho de banda libre sin desestabilizar el sistema.
- **Decremento Multiplicativo (MD):** Al detectar congestión, los usuarios reducen drásticamente su tasa de envío en una proporción o porcentaje (apuntando al origen en el espacio de fase).
- Las alternativas como incrementar o reducir de forma puramente multiplicativa o aditiva (AIAD, MIMD, MIAD) no logran converger al punto óptimo de la red.
#### Aplicación en Internet y compatibilidad

TCP implementa una ley de control AIMD para ajustar la tasa de envío y proveer control de congestión. En vez de ajustar la tasa directamente, una estrategia de uso común en la práctica es ajustar el tamaño de una ventana deslizante. Si el tamaño de la ventana es W y el tiempo de ida y vuelta es RTT, la tasa equivalente es W/RTT. El tamaño de la ventana determina cuántos datos pueden estar en tránsito, y los ACK del receptor marcan el ritmo. Cuando los ACK dejan de llegar, TCP interpreta congestión y reduce el envío en un porcentaje.

- Conexiones con un tiempo menor de ida y vuelta tienden a crecer más rápido.

**Compatibilidad con TCP (** **TCP-friendly** **):** Como TCP es el protocolo predominante de control de congestión en Internet, cualquier nuevo protocolo de transporte debe comportarse de forma similar a AIMD para competir equitativamente por el ancho de banda y no acaparar los recursos frente a los flujos TCP existentes.
# Cuestiones inalámbricas (Sección 6.3.3)

Los protocolos de transporte como TCP que implementan control de congestión deben ser independientes de la red subyacente y de las tecnologías de capa de enlace. La cuestión principal es que la pérdida de paquetes se usa con frecuencia como señal de congestión. Las redes inalámbricas pierden paquetes todo el tiempo debido a errores de transmisión.

Los enlaces inalámbricos (como Wi-Fi 802.11) sufren con frecuencia pérdidas de tramas debido a **ruido, interferencias electromagnéticas o desvanecimiento de señal** (con tasas de pérdida comunes del 10% o más).

Si TCP opera sobre un enlace inalámbrico sin modificaciones, interpretará estas pérdidas físicas como congestión y aplicará el decremento multiplicativo, reduciendo drásticamente su velocidad de transmisión aun cuando la red no esté congestionada

La solución es que los dos mecanismos actúan en distintas escalas de tiempo. Las retransmisiones  
de la capa de enlace ocurren en el orden de microsegundos a milisegundos para los enlaces inalámbricos. Los temporizadores de pérdidas en los protocolos de transporte se activan en el orden de milisegundos a segundos. La diferencia es de tres órdenes de magnitud.

Esto permite a los enlaces inalámbricos detectar las pérdidas de tramas y retransmitirlas para reparar los errores de transmisión mucho antes de que la entidad de transporte deduzca la pérdida de paquetes.

La estrategia de enmascaramiento es suficiente para permitir que la mayoría de los protocolos de transporte operen bien a través de la mayoría de los enlaces inalámbricos. Sin embargo, no siempre es una solución adecuada. Algunos enlaces inalámbricos tienen tiempos de ida y vuelta largos, como los satélites. Para estos enlaces se deben usar otras técnicas para enmascarar la pérdida, como FEC (forward error correction).

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
- Un byte en cero y el número de protocolo (17 para UDP) y (6 para TCP).
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
Propuesta por Birrell y Nelson (1984), **RPC** es una técnica que permite a los programas llamar a procedimientos ubicados en máquinas remotas, haciendo que las interacciones de interacciones de red sean más fáciles de programar, todos los detalles de la conectividad pueden ocultarse al programador.
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

#### ¿Cómo se utiliza este campo de marca de tiempo junto con RTCP para mantener sincronizados los flujos independientes de audio y vídeo?
##### 1. El reto técnico: ¿Por qué se desincronizan?
En una transmisión multimedia (como una llamada o un video en vivo), el audio y el vídeo se capturan, codifican y transmiten como **flujos RTP independientes**, cada uno viajando en sus propios paquetes con su propio identificador de fuente (**SSRC**).
Esto plantea dos problemas principales:
- **Relojes físicos diferentes:** La tarjeta de sonido y la cámara de vídeo utilizan relojes de hardware distintos que funcionan a frecuencias diferentes (por ejemplo, el audio puede tomar muestras a 8 kHz o 44.1 kHz, mientras que el vídeo trabaja con un contador de 90 kHz o con frecuencias de fotogramas como 25/30/60 fps) y sufren pequeñas desviaciones (_clock drift_).
- **Jitter de la red:** Los paquetes de cada flujo viajan por separado a través de la red y sufren variaciones de retardo variables (_jitter_), haciendo que los paquetes de vídeo y audio lleguen en momentos desincronizados al receptor.
##### 2. Primer nivel: La Marca de Tiempo en RTP (_Intra-stream_)
Cada paquete RTP lleva en su cabecera un campo de **Marca de tiempo de 32 bits**.
- **Función:** Registra el instante exacto en que se tomó la **primera muestra de datos** contenida en ese paquete.
- **Carácter relativo:** Esta marca de tiempo es **relativa** al inicio del propio flujo dentro de la escala de tiempo de su reloj local; no refleja una hora del día ni un tiempo real absoluto.
- **Uso:** Le permite al receptor ordenar los paquetes dentro del _mismo flujo_ de audio o vídeo y colocarlos en un **búfer de reproducción** (_playout buffer_) para suavizar el _jitter_.
##### 3. El límite: ¿Por qué las marcas de tiempo RTP solas no bastan?
Debido a que cada flujo (audio y vídeo) inicia con un número de secuencia y una marca de tiempo inicial aleatorios, y utiliza frecuencias de muestreo distintas, **es imposible comparar directamente el número de marca de tiempo RTP de un paquete de audio con el de uno de vídeo**. Un valor de marca de tiempo `24000` en audio no corresponde al mismo instante físico que `24000` en vídeo.
##### 4. Segundo nivel: RTCP y la sincronización inter-flujo (_Inter-stream_)
Para resolver esta incompatibilidad y lograr que los labios de la persona en pantalla coincidan con su voz, entra en juego el protocolo hermano **RTCP**.
1. **Informes del Emisor (RTCP Sender Reports):** De manera periódica, la fuente emisora transmite paquetes de control RTCP para cada flujo activo.
2. **Emparejamiento de relojes (NTP + RTP):** Cada informe de emisor en RTCP incluye una **pareja de marcas de tiempo vinculadas**:
    - La **hora real absoluta** (tiempo de pared o _wall-clock time_, utilizando el formato de reloj global NTP de 64 bits).
    - La **marca de tiempo RTP correspondiente** en ese exacto instante para ese flujo en particular.
3. **Mapeo a un tiempo de referencia común:** Al recibir los informes RTCP de ambos flujos (audio y vídeo), la aplicación receptora relaciona la marca de tiempo de RTP de cada flujo con la misma escala de tiempo real global (NTP).
##### 5. Resultado final en el Búfer de Reproducción
Con la relación matemática establecida por RTCP entre los contadores locales de RTP y el tiempo real común:
- El receptor toma los paquetes de audio y vídeo que descansan en sus respectivos búferes de almacenamiento.
- Calcula exactamente en qué milisegundo de tiempo real debe salir cada muestra de audio y cada fotograma de vídeo.
- Extrae y reproduce de forma simultánea el fotograma de vídeo y la muestra de audio que comparten el mismo instante de tiempo real de origen, logrando una **reproducción fluida y perfectamente sincronizada**.

[[5 6.5 Los protocolos de transporte]]