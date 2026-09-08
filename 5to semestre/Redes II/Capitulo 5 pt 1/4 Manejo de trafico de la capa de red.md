## Control de congestion vs control de flujo
# Control de congestión (Congestion Control)

El control de congestión es un **problema global** que involucra a toda la red o a una parte de ella.

- **Objetivo:** Asegurar que la red sea capaz de transportar el tráfico ofrecido por todos los usuarios. Afecta el comportamiento de **todos los hosts y enrutadores**.
- **Causa:** Ocurre cuando se ofrece demasiado tráfico a la red, superando su capacidad de transporte interna. Esto provoca que las colas en los routers se llenen, aumente la latencia y se pierdan paquetes.
- **Mecanismo común:** Requiere la cooperación de las capas de red y transporte. Se utilizan técnicas como el **control de admisión**, **notificación explícita de congestión (ECN)**, y la gestión de la **ventana de congestión (cwnd)** en protocolos como TCP para reducir la carga en la red.
- **Ejemplo:** 1,000 computadoras conectadas a líneas de 1 Mbps. Si 500 de ellas intentan transferir archivos a las otras 500 simultáneamente, el tráfico total superará con creces lo que la red puede soportar, independientemente de la velocidad de los receptores.
# Control de flujo (Flow Control)

El control de flujo se refiere específicamente al tráfico entre un **emisor y un receptor concreto**.

- **Objetivo:** Garantizar que un emisor rápido no transmita datos más rápido de lo que un receptor lento puede absorberlos.
- **Causa:** Se debe a una limitación de recursos (como el espacio de búfer) en el **extremo receptor**, no en la red.
- **Mecanismo común:** Se implementa típicamente en la capa de transporte mediante protocolos de **ventana deslizante**, donde el receptor informa al emisor cuánto espacio libre queda en su búfer (ventana de recepción).
- **Ejemplo:** Una supercomputadora enviando un archivo de gran tamaño a una PC a través de una red de 100 Gbps. La red es capaz de manejar el tráfico, pero la PC (el receptor) solo puede procesar 1 Gbps, por lo que el emisor debe detenerse para dejarla "respirar".
## Network Provisioning (Aprovisionamiento de red)

Es el enfoque más lento y de carácter puramente preventivo. Sus características principales incluyen:
- **Diseño robusto:** Consiste en diseñar la red de forma que sea lo suficientemente robusta para que, idealmente, no sufra congestión en ningún caso.
- **Sobre-dimensionamiento (Overprovisioning):** Se basa en dotar a la red de recursos suficientes (enlaces y routers) que no siempre se utilizan a su máxima capacidad para absorber ráfagas de tráfico.
- **Planificación a largo plazo:** El aprovisionamiento se realiza en plazos de meses, basándose en el análisis de las tendencias de tráfico a largo plazo para actualizar los equipos.
- **Costo:** Aunque es la forma más sencilla de evitar problemas, es una solución cara, ya que implica gastar dinero en capacidad que puede estar ociosa gran parte del tiempo.
# Enrutamiento Consciente del Tráfico (Traffic-Aware Routing)

Este enfoque adapta las rutas según los patrones de tráfico cambiantes para alejar la carga de los "puntos calientes" o congestionados.

- **Funcionamiento:** Se ajustan los pesos de los enlaces (métrica de ruta más corta) en función de la carga medida, el retardo medio de las colas o el ancho de banda disponible.
- **Desafío (Oscilaciones):** Si las rutas cambian bruscamente hacia enlaces menos cargados, estos se congestionan rápidamente y los anteriores se vacían, provocando que las tablas de enrutamiento oscilen salvajemente.
- **Soluciones:**
    - **Enrutamiento multitrayectoria:** Repartir el tráfico entre múltiples rutas simultáneas hacia un mismo destino.
    - **Ajustes lentos:** Desplazar el tráfico de forma gradual para permitir la convergencia del sistema.
    - **Ingeniería de Tráfico:** En Internet, los cambios suelen hacerse manualmente o fuera del protocolo debido a la imprevisibilidad de las cargas.
# Control de Admisión (Admission Control)

Se utiliza principalmente en redes de **circuitos virtuales** para evitar que la red acepte más carga de la que puede transportar de forma segura.

- **Funcionamiento:** No se establece una nueva conexión o circuito virtual a menos que la red confirme que tiene capacidad suficiente para soportar ese tráfico adicional sin congestionarse.
- **Caracterización del tráfico:** Para que funcione, el emisor debe describir su patrón de tráfico (velocidad y forma) mediante descriptores como la **"cubeta con goteo"** o **"cubeta con tokens"**.
- **Negociación:** Si la red no puede garantizar los recursos, la solicitud de conexión falla, de forma similar a cuando un sistema telefónico no da tono de llamada por sobrecarga.
# Regulación y Modelado de Tráfico (Traffic Shaping)

Es una técnica para suavizar el flujo de datos que entra a la red, regulando la tasa promedio y la intensidad de las ráfagas.

- **Cubeta con Goteo (Leaky Bucket):** Fuerza una tasa de salida constante (R bits/seg). Si el host envía ráfagas, se almacenan en un buffer (B bytes); si el buffer se llena, el exceso de agua (paquetes) se desborda y se pierde.
- **Cubeta con Tokens (Token Bucket):** Permite ráfagas de hasta un tamaño B, pero a una tasa promedio R. Para enviar un paquete, el host debe "gastar" un token del cubo. Si el cubo está vacío, el host debe esperar a que se generen nuevos tokens.
- **Vigilancia del tráfico (Traffic Policing):** Es el proceso donde el proveedor de red supervisa el flujo para asegurarse de que el cliente cumple con el acuerdo (SLA); si el tráfico excede lo pactado, los paquetes se descartan o se les baja la prioridad.
# Entrega de Retroalimentación

Cuando un enrutador detecta que la congestión es inminente o ya existe (monitoreando la carga de enlaces o el retardo de colas mediante algoritmos como **EWMA (Media Movil ponderada exponencialmente)**), debe informar a los emisores para que reduzcan su velocidad
>[!important] Funcionamiento:
>El objetivo de EWMA es **suavizar las fluctuaciones** instantáneas de una métrica para obtener una tendencia más estable. Funciona de manera similar a un **filtro de paso bajo**, descartando el "ruido" o variaciones bruscas y momentáneas en las muestras recolectadas.
>2. Funcionamiento Matemático
Para calcular un nuevo valor estimado (dnuevo​), el algoritmo utiliza una muestra actual (s) y el valor estimado anterior (dantiguo​), aplicando una constante de suavizado α:
$dnuevo​=α⋅dantiguo​+(1−α)⋅s$
>- **La constante** α**:** Determina la rapidez con la que el enrutador o el host "olvida" la historia reciente.
>- Si α es alto (cercano a 1), se da más peso al historial y el valor cambia lentamente.
>- En la práctica de TCP, se suele utilizar un valor de α=7/8 para el cálculo del RTT suavizado.

Existen dos mecanismos principales para entregar esta señal:

- **Paquetes Reguladores (Choke Packets):** El router selecciona un paquete del flujo congestionado y envía un mensaje directo al host de origen (como el mensaje **ICMP Source Quench**) pidiéndole que reduzca su velocidad, por ejemplo, en un 50%.
- **Notificación Explícita de Congestión (ECN):** En lugar de generar paquetes nuevos, el enrutador marca un bit específico en la cabecera de los paquetes que pasan a través de él. El receptor nota esta marca y se la comunica al emisor en su siguiente mensaje de respuesta, permitiendo que el host disminuya su tasa de envío de forma preventiva.

# Contrapresión de Salto por Salto (Hop-by-hop backpressure)

Esta técnica se utiliza para resolver la ineficiencia de los paquetes reguladores en redes de alta velocidad o largas distancias, donde la señal de retroalimentación tarda demasiado en llegar al origen mientras se siguen enviando megabits de datos.

- **Funcionamiento:** A diferencia de la señal que solo afecta a la fuente, la contrapresión hace que el paquete regulador tenga un **efecto inmediato en cada salto** por el que pasa.
- **Mecanismo:** Cuando un enrutador recibe un paquete de estrangulamiento, reduce inmediatamente el flujo hacia el siguiente nodo. Esto obliga al nodo anterior a dedicar más búferes temporalmente, proporcionando un **alivio instantáneo** en el punto crítico de congestión, como un "remedio para el dolor de cabeza". El efecto se propaga hacia atrás, salto a salto, hasta llegar finalmente al host emisor.
# Desprendimiento de Carga (Load Shedding)

Es la solución de "último recurso" cuando todos los demás métodos fallan y el router ya no puede manejar más paquetes.

- **Descarte de paquetes:** El router simplemente desecha los paquetes excedentes.
- **Estrategia del Vino vs. Leche:**
    - **Vino (lo viejo es mejor):** Se priorizan los paquetes nuevos (útil en streaming de video/audio, donde lo viejo ya no sirve).
    - **Leche (lo nuevo es mejor):** Se priorizan los paquetes viejos (útil en transferencia de archivos, donde perder un paquete inicial obliga a retransmitir todo lo posterior).
- **RED (Random Early Detection):** Técnica que descarta paquetes al azar cuando la cola promedio sobrepasa un umbral, avisando implícitamente al emisor (vía protocolos de transporte como TCP) que debe bajar la velocidad antes de que la red colapse totalmente.
[[5 Calidad de servicio]]