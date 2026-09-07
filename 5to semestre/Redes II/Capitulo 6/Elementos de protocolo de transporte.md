Los servicios de esta capa se implementan mediante **protocolos de transporte** que se ejecutan entre las entidades de transporte de los sistemas finales. Aunque comparten la necesidad de gestionar el control de errores, el orden de los datos y el control de flujo con la capa de enlace (Capa 2), el entorno de transporte es infinitamente más complejo porque abarca la red completa en lugar de un único enlace directo.
#### A. Direccionamiento (TSAP y NSAP)

En un enlace directo de capa de enlace, el destino del paquete es obvio. En cambio, en la capa de transporte se requiere **direccionamiento explícito**.

- **NSAP (Network Service Access Point):** Es la dirección del host en la capa de red (por ejemplo, la dirección IP).
- **TSAP (Transport Service Access Point):** Es el punto final específico dentro del host donde escucha un proceso (los conocidos **puertos** TCP/UDP). Los TSAPs permiten multiplexar la red para que múltiples aplicaciones independientes compartan de manera simultánea una única dirección IP.
#### B. Establecimiento de Conexión (Apretón de Manos de Tres Vías)
A diferencia de un cable físico simple, la red puede almacenar, retrasar y duplicar paquetes de forma temporal. Si una máquina envía una solicitud de conexión, se agota el tiempo de espera, se envía una nueva y la primera aparece retrasada mucho tiempo después (un **duplicado antiguo**), el receptor podría establecer una conexión fantasma o duplicar transacciones críticas (como transferencias bancarias redundantes).

- Para solucionar esto, los protocolos implementan el **apretón de manos de tres vías** (_three-way handshake_). Este mecanismo asegura que ambos extremos propongan y confirmen números de secuencia iniciales unívocos, descartando cualquier segmento de control duplicado del pasado.
#### C. Liberación de Conexión
La desconexión puede ser **asimétrica** (se corta bruscamente una dirección, lo que puede provocar la pérdida de datos que estaban en tránsito) o **simétrica** (cada dirección de la comunicación bidireccional se cierra de manera independiente).

- La dificultad técnica de lograr un acuerdo perfecto para cerrar la conexión sobre una red no confiable se modela mediante el **Problema de los dos ejércitos** (_Two Armies Problem_), el cual demuestra matemáticamente que es imposible garantizar una sincronización perfecta de liberación si los mensajes de confirmación pueden perderse.
#### D. Control de Errores de Extremo a Extremo
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

