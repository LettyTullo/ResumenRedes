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