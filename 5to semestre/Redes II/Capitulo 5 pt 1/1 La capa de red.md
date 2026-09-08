>[!example] Funciones principales de la capa de red
>- **Enrutamiento de paquetes:** Su función fundamental es determinar la ruta que deben seguir los paquetes desde la máquina de origen hasta la de destino. Para ello, debe conocer la **topología de la red** (enrutadores y enlaces) y calcular las rutas óptimas.
>- **Interconexión de redes (Internetworking):** Permite la comunicación entre dispositivos situados en redes o subredes autónomas distintas, lidiando con la heterogeneidad de las tecnologías de red.
>- **Gestión del tráfico y congestión:** Debe evitar sobrecargar ciertas líneas o enrutadores mientras otros permanecen ociosos. Cuando hay demasiados paquetes en una parte de la red, se produce congestión, y la capa de red debe aplicar mecanismos para mitigarla.
>- **Garantía de Calidad de Servicio (QoS):** Gestiona parámetros como el ancho de banda, el retardo, la fluctuación (jitter) y la pérdida de paquetes para satisfacer los requisitos de las aplicaciones.
>- **Direccionamiento uniforme:** Proporciona a la capa de transporte un plan de numeración uniforme para identificar dispositivos, incluso si se atraviesan diferentes tipos de redes (LAN o WAN)

## Aspectos de diseño 

# 1. Conmutación de paquetes de almacenamiento y reenvío (Store-and-Forward):
Un host envía un paquete al router más cercano, el cual lo almacena hasta recibirlo por completo y verificar su suma de comprobación (control de errores); solo entonces lo reenvía al siguiente salto.

![[Screenshot 2026-08-10 203803.png|470]]


>[!danger] Explicacion
> **Hosts (H1 y H2) y Procesos (P1 y P2):** El origen y el destino de la comunicación. El proceso P1 en el Host H1 fragmenta la información en bloques independientes llamados **paquetes** para enviarlos al proceso P2 en el Host H2.
> - **Equipos del ISP (_ISP's equipment_):** Es la subred del proveedor de servicios de Internet, compuesta por los routers internos (A, B, C, D y E) interconectados por enlaces de comunicación.
> - **Router de acceso (F) y LAN:** El router F actúa como la pasarela local que entrega los paquetes a la red de área local (LAN) donde reside H2.

# 2. **Servicios proporcionados a la capa de transporte**:

>[!Rosado]- Servicio sin Conexión (Red de Datagramas)
En este modelo, inspirado en el sistema de telegramas, cada paquete se inyecta en la red de forma individual
>- **Funcionamiento:** Cada paquete (datagrama) lleva la **dirección de destino completa**.
>- **Enrutamiento:** Los routers deciden de forma independiente el camino para cada paquete, lo que significa que paquetes de un mismo mensaje podrían seguir rutas distintas y llegar en desorden.
>- **Configuración:** No se requiere ninguna configuración previa antes de enviar los datos.
>- **Ejemplo:** El protocolo **IP**, que es la base de Internet, funciona bajo este esquema.

![[Screenshot 2026-08-10 205108.png|558]]

>[!info] Explicacion
>**Estructura de la tabla**
>- **Dest (Destino):** La columna izquierda enumera todos los posibles routers o nodos de destino final en la red.
>- **Line (Línea de salida):** La columna derecha indica la interfaz física (o el router vecino directo) por la cual se debe despachar el paquete para alcanzar dicho destino.
>
**Cómo lee la tabla el Router A**
>1. **Consulta del destino:** Cuando H1 envía paquetes hacia H2 (conectado a F), el Router A revisa la cabecera del paquete y busca la letra **F** en la columna **Dest**.
>2. **Reenvío inicial:** En la tabla inicial (**A's table initially**), la fila **F** apunta a la línea **C**. Por esto, los paquetes 1, 2 y 3 son enviados a través del enlace hacia el Router C.
>3. **Decisión local en cada salto:** Al recibir el paquete, el Router C consulta su propia tabla (**C's Table**). Para el destino **F**, su línea es **E**. El paquete 1, 2 y 3 sigue la ruta A $\rightarrow$ C $\rightarrow$ E $\rightarrow$ F.

>[!tip] Servicio Orientado a la Conexión (Red de Circuitos Virtuales - CV)
 Este modelo imita el funcionamiento del sistema telefónico tradicional.
>- **Funcionamiento:** Antes de enviar datos, se establece una ruta fija desde el router de origen hasta el de destino, denominada **Circuito Virtual (CV)**.
 >- **Identificadores:** Los paquetes no llevan la dirección completa, sino un **identificador o etiqueta corta** que indica a qué CV pertenecen.
>- **Enrutamiento:** La decisión de ruta se toma una sola vez al establecer la conexión; todos los paquetes subsiguientes siguen esa misma trayectoria.
>- **Ejemplo:** Tecnologías como ATM, Frame Relay y **MPLS** utilizan este enfoque

![[Screenshot 2026-08-10 210650.png|461]]

>[!info] Explicacion
>**Estructura de las tablas de conmutación**
Las tablas ya no buscan destinos globales, sino coincidencias exactas entre la entrada y la salida:
>- **Entrada (`In`):** Identifica la interfaz física por donde entra el paquete (**Line**) y su número de etiqueta actual (**Tag**).
>- **Salida (`Out`):** Define hacia qué interfaz sale el paquete (**Line**) y con qué nueva etiqueta se reescribe (**Tag**).
**Cómo se leen las tablas (Paso a paso)**
>1. **Evitar colisiones en el Router A:**
  >- **Host H1** envía un paquete etiquetado como `1` por la línea `H1`. El Router A lo mapea a la salida línea `C` con etiqueta `1`.
>- **Host H3** envía _también_ un paquete etiquetado como `1` por su línea `H3`. Como el enlace A-C no puede tener dos circuitos usando la misma etiqueta `1`, el Router A la cambia a **etiqueta `2`** al reenviarlo por la línea `C`.
>1. **Tránsito en el Router C:**
>- El paquete del circuito de H1 llega desde `A` con `Tag 1` $\rightarrow$ C lo despacha hacia la línea `E` con `Tag 1`.
>- El paquete del circuito de H3 llega desde `A` con `Tag 2` $\rightarrow$ C lo despacha hacia la línea `E` con `Tag 2`.      
>3. **Entrega final en el Router E:**
>- Ambas conexiones llegan al Router E y este las reenvía al Router F (línea `F`), entregando los paquetes en orden hacia el Host H2.

# Comparación entre Datagramas y Circuitos Virtuales

| Aspecto                         | Red de Datagramas (Sin conexión)                                     | Red de Circuitos Virtuales (Con conexión)                                       |
| ------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Configuración**               | **No necesaria.** Se envían datos de inmediato.                      | **Requerida.** Toma tiempo establecer el circuito antes de enviar datos.        |
| **Direccionamiento**            | Cada paquete contiene la dirección completa de origen y destino.     | Cada paquete contiene un **número de CV corto**.                                |
| **Información de Estado**       | Los routers no conservan información de estado sobre las conexiones. | Cada CV requiere espacio en las tablas del router para cada conexión.           |
| **Enrutamiento**                | Cada paquete se enruta de forma independiente.                       | La ruta se elige al inicio; todos los paquetes la siguen.                       |
| **Efecto de fallas del router** | Ninguno, excepto por los paquetes perdidos durante la caída.         | **Fatal.** Terminan todos los CV que pasaban por el router defectuoso.          |
| **Calidad de Servicio (QoS)**   | **Difícil** de garantizar de forma estricta.                         | **Fácil**, si se pueden asignar recursos (ancho de banda, buffers) de antemano. |
| **Control de Congestión**       | Difícil de implementar proactivamente.                               | Fácil mediante el **control de admisión** al momento de crear el circuito.      |
[[2 Algoritmos de enrutamiento pt1]]