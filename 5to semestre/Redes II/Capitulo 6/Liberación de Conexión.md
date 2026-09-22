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
El bit RST es un mensaje especial diseñado originalmente para restablecer de forma inmediata una conexión que se ha vuelto confusa o que ha sufrido un error grave (como la caída de un host). Sin embargo, se le da un uso estratégico para cerrar conexiones normales de forma más rápida.#### D. Control de Errores de Extremo a Extremo
En la capa de enlace se protege un tramo de cable individual. La capa de transporte realiza una verificación de extremo a extremo.

- Esto responde al **argumento de extremo a extremo** (_end-to-end argument_): si un paquete se corrompe internamente en la memoria de un router intermedio, las capas de enlace individuales no lo detectarán porque la trama se recalcula en cada salto. Solo la suma de comprobación (_checksum_) de la capa de transporte, calculada en el emisor y verificada en el destino final, asegura la integridad del paquete. Para corregir fallas, utiliza retransmisiones automáticas (**ARQ** - _Automatic Repeat reQuest_).
#### E. Control de Errores y Flujo

# 1. Control de Errores
El **control de errores** busca garantizar que los datos se entreguen al proceso receptor con total fiabilidad, libres de corrupción, en orden y sin duplicados.

- **Suma de comprobación de extremo a extremo (_End-to-End Checksum_):** En la capa de enlace, la detección de errores sólo protege la trama durante su trayecto por un único cable o enlace físico entre dos routers adyacentes. En cambio, en la capa de transporte la verificación es de **extremo a extremo** (_end-to-end_), protegiendo al segmento a lo largo de toda la ruta por la red.
- **El argumento de extremo a extremo (_End-to-End Argument_):** Propuesto por Saltzer et al., este principio demuestra que el control de errores en cada enlace individual no es suficiente, ya que los paquetes pueden corromperse internamente dentro de la memoria RAM o las tablas de un enrutador intermedio. Aunque la capa de enlace de cada tramo no detecte fallos, la suma de comprobación de la capa de transporte en el receptor sí los identificará y solicitará la retransmisión.
- **Mecanismos ARQ (_Automatic Repeat reQuest_):** Para corregir pérdidas o segmentos dañados, la entidad de transporte emisora mantiene un temporizador por segmento. Si el acuse de recibo (**ACK**) no llega antes de que el temporizador venza, el segmento se retransmite automáticamente.
# 2. Control de Flujo y Gestión Dinámica de Búferes

El **control de flujo** evita que un emisor rápido sobrepase a un receptor lento que carece de capacidad suficiente en sus búferes para procesar los datos a tiempo.

- **Diferencia de grado con la capa de enlace:** A diferencia de la capa de enlace (donde los enlaces físicos suelen usar ventanas muy pequeñas como _stop-and-wait_ debido al bajo retardo), en transporte el producto ancho de banda-retardo es grande y se requieren **ventanas deslizantes mucho mayores**. Esto exige gestionar grandes cantidades de memoria (_buffers_) tanto en el emisor (para guardar copias por si se requiere reenviar) como en el receptor (para reordenar y almacenar antes de entregar a la aplicación).
- **Organización de los Búferes en el Receptor:** La entidad receptora organiza sus búferes de tres formas principales según la variabilidad del tráfico:
    1. _Pool de búferes de tamaño fijo:_ Se asigna un búfer idéntico por cada segmento. Es simple, pero desperdicia espacio si llegan segmentos pequeños.
    2. _Búferes de distintos tamaños:_ Maneja mejor la mezcla de paquetes cortos y largos, aunque con mayor complejidad de gestión.
    3. _Un solo búfer circular por conexión:_ Reserva un espacio contiguo para el flujo completo.
- **Ventana de Tamaño Variable (Desacople del ACK):** A diferencia de los protocolos de ventana fija de capa 2, en transporte se desacopla la confirmación del segmento de la asignación de memoria. El receptor le informa activamente al emisor la cantidad de espacio libre en sus búferes mediante el campo **Tamaño de Ventana** (_Window size_) incluido en los ACKs devueltos (como hace TCP). Cada vez que el emisor transmite, descuenta ese espacio de su crédito autorizado hasta llegar a cero.
- **Sondeos de Ventana (_Window Probes_):** Si el receptor anuncia una ventana de `0`, el emisor debe detener la transmisión. Para evitar que la pérdida de un posterior anuncio de ventana provoque un bloqueo permanente, el emisor transmite de forma periódica un **sondeo de ventana** de 1 byte para obligar al receptor a responder reanunciando su espacio de búfer.
# 3. Diferencia entre Control de Flujo y Control de Congestión

Es muy común confundir ambos términos, pero apuntan a cuellos de botella distintos:

- **Control de flujo:** Relación directa entre **emisor y receptor** (evita ahogar al receptor por falta de búferes).
- **Control de congestión:** Relación entre el **emisor y la red** (evita saturar la capacidad de transporte de los enrutadores intermedios).
# Manejo dinámico del tamaño de ventana
Es el mecanismo mediante el cual la capa de transporte (como en TCP) regula de forma flexible la cantidad de datos que un emisor puede transmitir sin quedarse esperando una confirmación.
#### 1. Desacople entre Acuses de Recibo y Búferes
La clave de la gestión dinámica es que el protocolo **desacopla el acuse de recibo de la concesión de crédito para enviar más datos**.

En cada segmento de retorno, el receptor le comunica al emisor dos cosas distintas:
1. **Acuse de recibo (ACK):** Indica hasta qué byte ha recibido los datos de forma correcta.
2. **Tamaño de ventana (_Window Size_):** Informa de manera independiente cuántos bytes adicionales tiene espacio para almacenar en sus búferes en ese preciso instante.

>[!info] Proceso de Intercambio y Flujo Dinámico
>1. **Anuncio Inicial:** Durante el establecimiento de la conexión o en el envío de datos, el emisor transmite segmentos y el receptor responde con un tamaño de ventana proporcional a la memoria libre que le queda.
>2. **Consumo de Crédito:** Cada vez que el emisor transmite un segmento, descuenta esa cantidad de bytes de su asignación o crédito permitido.
>3. **Bloqueo por Ventana Cero:** Si el proceso de aplicación en el receptor no lee los datos a tiempo y los búferes se llenan, el receptor envía un `WIN = 0`. Al recibir esto, el emisor **se detiene de inmediato** y no envía más datos normales.
>4. **Sondeos de Ventana (_Window Probes_):** Si se perdiera el paquete posterior donde el receptor avisa que volvió a liberar espacio, la conexión quedaría en un bloqueo permanente. Para evitarlo, el emisor transmite periódicamente un paquete especial de 1 byte denominado **sondeo de ventana** (_window probe_), forzando al receptor a responder re-anunciando su estado actual.

 >[!success] La Ventana Efectiva: Control de Flujo vs. Control de Congestión
En la práctica, el emisor mantiene **dos ventanas dinámicas simultáneas** para no saturar al receptor ni a los routers intermedios:
>- **Ventana de Control de Flujo (\(WIN_{receptor}\)):** Dictada y anunciada explícitamente por el receptor según su memoria disponible.
>- **Ventana de Congestión (\(cwnd\)):** Calculada dinámicamente por el propio emisor evaluando la capacidad de la red (mediante algoritmos como _Slow Start_ e _AIMD_ al detectar o evitar la pérdida de paquetes).
>El emisor determina su **Ventana Efectiva** calculando el **mínimo entre ambas**: $\text{Ventana Efectiva} = \min(WIN_{receptor}, cwnd)$

De esta manera, el flujo de datos se adapta en todo momento al cuello de botella más estricto.
#### Problemas de Eficiencia y Soluciones
- **Síndrome de la Ventana Tonta (_Silly Window Syndrome_):** Ocurre si la aplicación receptora lee los datos de a 1 byte. El receptor enviaría actualizaciones constantes proponiendo `WIN = 1`, haciendo que el emisor mande paquetes pequeños llenos de cabeceras innecesarias.
- **Solución (Regla de Clark):** Se prohíbe al receptor anunciar ventanas diminutas; debe esperar a tener libre al menos el tamaño de un segmento máximo (MSS) o la mitad de su búfer total antes de enviar una actualización.
#### F. Multiplexación y Multiplexación Inversa
- **Multiplexación:** Permite que múltiples conexiones de transporte compartan una única interfaz y dirección IP de red.
- **Multiplexación inversa:** Permite que una única conexión de transporte distribuya su tráfico a través de múltiples rutas de red físicas de forma paralela (como lo hace el protocolo SCTP) para incrementar el ancho de banda efectivo y la fiabilidad.
