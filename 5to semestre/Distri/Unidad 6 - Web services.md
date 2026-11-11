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
