Los servicios de esta capa se implementan mediante **protocolos de transporte** que se ejecutan entre las entidades de transporte de los sistemas finales. Aunque comparten la necesidad de gestionar el control de errores, el orden de los datos y el control de flujo con la capa de enlace (Capa 2), el entorno de transporte es infinitamente más complejo porque abarca la red completa en lugar de un único enlace directo.
#### A. Direccionamiento (TSAP y NSAP)
En un enlace directo de capa de enlace, el destino del paquete es obvio. En cambio, en la capa de transporte se requiere **direccionamiento explícito**.
- **NSAP (Network Service Access Point):** Es la dirección del host en la capa de red (por ejemplo, la dirección IP).
- **TSAP (Transport Service Access Point):** Es el punto final específico dentro del host donde escucha un proceso (los conocidos **puertos** TCP/UDP). Los TSAPs permiten multiplexar la red para que múltiples aplicaciones independientes compartan de manera simultánea una única dirección IP.
##### El Proceso de Envío Paso a Paso (Anidamiento de Cabeceras)
El viaje de un mensaje desde el origen hasta el destino se basa en el **anidamiento sucesivo de cabeceras** en el emisor y su posterior extracción en el receptor:
![[Pasted image 20260908094250.png|399]]

>[!info] **En el Host de Origen (Host 1):**
>1. **Paso de Aplicación a Transporte:** Un proceso de aplicación (por ejemplo, un cliente de correo conectado al TSAP local de origen 1208) genera datos y los pasa a la **entidad de transporte**. Para ello, especifica el **TSAP de destino** (el puerto del servidor de correo, ej. 1522) y el **NSAP de destino** (la dirección IP del Host 2).
>2. **Creación del Segmento:** La entidad de transporte toma el mensaje, le añade una cabecera de transporte que contiene los campos de **TSAP de origen** y **TSAP de destino**, construyendo así un **segmento**.
>3. **Paso de Transporte a Red:** La capa de transporte entrega el segmento a la **capa de red** indicando el NSAP (IP) de destino.
>4. **Creación del Paquete:** La capa de red añade su propia cabecera (con los NSAPs/IPs de origen y de destino) para formar un **paquete** (o datagrama). Si el mensaje es muy largo, la capa de red puede segmentarlo.
>5. **Creación de la Trama:** El paquete se entrega a la **capa de enlace de datos**, la cual envuelve el paquete dentro de una **trama** (añadiendo cabeceras físicas como las direcciones MAC de las tarjetas de interfaz de red) y lo transmite en forma de bits por el medio físico.

>[!info] **En el Host de Destino (Host 2):**
>1. **Recepción Física:** La tarjeta de red del Host 2 recibe la trama física, comprueba su integridad, remueve la cabecera de la trama y pasa la carga útil (el paquete) a la capa de red.
>2. **Procesamiento de Red:** La capa de red analiza la dirección de destino (NSAP/IP) en la cabecera del paquete, verifica que coincida con la entrega local, remueve dicha cabecera y pasa el segmento resultante a la entidad de transporte.
>3. **Demultiplexación por TSAP:** La entidad de transporte lee el campo del **TSAP de destino** (puerto) de la cabecera del segmento. Utiliza este valor para entregar de manera directa y exacta la carga útil al proceso de aplicación que esté escuchando activamente en ese puerto específico (el servidor de correo en el TSAP 1522).

##### ¿Cómo descubre el emisor qué TSAP (puerto) utilizar?
Dado que un proceso cliente necesita saber de antemano en qué puerto está escuchando el servidor, se emplean tres estrategias principales:
- **TSAPs estables o "Bien Conocidos" (Well-known ports):** Los servicios globales clave operan en puertos fijos y estandarizados (por ejemplo, el puerto 80 para HTTP, el 25 para SMTP o el 443 para HTTPS). Estos puertos (por debajo de 1024) vienen registrados en archivos locales del sistema operativo (como `/etc/services` en UNIX) para que las aplicaciones los localicen directamente.
- **El Portmapper (Asociador de puertos):** Para servicios efímeros o dinámicos que no tienen un puerto fijo, se utiliza un proceso especial llamado **Portmapper** que escucha en un TSAP bien conocido. El cliente se conecta primero al Portmapper, envía el nombre del servicio en texto plano (ej. "BitTorrent") y el Portmapper le devuelve el TSAP dinámico actual donde está escuchando ese proceso.
- **ICP (Initial Connection Protocol / inetd):** Mantener decenas de servidores escuchando de manera pasiva sus respectivos puertos individuales todo el día consume recursos del sistema. El ICP propone utilizar un **servidor de procesos especial** (como `inetd` en UNIX) que actúa como proxy y escucha de forma simultánea un conjunto de puertos. Cuando llega una solicitud `CONNECT` de un cliente a un puerto específico sin un servidor activo, `inetd` crea el proceso bajo demanda, le transfiere la conexión activa y vuelve a escuchar nuevas solicitudes
#### B. Establecimiento de Conexión (Apretón de Manos de Tres Vías)
##### ¿Por qué es necesario? El problema de los duplicados retrasados

En la capa de enlace (Capa 2), el establecimiento de conexión es sencillo porque los dispositivos están directamente conectados por un cable. Sin embargo, en la capa de transporte, los paquetes viajan a través de múltiples routers intermedios y redes heterogéneas.

Si la red se congestiona, un paquete de "solicitud de conexión" puede quedar atrapado en la memoria de un router durante varios segundos. Si el host emisor no recibe respuesta, asume que el paquete se perdió, agota su temporizador y envía una nueva solicitud para abrir la conexión. Si tiempo después el paquete original atrapado se libera y llega al destino (un **duplicado antiguo retrasado**), el receptor podría pensar que es una nueva solicitud de conexión y abrir un canal fantasma, duplicando transacciones críticas o desperdiciando recursos.
# Metodos de solucion a este problema
##### 1. Direcciones de transporte desechables (Disposable transport addresses)

- **Concepto:** Cada vez que una aplicación necesita abrir una conexión, se genera una dirección de transporte (un TSAP o puerto) completamente nueva y única. Al finalizar y liberar la conexión, esa dirección específica **se descarta y no se vuelve a utilizar**.
- **Cómo ataca el problema:** Si un paquete duplicado y retrasado de una conexión anterior reaparece flotando en la red mucho tiempo después, intentará entregarse a un puerto que ya fue descartado. Al no haber ningún proceso escuchando allí, el paquete se ignora y no puede causar ningún daño.
- **Desventaja:** Complica enormemente el proceso de conexión inicial, ya que los hosts deben negociar dinámicamente y descubrir de manera constante qué nuevos TSAPs utilizar en cada intento.
##### 2. Identificadores únicos de conexión con tablas de estado histórico

- **Concepto:** Se asigna un **identificador único secuencial** a cada conexión establecida. Al cerrarse la conexión, las entidades de transporte guardan este identificador dentro de una tabla de conexiones obsoletas en su memoria local.
- **Cómo ataca el problema:** Cuando llega una nueva solicitud de conexión, se compara su identificador con la tabla histórica. Si coincide con uno de la lista, se detecta de inmediato como un duplicado antiguo y se descarta.
- **Desventaja:** Requiere que la entidad de transporte almacene y mantenga esta información histórica de manera **indefinida** tanto en el origen como en el destino. El gran peligro ocurre si un host sufre una caída (_crash_) y se reinicia: al perder la memoria volátil, **pierde la tabla histórica**, quedando vulnerable a aceptar duplicados antiguos porque ya no sabe qué identificadores se usaron previamente.
##### 3. Limitar el tiempo de vida de los paquetes (Restricting packet lifetime)
Para hacer que el problema sea manejable, se puede implementar un método que impida que los paquetes vivan eternamente en la red. Si se garantiza que ningún paquete puede sobrevivir más allá de un tiempo \(T\) (en Internet se toma arbitrariamente como 120 segundos), cualquier duplicado de un paquete o de su acuse de recibo habrá muerto con absoluta certeza al pasar ese periodo.
Existen tres técnicas para restringir esta vida útil:
1. **Diseño de red restringido:** Se evita la formación de ciclos y bucles de enrutamiento físicos, y se controla la congestión para acotar el retardo en el peor camino posible.
2. **Contador de saltos (Hop limit / TTL):** Se añade un campo contador en la cabecera del paquete (como el TTL en IPv4 o Hop Limit en IPv6) que disminuye en cada enrutador. Si llega a cero, el paquete se descarta de inmediato.
3. **Marca de tiempo (Timestamping):** Cada paquete lleva registrada su hora de creación, y los routers acuerdan descartar cualquier paquete que supere un umbral de antigüedad. _Desventaja:_ Requiere mantener los relojes de todos los routers de la red perfectamente sincronizados, lo cual es sumamente difícil.

##### El método complementario: Relojes locales de Tomlinson (1975)
Una vez que el tiempo de vida del paquete está acotado por un periodo \(T\), Ray Tomlinson propuso equipar a los hosts con **relojes locales basados en contadores binarios** que siguen funcionando incluso si el host falla.
- Cuando se abre una conexión, se toman los \(k\) bits inferiores del reloj para usarlos como el **número de secuencia inicial**.
- Como el reloj avanza constantemente, los números de secuencia iniciales varían en cada conexión.
- El espacio de secuencia es tan grande que para cuando los números dan la vuelta y vuelven a empezar, cualquier segmento viejo con ese mismo número ya ha muerto de forma natural al superar el tiempo \(T\).
- _Nota de diseño:_ Aunque este método basado en reloj soluciona la duplicación durante la transmisión de datos, el **apretón de manos de tres vías** se sigue necesitando obligatoriamente justo al inicio. Esto es porque el host de destino no recuerda de forma activa los números de secuencia de conexiones ya cerradas, por lo que necesita el intercambio de tres pasos para verificar que la solicitud basada en el reloj del emisor sea realmente actual y no un duplicado antiguo.
#### Acuerdo de tres vias
##### El Proceso Paso a Paso (Funcionamiento Normal)
En una arquitectura cliente-servidor típica (como un navegador web conectándose a un servidor), el proceso se ejecuta de la siguiente manera:

```
  Cliente (Host 1)                             Servidor (Host 2)
  (Apertura Activa)                             (Apertura Pasiva)
          |                                             |
          | ----- 1. SYN (seq=x) ---------------------> | [LISTEN]
      [SYN SENT]                                        | [SYN RCVD]
          |                                             |
          | <---- 2. SYN+ACK (seq=y, ack=x+1) --------- |
    [ESTABLISHED]                                       |
          |                                             |
          | ----- 3. ACK (seq=x+1, ack=y+1) ----------> |
          |                                       [ESTABLISHED]
          v                                             v
```

#### **Paso 1: Solicitud de Conexión (SYN)**
El cliente inicia activamente el proceso ejecutando la primitiva `CONNECT`. Su entidad de transporte envía un segmento especial con el bit de control **SYN (Synchronize)** activado en `1` (y el bit `ACK` en `0`).

- Este segmento contiene el **número de secuencia inicial del cliente (\(x\))**, el cual se elige de manera pseudoaleatoria para evitar ataques de predicción.
- _Nota:_ Este segmento SYN consume exactamente 1 byte del espacio de secuencia para poder recibir acuse de recibo de manera clara. El cliente pasa al estado `SYN SENT`.
#### **Paso 2: Confirmación y Respuesta (SYN + ACK)**
El servidor, que se encuentra en estado `LISTEN` esperando conexiones entrantes, recibe el segmento. Si decide aceptar la conexión, la entidad de transporte del servidor pasa al estado `SYN RCVD` y responde enviando un segmento con los bits **SYN = 1 y ACK = 1**.

- El campo de acuse de recibo (**acknowledgment**) se establece en **\(x + 1\)** para confirmar que recibió con éxito el SYN del cliente.
- Además, el servidor propone su propio **número de secuencia inicial (\(y\))** para el tráfico de vuelta (servidor a cliente).
#### **Paso 3: Confirmación Final (ACK)**
Al recibir la respuesta del servidor, el cliente se desbloquea. Para finalizar el protocolo, envía un segmento final de confirmación con el bit **ACK = 1** (y el bit `SYN = 0`).

- Este segmento lleva como número de secuencia \(x + 1\) y confirma la recepción del número de secuencia del servidor estableciendo el campo de acuse de recibo en **\(y + 1\)**.
- Este tercer paquete **ya puede contener datos normales de usuario** en su carga útil.
- Una vez transmitido/recibido este segmento, ambos extremos entran en el estado **`ESTABLISHED`**, la tubería lógica queda formalmente abierta y el intercambio bidireccional de datos puede comenzar.
##### ¿Cómo previene los fallos por duplicados antiguos?
Tomlinson demostró que ninguna combinación de segmentos antiguos retrasados puede engañar a este protocolo para establecer una conexión falsa si uno de los hosts no la desea.
- **Escenario A: Llega un duplicado de solicitud antiguo (CR):** Si una solicitud antigua `SYN (seq=x)` que andaba perdida en la red llega al servidor de la nada, el servidor la procesa y responde enviando un `SYN+ACK` con el acuse de recibo `ack=x+1` y su propia secuencia `y`. Al recibir este paquete, el cliente (Host 1) se da cuenta de que él no ha enviado ninguna solicitud de conexión reciente con secuencia `x`. Por lo tanto, el cliente **rechaza la conexión** enviando un segmento de reinicio o rechazo (`RST` o `REJECT`). El servidor se da cuenta de que fue engañado por un duplicado retrasado y aborta el intento sin consecuencias.
- **Escenario B: Llega una solicitud antigua y un ACK antiguo a la vez:** Si coinciden en la red una solicitud antigua retrasada y un acuse de recibo antiguo, el servidor responderá a la solicitud antigua proponiendo una secuencia `y`. Al recibir después el acuse de recibo antiguo retrasado que hace referencia a una secuencia antigua `z` (en lugar de reconocer la secuencia actual `y`), el servidor detecta inmediatamente el desajuste numérico, identifica que se trata de otro duplicado antiguo y descarta la conexión de inmediato.
##### Vulnerabilidad y Mitigación: El Ataque SYN Flood
Dado que el servidor tradicionalmente debe reservar espacio de memoria y recordar su número de secuencia propuesto (\(y\)) en cuanto responde en el **Paso 2** (mientras espera el **Paso 3** del cliente), este protocolo presenta una vulnerabilidad de seguridad conocida como **Inundación SYN (SYN Flood)**.
Un atacante malicioso puede inundar un servidor web enviando miles de segmentos SYN falsificados (con IPs de origen inexistentes) y nunca responder con el ACK final. Esto satura las colas de conexión del servidor y agota sus recursos de memoria.
#### **La Solución: SYN Cookies**
Para mitigar esto, los sistemas operativos modernos utilizan una técnica llamada **cookies SYN**.
- En lugar de almacenar el estado y reservar memoria al enviar el `SYN+ACK`, el servidor calcula un número de secuencia inicial \(y\) **criptográficamente generado** basándose en la IP de origen, puerto de origen, puerto de destino y un secreto local. El servidor envía este valor como su secuencia y se "olvida" de la conexión temporalmente.
- Si el cliente es legítimo y responde con el `ACK` final (que contendrá \(y + 1\)), el servidor resta `1` a ese número, vuelve a ejecutar de forma local la función criptográfica y comprueba si coincide. Si es correcto, reconstruye la conexión en ese preciso instante en la memoria, frustrando por completo el ataque de denegación de servicio.












