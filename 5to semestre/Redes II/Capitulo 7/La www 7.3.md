# Sección 7.3: La World Wide Web (La Red Mundial)

* **Definición y Propósito:** La World Wide Web es un marco arquitectónico diseñado para acceder a contenidos enlazados y distribuidos en millones de computadoras a través de Internet.
* **Origen e Historia:**
  * **1989:** Tim Berners-Lee propone en el CERN un sistema de documentos enlazados para facilitar la colaboración entre equipos internacionales de física de altas energías.
  * **1991:** Se realiza la primera demostración pública en la conferencia *Hypertext '91*.
  * **1993-1994:** En febrero de 1993, Marc Andreessen desarrolla el navegador gráfico Mosaic y en 1994 funda *Netscape Communications Corp.*, desencadenando la primera "guerra de navegadores" frente a Microsoft Internet Explorer.
  * **1994 (Creación del W3C):** El CERN y el M.I.T. fundan el **W3C (*World Wide Web Consortium*)**, organización dirigida por Berners-Lee dedicada a estandarizar los protocolos e impulsar la interoperabilidad (`www.w3.org`).
  * **Años 90 - 2000:** Expansión masiva de sitios web que desemboca en la era *dot-com*.
#### Sección 7.3.1: Panorama Arquitectónico (Arquitectura de la Web)

Esta sección desglosa cómo se estructuran la visualización, los recursos y la transferencia de datos entre el cliente (navegador) y el servidor:

#### 1. Conceptos Fundamentales
* **El Hipertexto y los Hipervínculos:** El concepto de **hipertexto** fue acuñado en 1945 por **Vannevar Bush** (mucho antes del desarrollo de Internet y las computadoras comerciales). Consiste en permitir que un documento apunte a otro mediante **hipervínculos** (elementos de texto, imágenes o íconos). Al hacer clic sobre ellos, la navegación entre servidores distribuidos globalmente se ejecuta de forma totalmente transparente y fluida para el usuario.
* **El Navegador Web (*Client / Browser*):** Es el programa intérprete (ej. Chrome, Firefox, Safari, Edge, Opera, Brave) que solicita los recursos a la red, procesa sus instrucciones de marcado y renderiza la página en pantalla.
* **Páginas Estáticas vs. Dinámicas:**
  * **Estáticas:** Archivos precargados en el servidor que se entregan idénticos en cada consulta.
  * **Dinámicas:** Contenidos generados a petición mediante la ejecución de un programa en el servidor (ej. consulta a base de datos) o de código ejecutado en el propio navegador del cliente.
#### 2. Identificador Universal de Recursos (URL) y Protocolos
* **Estructura de la URL (*Uniform Resource Locator*):** Define la ubicación de cualquier objeto en la red mediante tres partes principales:
  1. **Protocolo:** Mecanismo de transferencia (ej. `https`).
  2. **Nombre DNS del host:** Identificador del servidor (ej. `fcc.gov`).
  3. **Ruta del archivo:** Camino interno hacia el objeto dentro del servidor (ej. `/`).
* **Esquemas de Protocolos soportados en URLs:**
  * `http` / `https`: Transferencia de hipertexto estándar y segura cifrada con TLS.
  * `ftp`: Acceso a servidores de archivos mediante el protocolo FTP.
  * `file`: Lectura de archivos locales en el equipo cliente sin requerir servidor.
  * `mailto`: Apertura directa del agente de correo para enviar un e-mail.
  * `rtsp` / `sip`: Transmisión de datos multimedia en tiempo real y llamadas de voz sobre IP.
  * `about`: Diagnósticos e información interna del navegador.

#### 3. Flujo Detallado de Resolución de un Hipervínculo
Cuando un usuario hace clic sobre una URL (por ejemplo, `https://fcc.gov/`), el navegador ejecuta secuencialmente los siguientes pasos:

1. El navegador analiza e interpreta la URL seleccionada.
2. Consulta al servidor DNS la dirección IP asociada al nombre del host (`fcc.gov`).
3. El servidor DNS devuelve la dirección IP numérica.
4. El navegador inicia una conexión TCP hacia el puerto 80 (HTTP) o 443 (HTTPS) de esa IP.
5. Envía la solicitud HTTP/HTTPS especificando la ruta elegida (ej. `GET /index.html HTTP/1.1`).
6. El servidor procesa la solicitud y devuelve el documento principal (`index.html`).
7. El navegador analiza la estructura del documento devuelto; si identifica referencias a objetos incrustados necesarios para la visualización (imágenes, fuentes en `gstatic.com`, scripts de análisis en `google-analytics.com`), repite el proceso para obtener cada recurso.
8. El navegador ensambla y muestra la página web completa en la pantalla.
9. Transcurrido un breve periodo de inactividad, las conexiones TCP se liberan.
#### Protocolos en el URL

![[Pasted image 20261030225355.png]]
#### El Lado del Servidor: Arquitectura, Problemas y Soluciones
El servidor web tiene como función aceptar conexiones, procesar la ruta solicitada y devolver el contenido correspondiente.
##### A. Servidor Web Simple (Mono-hilo) y sus Limitaciones
En un diseño elemental, el servidor opera en un bucle secuencial:
1. Acepta la conexión TCP del cliente
2. Obtiene la ruta del archivo solicitado.
3. Busca el archivo en el disco (o ejecuta un programa)
4. Envía el contenido al cliente
5. Libera la conexión TCP
6. **Problemas y Limitaciones:**
    - **Cuello de botella en disco:** La lectura de archivos en disco es muy lenta comparada con la ejecución en memoria
    - **Procesamiento secuencial:** Solo atiende una petición a la vez; si un archivo es grande o la lectura se retrasa, todas las demás solicitudes entrantes quedan bloqueadas

#### B. Solución: Servidor Web Multihilo
Para resolver los bloqueos por E/S y atender miles de peticiones por segundo, la arquitectura migra hacia un **servidor multihilo** compuesto por $k+1$ hilos en un mismo proceso:
- **Módulo Front-End (1 hilo):** Acepta todas las solicitudes entrantes y construye un registro corto descriptivo.
- **Módulos de Procesamiento (**$k$ **hilos):** Procesan las peticiones en paralelo entregadas por el front-end.
- **Caché compartida en memoria:** Todos los hilos comparten el espacio de direcciones de memoria donde residen los archivos leídos recientemente.
**Flujo de ejecución:**

1. El hilo de procesamiento revisa si el objeto solicitado se encuentra en la **caché compartida**.
2. Si está presente, actualiza el registro con un puntero al archivo y lo entrega directamente al cliente desde la memoria.
3. Si no está en caché, inicia la lectura de disco para cargarlo en la memoria y servirlo al cliente,
4. **Ventajas:** Si un módulo se bloquea esperando datos de disco o red, los demás hilos continúan procesando otras peticiones activamente1920. Con $k$ módulos, el rendimiento del servidor puede multiplicarse hasta $k$ **veces** respecto al modelo mono-hilo.

#### C. Arquitecturas Modernas: Front-End / Back-End y Granjas de Servidores

- **División Front-End (Proxy Inverso) y Back-End:** En sistemas modernos, el front-end actúa como un **proxy inverso** que entrega de inmediato objetos estáticos en caché al navegador, mientras el **back-end** ejecuta tareas complejas como consultas a bases de datos o lógica de negocio dinámica.
- **Granjas de Servidores y Balanceo de Carga:** Cuando un solo equipo físico no basta, se utiliza un clúster de máquinas (granja de servidores) respaldadas por una base de datos común2223. Un conmutador/enrutador en el front-end o un mapeo por DNS distribuye el tráfico entre los servidores aplicando políticas de equilibrio como _Round-Robin_ o asignación por IP de origen

![[Pasted image 20261030225739.png]]
#### 3. Ventajas y Soluciones de Rendimiento en la Red
- **Almacenamiento en Caché y Proxies:** El navegador y los servidores proxy intermedios guardan copias locales de recursos frecuentes para reducir la latencia, ahorrar ancho de banda y aliviar la carga de la red.
- **Redes de Distribución de Contenido (CDNs):** En lugar de concentrar las respuestas en una única ubicación, las CDNs replican objetos en nodos perimetrales (_edge_) distribuidos geográficamente cerca del usuario final, optimizando los tiempos de RTT y absorbiendo posibles ataques masivos DDoS.