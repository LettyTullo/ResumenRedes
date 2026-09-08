Los servicios de esta capa se implementan mediante **protocolos de transporte** que se ejecutan entre las entidades de transporte de los sistemas finales. Aunque comparten la necesidad de gestionar el control de errores, el orden de los datos y el control de flujo con la capa de enlace (Capa 2), el entorno de transporte es infinitamente más complejo porque abarca la red completa en lugar de un único enlace directo.
#### A. Direccionamiento (TSAP y NSAP)
En un enlace directo de capa de enlace, el destino del paquete es obvio. En cambio, en la capa de transporte se requiere **direccionamiento explícito**.
- **NSAP (Network Service Access Point):** Es la dirección del host en la capa de red (por ejemplo, la dirección IP).
- **TSAP (Transport Service Access Point):** Es el punto final específico dentro del host donde escucha un proceso (los conocidos **puertos** TCP/UDP). Los TSAPs permiten multiplexar la red para que múltiples aplicaciones independientes compartan de manera simultánea una única dirección IP.
##### El Proceso de Envío Paso a Paso (Anidamiento de Cabeceras)
El viaje de un mensaje desde el origen hasta el destino se basa en el **anidamiento sucesivo de cabeceras** en el emisor y su posterior extracción en el receptor:

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
El **apretón de manos de tres vías** (o _**three-way handshake**_, también denominado _acuerdo de tres vías_) es el método estándar y obligatorio que utilizan los protocolos de la capa de transporte (como **TCP**) para establecer de manera confiable una conexión bidireccional entre dos hosts.

Fue propuesto originalmente por **Ray Tomlinson en 1975** para solucionar uno de los problemas más difíciles en redes de comunicaciones: el establecimiento de conexiones seguras sobre canales no confiables que pueden retrasar, duplicar, perder o almacenar paquetes temporalmente.
##### ¿Por qué es necesario? El problema de los duplicados retrasados

En la capa de enlace (Capa 2), el establecimiento de conexión es sencillo porque los dispositivos están directamente conectados por un cable. Sin embargo, en la capa de transporte, los paquetes viajan a través de múltiples routers intermedios y redes heterogéneas.

Si la red se congestiona, un paquete de "solicitud de conexión" puede quedar atrapado en la memoria de un router durante varios segundos. Si el host emisor no recibe respuesta, asume que el paquete se perdió, agota su temporizador y envía una nueva solicitud para abrir la conexión. Si tiempo después el paquete original atrapado se libera y llega al destino (un **duplicado antiguo retrasado**), el receptor podría pensar que es una nueva solicitud de conexión y abrir un canal fantasma, duplicando transacciones críticas o desperdiciando recursos.

El apretón de manos de tres vías está diseñado específicamente para que ambos hosts verifiquen que la solicitud de conexión es **actual** y se pongan de acuerdo en los **números de secuencia iniciales** que usarán para el flujo de datos.
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
- _Nota:_ Este segmento SYN consume exactamente 1 byte del espacio de secuencia para poder recibir acuse de recibo de manera inequívoca. El cliente pasa al estado `SYN SENT`.
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
#### C. Liberación de Conexión
La desconexión puede ser **asimétrica** (se corta bruscamente una dirección, lo que puede provocar la pérdida de datos que estaban en tránsito) o **simétrica** (cada dirección de la comunicación bidireccional se cierra de manera independiente).
En el ámbito de las redes de computadoras, este problema ilustra por qué es teóricamente imposible lograr una **liberación de conexión perfecta y sincronizada** en la capa de transporte sin el riesgo de perder datos o dejar conexiones "semiabiertas".
##### La analogía del dilema militar
La literatura de redes plantea el problema mediante la siguiente analogía:
- Un **ejército blanco** (muy grande) se encuentra acampado en un valle.
- En las laderas de las montañas que rodean al valle, se encuentran dos **ejércitos azules** (el ejército nº 1 y el ejército nº 2).
- El ejército blanco es más fuerte que cualquiera de los dos ejércitos azules por separado. Por lo tanto, si un solo ejército azul ataca, será derrotado de forma inevitable. Sin embargo, si ambos ejércitos azules atacan de manera **simultánea**, vencerán al ejército blanco.
- Para coordinar el momento del ataque, los comandantes azules deben comunicarse. Pero su único medio de comunicación es enviar mensajeros a pie a través del valle. Debido a que el valle está patrullado por el ejército blanco, los mensajeros corren un alto riesgo de ser capturados (es decir, el **canal de comunicación no es confiable**).
##### ¿Por qué el problema no tiene solución teórica?
Si intentamos diseñar un protocolo de comunicación para que se pongan de acuerdo, este falla sistemáticamente debido a la incertidumbre inherente del canal:

1. **Propuesta inicial:** El comandante del ejército azul nº 1 envía un mensajero al ejército azul nº 2 que dice: _"Propongo que ataquemos al amanecer del 29 de marzo. ¿Qué te parece?"_.
2. **Confirmación:** El mensajero logra cruzar el valle y el comandante nº 2 responde: _"De acuerdo. Atacaré"_.
3. **La duda del receptor:** Aunque el mensaje de confirmación cruce de vuelta a salvo, el comandante nº 2 entrará en un dilema: él **no sabe si su confirmación llegó con éxito** al comandante nº 1. Si no llegó, el comandante nº 1 no atacará porque pensará que la propuesta fue rechazada o perdida. Por lo tanto, atacar para el comandante nº 2 sería un suicidio militar.
4. **El acuse de recibo (Apretón de manos de tres vías):** Para dar tranquilidad al comandante nº 2, cambiamos el protocolo para que el comandante nº 1 deba enviar un acuse de recibo (_ACK_) de la confirmación.
5. **Incertidumbre infinita:** Ahora la duda pasa al comandante nº 1, quien pensará: _"Sé que aceptó mi plan y sé que recibí su respuesta. Pero no sé si él recibió mi acuse de recibo final. Si no lo hizo, él no se arriesgará a atacar"_.

Este ciclo de confirmaciones (_un acuse de recibo del acuse de recibo del acuse de recibo..._) se puede extender infinitamente sin llegar jamás a un acuerdo seguro. Se demuestra matemáticamente que **ningún protocolo con un número finito de mensajes funciona**:

- Para que un protocolo funcione, el último mensaje enviado debe ser **esencial** para tomar la decisión.
- Dado que el emisor de ese último mensaje nunca puede estar completamente seguro de que su envío llegó a destino (debido a la inestabilidad del canal), no se arriesgará a actuar.
- Y como el otro ejército sabe que el emisor del último mensaje no se arriesgará, él tampoco actuará, frustrando el acuerdo.
##### Su aplicación a la liberación de conexiones
Para entender cómo afecta esto a las redes de datos, solo hace falta **sustituir la palabra "atacar" por "desconectar"** (liberación de la conexión).

Cuando un cliente y un servidor deciden cerrar una conexión activa de forma simétrica (es decir, cerrando de forma independiente y segura cada sentido de la transmisión), **ninguna de las dos máquinas querrá borrarse de las tablas de la entidad de transporte hasta estar 100% segura de que la otra máquina también se ha desconectado**. Como la red intermedia puede perder paquetes, es teóricamente imposible alcanzar esa certeza absoluta.
##### La solución práctica en la ingeniería de protocolos
Dado que la teoría demuestra que no hay solución perfecta, la ingeniería de redes recurre a **soluciones prácticas aproximadas** para evitar que los sistemas se queden congelados indefinidamente en conexiones a medias:

- **Handshake de 3 pasos con temporizadores (_timers_):** El host que inicia la desconexión envía un segmento de solicitud de desconexión (DR - _Disconnection Request_) y activa un temporizador local. Si tras enviar el paquete varias veces no recibe respuesta debido a pérdidas consecutivas en la red, el host **se rinde tras un número \(N\) de intentos y se desconecta de forma unilateral**. El otro extremo, al agotarse su propio temporizador por falta de actividad, eventualmente hará lo mismo, previniendo que los recursos queden bloqueados para siempre.
- **Regla de desconexión automática por inactividad:** Para resolver el problema de las conexiones semiabiertas (donde un lado se desconecta pero el otro sigue activo sin saberlo), se establece que si un host no recibe ningún tipo de tráfico durante un número determinado de segundos, la conexión se aborta automáticamente. Para mantener conexiones legítimas abiertas durante periodos de silencio, las entidades de transporte envían de forma automática paquetes "ficticios" (_keep-alive_) de forma periódica.
- **El cierre normal (Simétrico con FIN):** En el funcionamiento estándar de TCP, para cerrar una conexión de forma limpia y sin perder datos, se utiliza un **cierre simétrico**. Como la conexión es bidireccional (full-duplex), se trata como si fueran dos conexiones independientes de un solo sentido (simplex):
	- El Host 1 envía un segmento **FIN** para avisar que terminó de enviar sus datos.
	- El Host 2 responde con un **ACK** para confirmar que lo recibió. En este punto, el canal de Host 1 a Host 2 está cerrado, pero el Host 2 todavía puede seguir enviando datos en la otra dirección si lo necesita.
	- Cuando el Host 2 también termina, envía su propio **FIN**, y el Host 1 responde con un **ACK**.
	- Este proceso normal requiere obligatoriamente el intercambio de **4 segmentos**.
##### El cierre abrupto con RST (Reset)
En lugar de pasar por este intercambio lento de mensajes de FIN y ACK, algunos servidores (especialmente los **servidores web HTTP**) optan por un **cierre abrupto** utilizando un segmento con el bit **RST (Reset)** activado.
El bit RST es un mensaje especial diseñado originalmente para restablecer de forma inmediata una conexión que se ha vuelto confusa o que ha sufrido un error grave (como la caída de un host). Sin embargo, se le da un uso estratégico para cerrar conexiones normales de forma más rápida.








