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
# Seccion 7.3.2 Objetos web estaticos

#### 1. Definición de Objetos Web Estáticos
- **Concepto:** Son archivos almacenados en un servidor web que se presentan exactamente de la misma manera cada vez que son recuperados y visualizados por un cliente.
- **Ejemplos comunes:** Incluyen el logotipo de un sitio, las hojas de estilo, los encabezados y pies de página (_headers/footers_), imágenes, gráficos, fuentes tipográficas, páginas de inicio en HTML y archivos de vídeo o audio precodificados.
- **Presencia en la Web moderna:** Aunque casi cualquier página actual incluye contenido dinámico, una proporción significativa de los elementos visuales de una página sigue siendo estática. Cabe destacar que el carácter "estático" no significa que el elemento sea inerte dentro del navegador; por ejemplo, un archivo de vídeo grabado es técnicamente un objeto estático.
#### Características Principales y Almacenamiento en Caché

- **Consistencia:** Mantienen el mismo contenido y apariencia en cada descarga.
- **Capacidad de almacenamiento en caché (_Cacheables_):** Dado que su contenido no varía frecuentemente, son idóneos para ser almacenados en la memoria caché local del navegador, en servidores proxy intermedios o en Redes de Distribución de Contenidos (CDNs).
- **Reducción de latencia y tráfico:** Al ser servidos desde ubicaciones u objetos en caché cercanos al usuario (_edge nodes_), el almacenamiento de objetos estáticos disminuye drásticamente el tráfico en la red, reduce la latencia de carga y ahorra ancho de banda en el servidor de origen.
#### 3. HTML (_HyperText Markup Language_)
- **Lenguaje estándar (_Lingua Franca_):** Es el lenguaje de marcado fundamental sobre el cual se estructuran las páginas web.
- **Etiquetas de marcado:** Combina el contenido del documento con etiquetas o comandos de marcado explícitos que indican cómo estructurar el texto. Por ejemplo:
    - `<b>...</b>`: Inicia y finaliza el texto en negrita.
    - `<h1>...</h1>`: Define un encabezado principal de nivel 1.
- **Separación de contenido y formato:** La gran ventaja de un lenguaje de marcado (como HTML, TeX o LaTeX) frente a textos no marcados es que **separa el contenido estructural de la forma de presentación**. A diferencia de procesadores de texto como Microsoft Word, en HTML las órdenes de formato están explícitamente incrustadas dentro del propio código de texto.
#### 4. CSS (_Cascading Style Sheets_)

- **Rol de la presentación visual:** Es el lenguaje de hojas de estilo encargado de controlar la **apariencia visual y formato** de los elementos de la página.
- **Propiedades controladas:** CSS define atributos como fuentes tipográficas, colores, tamaños de texto, márgenes, relleno (_padding_), espaciados y diagramación de tablas, listas y contenedores.
- **Ventaja de la separación:** Al aislar las reglas de estilo (CSS) de la estructura del documento (HTML), se facilita el mantenimiento visual global de un sitio web y su coherencia gráfica.
#### 5. Adaptabilidad a Dispositivos y Renderizado

- **Interpretación por el navegador:** Escribir un navegador implica implementar un intérprete que aplique los comandos de marcado (HTML) y las hojas de estilo (CSS) sobre el contenido para renderizarlo en pantalla.
- **Responsividad ante pantallas heterogéneas:** Estandarizar estos comandos permite que una página diseñada en un monitor de alta resolución (por ejemplo, 3840×2160 píxeles) pueda ser leída y reformateada por el navegador para visualizarse en la pantalla pequeña de un teléfono móvil (640×320 píxeles) sin forzar una reducción escalar simple que haría el texto ilegible.
- **Herramientas de creación:** Los objetos estáticos en HTML y CSS se pueden crear manualmente mediante editores de texto plano o utilizando programas de diseño de páginas web como Adobe Dreamweaver.
# 7.3.3 Páginas y aplicaciones web dinámicos

#### 1. Evolución: De la Web Estática a las Aplicaciones Web
- **Web Estática tradicional:** Trataba a las páginas como simples documentos multimedia enlazados entre sí que el servidor leía del disco y entregaba de forma idéntica a todos los visitantes.
- **Aplicaciones Web modernas:** Los sitios funcionan como auténticas aplicaciones de software dentro del navegador (ejemplos: correo electrónico web, comercio electrónico, catálogos interactivos, servicios de mapas y herramientas de colaboración).
- **Computación en la nube (_Cloud Computing_):** Representa una forma dominante de computación donde el procesamiento de la información se traslada desde las computadoras individuales hacia _clusters_ compartidos de servidores en centros de datos distribuidos.
- **Ventajas operativas:** Los usuarios no necesitan instalar programas locales; los datos permanecen accesibles desde cualquier ordenador o dispositivo, y el operador del servicio se encarga de las copias de seguridad y actualizaciones de manera automática.
#### 2. La Necesidad de Contenido Dinámico
Para actuar como aplicaciones interactivas, las páginas no pueden ser estáticas. Requieren **contenido dinámico**, el cual se genera a petición del usuario y puede ser producido por programas que se ejecutan en el **servidor**, en el **navegador (cliente)**, o combinando ambos entornos.
#### 3. Generación Dinámica en el Servidor (_Server-Side_)
Se utiliza cuando el usuario envía información (por ejemplo, al completar un formulario de compra). La URL enviada identifica al programa que debe ejecutarse en el servidor, utilizando los datos proporcionados como entrada para consultar una base de datos y construir dinámicamente el documento HTML que se devolverá al cliente.
Las dos interfaces y tecnologías estandarizadas en el servidor descritas en el material son:
- **CGI (_Common Gateway Interface_, RFC 3875):** Es el método clásico disponible desde los inicios de la Web. Proporciona una interfaz para que el servidor web invoque programas o scripts externos (escritos en Python, Perl, Ruby, etc.) que residen convencionalmente en un directorio denominado `/cgi-bin`. El servidor ejecuta cada solicitud a este directorio iniciando un proceso separado en el sistema operativo y entrega el HTML generado como respuesta al cliente.
- **PHP (_Hypertext Preprocessor_):** A diferencia de CGI, PHP permite incrustar código de scripting directamente dentro de las páginas HTML (identificadas por la extensión `.php`). El servidor web interpreta las instrucciones en PHP y genera la salida HTML final. Es de código abierto, muy extendido, fácil de integrar con bases de datos y diseñado para funcionar de manera óptima con servidores web como Apache.
#### 4. Generación Dinámica en el Cliente (_Client-Side_)
Los scripts del servidor (como PHP o CGI) pueden gestionar bases de datos, pero **no pueden responder a eventos en tiempo real ni a los movimientos del ratón** sin realizar un viaje completo de red de ida y vuelta al servidor. Para lograr interacción inmediata, el código debe ejecutarse directamente en la máquina cliente.

- **Etiqueta `<script>` y HTML5:** A partir de HTML 4.0 se introdujo la etiqueta `<script>` para incrustar código ejecutable en el navegador. HTML5 añadió elementos multimedia y gráficos nativos como `<video>`, `<audio>` y **`<canvas>`** (diseñado para la representación dinámica de formas bidimensionales e imágenes de mapa de bits).
    - _Consideración de Privacidad del `<canvas>`:_ Las propiedades de renderizado del elemento `<canvas>` suelen ser únicas en cada dispositivo, lo que permite a las redes de seguimiento generar una "huella digital" (_canvas fingerprinting_) para rastrear a los usuarios incluso si borran sus cookies.
- **JavaScript:** Es el lenguaje de scripting de muy alto nivel dominante para el lado del cliente. Permite gestionar eventos de la interfaz, validar formularios, realizar animaciones y actualizar contenidos en tiempo real al instante sin contacto previo con el servidor.
####Q
#### 5. Ejemplo de Integración: Servicio de Mapas en Línea (Flujo en 7 Pasos)

El material ilustra la interacción entre ambos modelos mediante un servicio interactivo de mapas:

1. **Petición del usuario:** El navegador envía una solicitud con los parámetros o la dirección buscada.
2. **Procesamiento en Servidor:** Un programa en el servidor recibe la petición y consulta la base de datos geográfica.
3. **Respuesta inicial:** El servidor devuelve la página principal (HTML con enlaces a fuentes, CSS y scripts).
4. **Interacción en Cliente:** El programa en el cliente (JavaScript) permite al usuario desplazarse, cambiar el zoom o filtrar la vista localmente.
5. **Solicitud en segundo plano (AJAX / Fetch):** Si el usuario se mueve a una región no cargada, el programa del cliente pide más datos al servidor en segundo plano sin cambiar la URL ni recargar la página entera.
6. **Consulta secundaria en Servidor:** El servidor recupera los datos adicionales de la base de datos.
7. **Actualización:** El servidor responde y el programa cliente redibuja la vista de inmediato (volviendo al paso 4).
![[Pasted image 20261030230658.png]]
#### 6. Cuadro Comparativo: Servidor (PHP) vs. Cliente (JavaScript)

|Característica|Procesamiento en Servidor (ej. PHP)|Procesamiento en Cliente (ej. JavaScript)|
|:--|:--|:--|
|**Ubicación de ejecución**|Se ejecuta en el servidor Web.|Se ejecuta en el navegador del usuario.|
|**Flujo operativo**|Envía datos al servidor \(\rightarrow\) ejecuta script \(\rightarrow\) genera y devuelve un nuevo HTML.|Ejecuta una función local ante un evento \(\rightarrow\) actualiza la interfaz al instante.|
|**Latencia**|Puede presentar demoras de varios segundos por el viaje de red.|Respuesta prácticamente instantánea sin retardo de red.|
|**Caso de uso ideal**|Consultas a bases de datos y procesamiento seguro de lógica de negocio o transacciones.|Interacción inmediata de la interfaz de usuario, eventos del ratón y animaciones.|
