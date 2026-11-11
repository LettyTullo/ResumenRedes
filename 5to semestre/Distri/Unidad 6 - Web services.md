#### ¿Qué son los Web Services?
Los **Web Services** (servicios web) son sistemas de software e interfaces de programación diseñados para permitir la interacción e interoperabilidad máquina a máquina a través de una red como Internet.

>[!info] Sus características principales son:
>- **Interoperabilidad estándar:** Ofrecen una forma estándar para que aplicaciones de software desarrolladas en diversas plataformas o _frameworks_ se comuniquen entre sí.
>- **Protocolos abiertos e independencia:** Son componentes de software independientes y autónomos que utilizan protocolos abiertos.
>- **Descripción de la interfaz:** Poseen una interfaz descrita en un formato procesable por máquina, específicamente mediante **WSDL** (_Web Services Description Language_).
> -  **Intercambio de mensajes:** Los clientes interactúan con los servicios mediante solicitudes y respuestas formateadas (habitualmente en **XML** usando mensajes **SOAP**, o mediante el enfoque **REST** manipulando recursos en XML/JSON), transmitidas típicamente a través del protocolo **HTTP/S**.

#### ¿Para qué se utilizan los Web Services?

La utilidad principal de los servicios web es facilitar la integración y comunicación entre diferentes sistemas distribuidos. Se utilizan principalmente para:

1. **Integración B2B y aplicaciones _mashup_:** Soportan la integración entre sistemas empresariales y permiten a los desarrolladores crear aplicaciones innovadoras combinando servicios existentes.
2. **Infraestructura para Cloud y Grid Computing:** Proporcionan el _middleware_ subyacente para entornos de computación en la nube y en malla.
3. **Casos de uso reales (destacados en la unidad):**
    - **Google Web API:** Permite a desarrolladores consultar información de miles de millones de documentos web usando SOAP y WSDL desde entornos como Java, PHP, Perl o .NET.
    - **Integración comercial y de transporte:** Caso de _Dollar Rent-a-Car_ enlazado con _Southwest Airlines_, permitiendo reservar vehículos desde la misma plataforma al comprar un pasaje aéreo.
    - **Amazon Web Services:** Permite a aplicaciones de terceros (vía SOAP o REST) gestionar inventarios y realizar pedidos automáticos de suministros.
    - **Consultas automatizadas:** Verificación de titulares de dominios (_NetworkSolutions_), consulta de precios de libros por ISBN (_Barnes and Noble_), o información geográfica y de tráfico (_MapPoint.Net_).
    - **Interoperabilidad en el sector público:** Sistemas gubernamentales de intercambio de datos (como el del MITIC) para consultar cédulas de identidad, actas de nacimiento, matrimonio, defunción y otros registros públicos en beneficio del ciudadano.

#### Diferencia entre URI, URL y URN

- **URI (_Uniform Resource Identifier_):** Es el **identificador de recurso general**. Un URI es la categoría global cuyo valor puede ser una URL o una URN.
- **URL (_Uniform Resource Locator_):** Es un tipo de URI que **incluye información de ubicación del recurso**, como por ejemplo el nombre de dominio del servidor (ejemplo: `https://www.paraguay.gov.py/`).
- **URN (_Uniform Resource Name_):** Es un nombre de recurso uniforme que **no depende de la ubicación**. En su lugar, requiere de un servicio de búsqueda para mapearlo o asignarlo a las URLs correspondientes del recurso (ejemplos: `urn:isbn:0451450523` o `urn:ISSN:0167-6423`).
#### Web Services SOAP

![[Pasted image 20261111124702.png|510]]

El diagrama adjunto describe el modelo conceptual estándar de arquitectura de servicios web del W3C y se articula directamente con los componentes técnicos de un servicio web SOAP.
- **1. Descubrimiento y conocimiento mutuo (_Parties "become known" to each other_)**
    - **En el diagrama:** La entidad cliente (_Requester Entity_) y la proveedora (_Provider Entity_) se reconocen mutuamente a nivel conceptual u organizacional.
    - **En la técnica SOAP:** Este paso se logra mediante **UDDI** (Universal Description, Discovery and Integration), un catálogo basado en XML donde los proveedores registran sus servicios para que los clientes puedan descubrirlos.
- **2. Acuerdo de semántica y contrato (_Agree on semantics & WSD_)**
    - **En el diagrama:** Las partes definen y acuerdan las reglas semánticas (_Sem_) y la descripción del servicio (_WSD - Web Services Description_).
    - **En la técnica SOAP:** Corresponde al archivo **WSDL** (Web Services Description Language), un documento XML que actúa como contrato público definiendo las operaciones disponibles, los formatos de los mensajes e interfaz de comunicación.
- **3. Configuración de los agentes de software (_Input Semantics & WSD_)**
    - **En el diagrama:** Las definiciones formales se introducen tanto en el agente del cliente (_Requester Agent_) como en el del servidor (_Provider Agent_).
    - **En la técnica SOAP:** El cliente utiliza el WSDL para instanciar un _Web Service Proxy_ (encargado de serializar las peticiones XML), mientras que el servidor configura su _Web Service Stub_ (encargado de recibir, deserializar y ejecutar la lógica correspondiente).
- **4. Interacción y ejecución (_Interact_)**
    - **En el diagrama:** Intercambio directo de mensajes entre el _Requester Agent_ y el _Provider Agent_.
    - **En la técnica SOAP:** El agente cliente empaqueta los datos dentro de un sobre XML con el formato **SOAP** (con encabezados y cuerpo) y lo transmite por la red mediante un protocolo de transporte (como HTTP/HTTPS, SMTP o FTP). El servidor procesa la solicitud (RPC) y devuelve la respuesta en otro mensaje XML/SOAP.
#### Cuáles son los protocolos y tecnologías asociados?
La arquitectura de los servicios web SOAP se fundamenta en un conjunto de tecnologías y protocolos abiertos:

- **SOAP (** **Simple Object Access Protocol** **):** Es el protocolo estándar de mensajería encargado de definir la estructura, empaquetado y reglas de intercambio de datos entre objetos distribuidos.
- **XML (** **Extensible Markup Language** **):** Es el lenguaje de etiquetas que proporciona el formato de datos estándar, flexible e intercambiable utilizado para construir el contenido de los mensajes SOAP. 
***- Ventajas de XML***
- Ofrece un formato de datos estándar, flexible y extensible.
- Reduce significativamente la carga de la implementación de las muchas tecnologías necesarias para garantizar el éxito de los servicios Web.
- El conjunto de información XML no es un formato de datos en sí, sino un conjunto formal de elementos de información y sus propiedades asociadas que conforman una descripción abstracta de un documento XML.
- **WSDL (** **Web Services Description Language** **):** Es el lenguaje de interfaz pública (IDL) que describe las capacidades funcionales y las especificaciones del servicio web. Describe servicios Web a partir de los mensajes que se intercambian entre los agentes del request (solicitante) y el provider (proveedor). El mensaje en sí se describe de forma abstracta y luego relacionado a un protocolo de red y formato de mensajes concreto. La aplicación puede ser implementada usando librerías como por ejemplo COM, JMS, CORBA, COBOL, etc. Mientras que el emisor y el receptor estén de acuerdo en la descripción del servicio (por ejemplo, el archivo WSDL), detrás de las implementaciones de los servicios Web pueden realizar las acciones que desean.
- **UDDI (** **Universal Description, Discovery and Integration** **):** Es el estándar basado en XML que sirve como catálogo o registro para la publicación y búsqueda de servicios web. Las empresas registras sus servicios. Dicho registro está basado en XML. La idea es una imputación en el registro de empresas UDDI, un servicio centralizado y replicado a distintos nodos en forma regular, quedando disponible para ser descubierta por otras empresas.
- **Protocolos de transporte y red:** Aunque típicamente los mensajes SOAP se transmiten a través de **HTTP** o **HTTPS**[5][11], SOAP es independiente de la capa de transporte y puede enviarse mediante **SMTP**, **FTP**, **RMI / IIOP** o sistemas propios de mensajería

![[Pasted image 20261111130027.png|429]]
El gráfico ilustra la arquitectura de ejecución (_runtime_) de la comunicación entre una aplicación cliente y un servidor mediante un servicio web SOAP, mostrando cómo se abstrae la complejidad de la red mediante componentes intermediarios.
- **Componentes desarrollados por el programador (Rosa)**  
    - **Client Code:** La aplicación o sistema cliente que requiere consumir una función remota.
    - **Web Service Implementation:** El código del servidor que contiene la lógica de negocio y realiza la tarea solicitada.
- **Componentes autogenerados (Celeste)**
    - **Web Service Proxy:** Objeto local en el cliente que simula ser el servicio remoto. Permite que el código cliente invoque métodos locales (_Method calls_) sin preocuparse por la red; el proxy toma esos datos y los empaqueta (serializa) en formato XML/SOAP.
    - **Web Service Stub:** Receptor en el servidor que recibe el mensaje SOAP enviado por la red, lo deserializa y llama a la implementación real del servicio web.
- **Mecanismo de transporte**
    - **SOAP Messages / Internet:** El proxy y el stub intercambian las peticiones y respuestas formateadas en XML utilizando un protocolo de red (como HTTP/HTTPS).
- **Conexión y rol de UDDI en el proceso**
    - **UDDI (Universal Description, Discovery and Integration)** es el estándar en XML —desarrollado por Ariba, Microsoft e IBM y adoptado por OASIS— que funciona como un registro centralizado y replicado entre nodos para publicar y descubrir servicios web.
    - **Cómo se conecta con el gráfico:** Antes de que ocurra la interacción del gráfico, la máquina cliente consulta el registro **UDDI** para encontrar el servicio web. Una vez descubierto, descarga el archivo de interfaz (WSDL), el cual se utiliza para autogenerar el **Web Service Proxy** (celeste) en la máquina cliente.
#### REST (Representational State Transfer)
**REST** es un enfoque o estilo de arquitectura propuesto por Roy Fielding (2000) como alternativa a los servicios web basados en SOAP.
#### **Principios y funcionamiento de REST:**
- **Enfoque centrado en recursos:** A diferencia de SOAP (que se enfoca en las interfaces de servicios y llamadas a operaciones), REST pone el énfasis en la **manipulación de recursos de datos**.
- **Representación del estado:** Los recursos se representan y transfieren habitualmente en formatos **XML o JSON**. Al consultar, el cliente recibe el estado completo del recurso en lugar de llamar a una operación para obtener solo una parte.
- **Direccionamiento único por URL:** Cada vez que se crea un nuevo recurso, se le asigna una nueva **URL** pública desde la cual se puede acceder o actualizar.
- **Uso de operaciones HTTP estándar:** Las aplicaciones interactúan con los recursos utilizando un conjunto básico y limitado de métodos HTTP:
    - **`GET`:** Solicita al servidor que busque y devuelva el recurso.
    - **`POST`:** Permite al servidor crear un nuevo recurso o entrada.
    - **`PUT`:** Indica al servidor que debe actualizar un recurso existente.
    - **`DELETE`:** Permite solicitar al servidor la eliminación de un recurso.
#### **Simplicidad y adopción:**
Fielding sostiene que en Internet un conjunto mínimo y uniforme de operaciones es más práctico que la proliferación de múltiples interfaces de servicio específicas. Como evidencia de su adopción masiva, se registró que el **80% de las solicitudes a los servicios web de Amazon.com se realizaban a través de REST**, mientras que solo el 20% restante utilizaba SOAP.
#### WebSocket
**WebSocket** es una tecnología y protocolo (definido en la RFC 6455) diseñado para establecer una **comunicación bidireccional y en tiempo real** entre un cliente (como un navegador ejecutando código en un entorno controlado) y un servidor o host remoto.
#### **Funcionamiento y características clave:**
- **Comunicación bidireccional (Full-Duplex):** Permite que tanto el cliente como el servidor se envíen datos de manera simultánea e interactiva sobre la misma conexión.
- **Superación de las limitaciones de HTTP:** Su objetivo principal es permitir la comunicación continua **sin necesidad de abrir múltiples conexiones HTTP** ni recurrir a técnicas pesadas como sondeos largos (_long polling_), `XMLHttpRequest` o marcos `<iframe>`.
- **Arquitectura sobre TCP:** Se distribuye directamente en capas sobre **TCP**. Inicia con un **protocolo de enlace de apertura** (_handshake_) y prosigue con un entramado de mensajes básico (_message framing_).
- **Modelo de seguridad:** Utiliza el modelo de seguridad basado en origen (_Origin-based security model_), estándar habitual en los navegadores web.