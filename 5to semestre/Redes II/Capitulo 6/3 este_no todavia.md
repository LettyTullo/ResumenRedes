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
El bit RST es un mensaje especial diseñado originalmente para restablecer de forma inmediata una conexión que se ha vuelto confusa o que ha sufrido un error grave (como la caída de un host). Sin embargo, se le da un uso estratégico para cerrar conexiones normales de forma más rápida.#### D. Control de Errores de Extremo a Extremo
En la capa de enlace se protege un tramo de cable individual. La capa de transporte realiza una verificación de extremo a extremo.

- Esto responde al **argumento de extremo a extremo** (_end-to-end argument_): si un paquete se corrompe internamente en la memoria de un router intermedio, las capas de enlace individuales no lo detectarán porque la trama se recalcula en cada salto. Solo la suma de comprobación (_checksum_) de la capa de transporte, calculada en el emisor y verificada en el destino final, asegura la integridad del paquete. Para corregir fallas, utiliza retransmisiones automáticas (**ARQ** - _Automatic Repeat reQuest_).
#### E. Control de Flujo y Asignación de Búferes (_Buffering_)
Debido a que el transporte gestiona un número alto y dinámico de conexiones con anchos de banda muy fluctuantes, no es eficiente asignar búferes fijos para cada conexión.

- Los protocolos utilizan **ventanas deslizantes de tamaño variable**. El receptor y el emisor negocian dinámicamente el tamaño del búfer disponible mediante el intercambio continuo de mensajes, indicando al emisor cuándo debe detener la transmisión para no saturar al receptor.
#### F. Multiplexación y Multiplexación Inversa
- **Multiplexación:** Permite que múltiples conexiones de transporte compartan una única interfaz y dirección IP de red.
- **Multiplexación inversa:** Permite que una única conexión de transporte distribuya su tráfico a través de múltiples rutas de red físicas de forma paralela (como lo hace el protocolo SCTP) para incrementar el ancho de banda efectivo y la fiabilidad.
#### G. Control de Congestión
Mientras que el control de flujo evita que un emisor rápido sature a un receptor lento, el **control de congestión** evita que los emisores saturen a los routers de tránsito de la propia red. Los hosts deben regular su velocidad de inyección de paquetes basándose en la retroalimentación que reciben:

- **Explícita y precisa:** Cuando los enrutadores informan la tasa exacta a usar (por ejemplo, en el protocolo XCP).
- **Implícita o imprecisa:** Deduciendo la saturación a través de la pérdida de paquetes o el incremento en los retardos de ida y vuelta (como en las distintas leyes de control AIMD de TCP).