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
6. **Tipo de carga útil / Payload Type (7 bits):** Especifica el algoritmo o formato de codificación utilizado para los datos multimedia (por ejemplo, audio comprimido, MP3, etc.). 
7. **Número de secuencia (16 bits):** Es un contador de 16 bits que se incrementa en `1` por cada paquete RTP transmitido. Permite al receptor **detectar paquetes perdidos** o reordenar aquellos que lleguen fuera de secuencia.
#### **Segunda y tercera palabras de 32 bits:**

8. **Marca de tiempo / Timestamp (32 bits):** Registra el instante exacto en que se tomó la primera muestra del paquete de datos. Sirve para que el receptor pueda **reproducir el contenido en el momento adecuado** y eliminar el efecto del _**jitter**_ (variación en el retardo de la red) mediante el uso de un búfer de reproducción.
9. **Identificador de la fuente de sincronización / SSRC (32 bits):** Es un número elegido de forma aleatoria que identifica la **fuente del flujo multimedia** (por ejemplo, el micrófono o la cámara de un participante en una conferencia). Esto evita que múltiples flujos enviados a una misma dirección IP y puerto se confundan entre sí.
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

[[6   6.5 Los protocolos de transporte]]