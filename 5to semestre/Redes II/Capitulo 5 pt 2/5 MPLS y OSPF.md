## MPLS
**MPLS** (siglas de _**MultiProtocol Label Switching**_ o **Conmutación Multiprotocolo de Etiquetas**) es una tecnología de red orientada a la conexión utilizada principalmente por los proveedores de servicios de Internet (ISP) para transportar y gestionar el tráfico a través de sus redes troncales.

Aunque funciona por encima de la capa de enlace y por debajo de la capa de red (conociéndose informalmente como un protocolo de **Capa 2.5**), se caracteriza por envolver los paquetes IP dentro de una cabecera de etiquetas para acelerar y flexibilizar el reenvío de datos.
# Principio de Funcionamiento: Conmutación vs. Reenvío
En el enrutamiento IP tradicional, los routers realizan un "reenvío" (_forwarding_), lo que implica buscar en una tabla la coincidencia de prefijo más largo para la dirección IP de destino de cada paquete. Este es un proceso de búsqueda que puede ser lento en redes de alta velocidad.

**MPLS introduce la conmutación (_switching_):**
- Cuando un paquete IP entra a la red MPLS, un enrutador de borde le añade una **etiqueta** en el frente.
- Los routers internos de la red MPLS ya no analizan la dirección IP de destino. En su lugar, utilizan el valor numérico de la etiqueta como un **índice directo** en una tabla interna de reenvío.
- Esto hace que encontrar la línea de salida correcta sea una búsqueda indexada extremadamente rápida y sencilla, optimizando el rendimiento del hardware.
# La Cabecera MPLS (4 bytes)
La cabecera de etiquetas se inserta justo antes de la cabecera IP. Tiene una longitud de **4 bytes** y consta de los siguientes cuatro campos:

![[Pasted image 20260830181115.png|498]]

1. **Etiqueta (20 bits):** Contiene el índice de conmutación. Estas etiquetas tienen únicamente un **significado local** entre routers adyacentes y se reasignan en cada salto para evitar conflictos de numeración.
2. **QoS (3 bits):** Indica la clase de servicio para dar prioridad al paquete (Calidad de Servicio).
3. **S (1 bit):** Bandera de apilamiento (_stacking_). Permite apilar múltiples etiquetas en un solo paquete; se pone en **1** para la etiqueta que está en el fondo de la pila y en **0** para las demás.
4. **TTL (8 bits):** Tiempo de vida (_Time to Live_). Funciona igual que en IP: se decrementa en cada router y, si llega a 0, el paquete se descarta para evitar bucles infinitos en caso de fallos de enrutamiento.
# Componentes Clave de una Red MPLS
El ecosistema MPLS define dispositivos y clasificaciones específicas para el manejo del tráfico:

- **LER (Label Edge Router / Enrutador de Etiquetas de Borde):** Se ubican en las fronteras de la red MPLS. Su función es examinar el paquete IP entrante, determinar qué ruta debe seguir, colocar la etiqueta adecuada (_push_) y, al salir de la red, retirar la etiqueta (_pop_) para entregar el paquete IP original intacto a la red destino.
- **LSR (Label Switched Router / Enrutador de Conmutación de Etiquetas):** Son los enrutadores internos de la red. Reciben el paquete etiquetado, consultan su tabla, reemplazan la etiqueta vieja por una nueva (_swap_) y reenvían el paquete por la línea de salida correspondiente.
- **FEC (Forwarding Equivalence Class / Clase de Equivalencia de Reenvío):** Es un grupo de flujos de datos que terminan en un destino común o comparten requerimientos de servicio. En lugar de asignar etiquetas individuales a cada conexión, el router agrupa múltiples flujos bajo una **misma etiqueta de FEC**, lo que permite una agregación eficiente del tráfico.
# Apilamiento de Etiquetas (_Label Stacking_)
Gracias al bit **S**, MPLS puede operar a varios niveles de jerarquía de forma simultánea. Si varios flujos que ya tienen etiquetas individuales deben seguir una ruta común a través de la red, el router puede agregar una **segunda etiqueta exterior** en el paquete (creando una pila). El tráfico viaja guiado por la etiqueta externa y, al finalizar el trayecto común, esta etiqueta es retirada, revelando la etiqueta interna original para que continúe su curso individual.


>[!tip] Ventajas y Aplicaciones de MPLS
>- **Independencia de protocolo (Multiprotocolo):** Al no formar parte de la capa de enlace ni de red, MPLS es independiente de ambas. Puede transportar tanto paquetes IP como no IP, o mover paquetes IP sobre infraestructuras físicas que no utilicen direccionamiento IP.
>- **Ingeniería de Tráfico:** Permite a los operadores calcular y establecer de antemano rutas específicas que eviten los enlaces congestionados, aprovechando la capacidad ociosa de la red de forma dinámica (algo que el enrutamiento IP tradicional por ruta más corta no puede hacer de forma eficiente).
>- **Redes Privadas Virtuales (VPN MPLS):** Los ISP utilizan MPLS para configurar túneles seguros para empresas. Al etiquetar y separar el tráfico de cada cliente, se garantiza el aislamiento de datos y se pueden ofrecer acuerdos de nivel de servicio (SLA) con ancho de banda y QoS garantizados a través de la red pública.

## OSPF
**OSPF** (siglas de _**Open Shortest Path First**_ o **El camino más corto primero abierto**) es un protocolo de enrutamiento dinámico de tipo **estado de enlace** desarrollado por el IETF que se convirtió en estándar en 1990. Está clasificado como un protocolo de pasarela interior o intradominio (**IGP - Interior Gateway Protocol**), lo que significa que se utiliza para gestionar el enrutamiento de paquetes dentro de una única red independiente o **Sistema Autónomo (AS)**, como la red de una corporación o una universidad.

# ¿Cómo funciona OSPF? (Mecánica de Estado de Enlace)

A diferencia de los protocolos de vector de distancia (como RIP) que solo intercambian tablas de distancias estimadas con sus vecinos inmediatos, OSPF requiere que los routers sigan un proceso de cinco pasos para calcular rutas basándose en un mapa completo de la red:

1. **Descubrir a los vecinos:** Al arrancar, el router envía paquetes especiales **HELLO** por todas sus líneas para descubrir qué otros routers están conectados a él directamente.
2. **Fijar costos de los enlaces:** El operador de red o el sistema asigna una métrica de distancia o costo a cada enlace. Una opción común es hacer que el costo sea inversamente proporcional al ancho de banda, de modo que una interfaz de 1 Gbps sea preferible a una de 100 Mbps.
3. **Construir paquetes de estado de enlace (LSP):** Cada router crea un paquete de estado de enlace (LSP o _Link State Packet_) que contiene su identidad, un número de secuencia, un campo de "edad" y la lista de sus vecinos con sus respectivos costos de enlace.
4. **Inundación confiable (_Flooding_):** Los routers inundan estos paquetes de estado de enlace a través de toda la red. Para que la inundación sea confiable y controlada, se utilizan:
    - **Números de secuencia:** Permiten a los routers saber si han recibido un paquete LSP nuevo o uno duplicado/antiguo (el cual descartan).
    - **Edad del paquete:** Un contador que disminuye con el tiempo; cuando llega a cero, la información se vuelve obsoleta, lo que ayuda a limpiar datos antiguos o solucionar caídas de routers.
5. **Cálculo de nuevas rutas con Dijkstra:** Una vez que todos los routers de la red han recibido los LSP de todos los demás, cada uno de ellos construye un grafo idéntico de la topología completa de la red. Con este mapa, cada router ejecuta localmente el **algoritmo de Dijkstra** para encontrar el camino de menor costo hacia cualquier destino, construyendo así su propio **árbol sumidero** para configurar su tabla de reenvío.
# Características clave de OSPF

- **Balanceo de carga (ECMP - Equal Cost MultiPath):** Si OSPF detecta múltiples rutas distintas hacia un mismo destino con el mismo costo mínimo, las recuerda todas y **divide el tráfico de forma equitativa** entre ellas. Esto ayuda a balancear la carga de la red, a diferencia de protocolos antiguos que enviaban todos los paquetes por una única ruta activa.
- **Seguridad integrada:** OSPF requiere un nivel mínimo de seguridad para autenticar los mensajes, evitando de este modo que se inyecte información de enrutamiento falsa dentro de la red.
- **Optimización en Redes de Difusión (Router Designado - DR):** En redes multiacceso con capacidad de difusión (como una LAN Ethernet), no es eficiente que todos los routers intercambien información directamente entre sí. Para optimizar esto, OSPF elige un **Router Designado (DR)**. El DR actúa como el único representante de la LAN y es adyacente a todos los demás enrutadores de la subred para centralizar el intercambio de información. También se elige un router designado de respaldo para contingencias.
# Enrutamiento Jerárquico: División en Áreas

Para permitir que el protocolo sea altamente escalable y evitar que los routers de redes muy grandes tengan que memorizar topologías gigantescas, OSPF permite dividir un Sistema Autónomo en **áreas numeradas contiguas** que no se solapan. Los componentes de esta jerarquía son:
- **Área 0 (Área troncal o _Backbone_):** Es el corazón de la topología jerárquica de OSPF. Todas las demás áreas deben conectarse al área 0, ya sea de forma física o a través de un túnel. Los paquetes que viajan entre distintas áreas deben pasar obligatoriamente por el _backbone_.
- **Routers de frontera de área (ABR - Area Border Router):** Son routers conectados a dos o más áreas. Su función es resumir la información de costos de un área e inyectarla en el resto de áreas conectadas, sin transmitir los detalles de la topología interna, lo que simplifica los cálculos de Dijkstra de los routers de otras áreas.
- **Área Stub (acoplada):** Es un área que tiene un único punto de salida hacia el resto del AS. En lugar de recibir un resumen detallado de todas las redes externas, sus routers internos simplemente configuran una regla por defecto que dice _"Ir al router de frontera"_.
# Los cinco tipos de mensajes en OSPF
1. **Hello:** Utilizado para descubrir y mantener el contacto con los routers vecinos.
2. **Link State Update (Actualización del estado de enlace):** Mensaje de inundación que proporciona los costos actuales de un router hacia sus vecinos directos.
3. **Link State Ack:** Acuse de recibo que confirma la recepción de una actualización de estado de enlace para que el proceso sea confiable.
4. **Database Description:** Anuncia los números de secuencia de todas las actualizaciones que el emisor posee para que el receptor pueda comparar y ver cuáles le faltan.
5. **Link State Request:** Petición que hace un router a su vecino para solicitar información específica o más reciente de un estado de enlace.
## BGP
**BGP** (siglas de _**Border Gateway Protocol**_ o **Protocolo de Puerta de Enlace de Frontera**) es el protocolo de enrutamiento interdominio o exterior exclusivo y obligatorio que se utiliza para interconectar los diferentes Sistemas Autónomos (AS) que componen Internet.

A diferencia de los protocolos intradominio (como OSPF o RIP), cuyo principal objetivo es encontrar la ruta físicamente más rápida o corta, **BGP está diseñado para implementar políticas de enrutamiento** basadas en criterios económicos, de seguridad o geopolíticos pactados comercialmente entre los operadores de red.
# Mecánica del Protocolo: Vector de Ruta
BGP se clasifica como un protocolo de **vector de ruta** (_path vector_), lo cual es una evolución de los protocolos de vector de distancia:
- **La ruta de AS (AS Path):** En lugar de registrar únicamente una métrica numérica de costo, cada router BGP mantiene un registro detallado de la **secuencia de Sistemas Autónomos** que un paquete debe atravesar para llegar a un prefijo de destino.
- **Prevención de bucles:** Cuando un router envía un anuncio de ruta fuera de su AS, añade su propio número de AS al inicio de la lista. Si un router recibe un anuncio que ya contiene su propio número de AS en la ruta, detecta un bucle y descarta el paquete de inmediato.
- **Conexiones TCP:** Para garantizar un intercambio fiable y ocultar los detalles físicos de las redes internas de tránsito, los routers BGP vecinos se comunican estableciendo **conexiones TCP** directamente entre sí.
# Los dos tipos de BGP

El prptocolo opera en dos variantes según la ubicación de los routers que se comunican:

- **eBGP (BGP Externo):** Se ejecuta entre routers de frontera que pertenecen a **diferentes Sistemas Autónomos** para anunciar rutas utilizables a través de Internet.
- **iBGP (BGP Interno):** Se ejecuta entre los routers de frontera de un **mismo Sistema Autónomo** para propagar la información de las rutas externas aprendidas de forma consistente y unificada a todos los nodos del dominio.

# Acuerdos Comerciales y Políticas de Tránsito
BGP permite a las empresas y proveedores de servicios (ISPs) definir con precisión qué tráfico aceptan transportar y cuál rechazan:
- **Servicio de Tránsito:** Un ISP cliente paga a un ISP proveedor para que este transporte sus paquetes hacia cualquier destino de Internet. El cliente solo anuncia sus propias direcciones y el proveedor le anuncia todas las rutas de la red.
- **Peering (Interconexión sin liquidación):** Dos ISPs competidores acuerdan conectar sus redes para intercambiar tráfico de manera directa y gratuita, evitando pagar tarifas a un tercero. El peering **no es transitivo** (el AS3 no transportará tráfico del AS4 destinado al AS2 de manera gratuita a menos que haya un pago de por medio).
- **Multihoming:** Técnica en la que una red corporativa se conecta a múltiples ISPs para mejorar su fiabilidad, utilizando BGP para indicar activamente por qué enlace prefiere que entre o salga el tráfico.
# Algoritmo de Selección de Ruta
Cuando un router BGP conoce múltiples rutas alternativas hacia un mismo prefijo de destino, ejecuta en secuencia el siguiente algoritmo para elegir la mejor ruta:

1. **Preferencia Local (_Local Preference_):** Es un valor configurable por el operador que se mantiene interno al AS. Permite dar la máxima prioridad a rutas específicas (por ejemplo, priorizar rutas de clientes que pagan frente a rutas de proveedores a los que se debe pagar).
2. **Longitud de la ruta AS (_AS Path_ más corto):** En caso de empate, se prefiere la ruta que cruce el menor número de Sistemas Autónomos.
3. **Origen de la ruta (eBGP sobre iBGP):** Se prefieren las rutas aprendidas externamente a través de eBGP que las aprendidas internamente por iBGP (lo que acelera la salida del tráfico del propio dominio).
4. **Atributo MED (_Multi-Exit Discriminator_):** Para rutas del mismo AS vecino, se prefiere la que tenga el menor valor de MED.
5. **Costo IGP más corto (Ruta de la "Papa Caliente"):** Se elige el punto de salida físicamente más cercano según el protocolo de enrutamiento interno (IGP) para deshacerse del paquete lo antes posible y evitar consumir recursos internos del AS. Esto suele generar que las rutas de ida y vuelta en Internet sean asimétricas.
# Ingeniería de Tráfico en BGP
Los operadores utilizan técnicas indirectas para influir en cómo entra y sale el tráfico de su red:

- **Ingeniería de tráfico saliente:** Se controla fácilmente ajustando de forma directa la _Preferencia Local_ en los routers internos.
- **Ingeniería de tráfico entrante:** Es más compleja porque no se puede obligar directamente a otra red autónoma a elegir una ruta. Se usan estrategias indirectas como el **AS path prepending** (inflar artificialmente el camino repitiendo varias veces el número de AS propio en los anuncios de ruta para que parezca más largo y menos atractivo) o la **división de prefijos** para aprovechar la coincidencia de prefijo más largo (_longest prefix match_).

[[6 Multidifusion en Internet]]