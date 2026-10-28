# Capítulo 7: La Capa de Aplicación (Visión General)
La **Capa de Aplicación** es el nivel superior de la arquitectura de red donde residen las aplicaciones reales con las que interactúan los usuarios. 
Incluso en este nivel, existen **protocolos de apoyo** fundamentales que permiten el funcionamiento global de las aplicaciones. El más importante es el **Sistema de Nombres de Dominio (DNS)**, el cual asigna nombres de Internet a direcciones IP. A partir de este servicio de soporte se estructuran las aplicaciones reales de la red, como el correo electrónico, la _World Wide Web_ (WWW), la transmisión de multimedia (_video streaming_) y las redes de distribución de contenidos (CDNs y P2P).
# Sección 7.1: El Sistema de Nombres de Dominio (DNS)

El **DNS** es un esquema de nombres jerárquico implementado sobre una base de datos distribuida. Está especificado principalmente en las RFC 1034, RFC 1035 y RFC 2181.
#### 7.1.1 Historia y Visión General
- **Contexto histórico:** Antes de 1983, la red ARPANET mantenía un archivo de texto centralizado denominado `hosts.txt` que contenía el listado de todos los nombres de las máquinas con sus correspondientes direcciones IP. Cada noche, los _hosts_ de la red descargaban una copia actualizada de este archivo. Sin embargo, el crecimiento exponencial de la red provocó la saturación del sistema, retrasos en las descargas y frecuentes conflictos de nombres al carecer de una administración descentralizada.
- **Creación y propósito:** En 1983 se creó el DNS para resolver estos problemas. Su función principal es **traducir nombres de dominio legibles por humanos a direcciones IP numéricas**.
- **Ventajas operativas:**
    1. Desvincula el nombre del servidor de su ubicación física o dirección IP, lo que permite cambiar de servidor o de proveedor sin alterar el nombre del dominio.
    2. Facilita el **equilibrio de carga** (_load balancing_) al mapear un único nombre de dominio hacia múltiples direcciones IP pertenecientes a _clusters_ de servidores distribuidos.
#### 7.1.2 El Proceso de Búsqueda DNS
El proceso de resolución abarca la interacción entre tres componentes clave:

1. **Stub Resolver:** Es un procedimiento de biblioteca de la pila del sistema operativo (como `gethostbyname`) invocado por la aplicación del usuario pasándole el nombre de dominio como parámetro.
2. **Local Recursive Resolver (Resolver Local):** Es el servidor que recibe la consulta del _stub resolver_ y se encarga de gestionar la búsqueda completa a través de la jerarquía de la red hasta obtener la respuesta final.
3. **Servidor de Nombres Autoritativo:** Es el servidor responsable de una porción específica de la jerarquía que almacena los registros reales y definitivos de esa zona. Por definición, su respuesta siempre es fidedigna y correcta.

```
[Aplicación] ---> (Stub Resolver)
                       |  (Consulta Recursiva sobre UDP)
                       v
            [Local Recursive Resolver]
                       |
                       +---> Consultas Iterativas a:
                       |     1. Servidor Raíz (.)
                       |     2. Servidor TLD (.edu)
                       |     3. Servidor Autoritativo (uchicago.edu)
                       v
            [Dirección IP entregada a la Aplicación]
```

- **Flujo y tipos de consultas:**
    - **Consulta Recursiva:** El _stub resolver_ envía una consulta recursiva al _local resolver_, solicitando que resuelva el nombre por completo y le entregue la respuesta definitiva sin respuestas parciales.
    - **Consultas Iterativas:** El _local resolver_ realiza una serie de consultas iterativas hacia los servidores jerárquicos (Raíz $(\rightarrow)$ TLD $(\rightarrow)$ Autoritativo). Cada servidor de la cadena devuelve una referencia hacia el siguiente nivel en lugar de realizar él mismo la búsqueda.
    - **Identificador de transacción:** Cada consulta lleva un identificador de 16 bits que el servidor copia en la respuesta para permitir al resolutor asociar las respuestas con sus peticiones pendientes.
    - **Transporte y tolerancia a fallos:** Las consultas y respuestas se transmiten mediante paquetes **UDP** (puerto 53). Si el _stub resolver_ no recibe respuesta dentro de un tiempo de espera (_timeout_), reintenta la solicitud dirigiéndose a otro servidor para tolerar caídas o pérdidas de paquetes.
- **Caché y Tiempo de Vida (TTL):**
    - Para evitar repetir todo el proceso iterativo en cada consulta, tanto el _local resolver_ como el _stub resolver_ guardan las respuestas obtenidas en una memoria caché local.
    - Cada registro incluye un campo **Time-To-Live (TTL)** que indica la cantidad de segundos que esa información puede ser conservada en caché antes de ser descartada.
- **Seguridad y Privacidad en la Búsqueda:**
    - **Minimización de QNAME:** Para proteger la privacidad del usuario y evitar revelar el nombre completo consultado (_FQDN_) Fully Qualified Domain Name, a todos los servidores de la cadena, el _local resolver_ envía a cada nivel jerárquico únicamente la etiqueta del nombre estrictamente necesaria para continuar la delegación (por ejemplo, envía solo `uchicago.edu` al servidor del TLD `.edu`).
    - **Codificación 0x20:** Dado que los nombres en las consultas no distinguen entre mayúsculas y minúsculas, este mecanismo altera arbitrariamente las mayúsculas y minúsculas del nombre consultado (ej. `uCHicaGO.EDu`) para inyectar bits de entropía adicionales al ID de transacción de 16 bits, dificultando ataques de envenenamiento de caché como el ataque Kaminsky.
#### 7.1.3 El Espacio de Nombres DNS y su Jerarquía

El espacio de nombres está organizado como una estructura jerárquica en árbol, análoga al sistema postal:

- **Dominio Raíz:** Es el nodo superior del árbol y se representa formalmente por un punto (`.`).
- **FQDN (_Fully Qualified Domain Name_):** Es el nombre de dominio completo que inicia con la máquina específica y recorre la jerarquía hasta concluir formalmente en el punto raíz (por ejemplo, `www.cs.uchicago.edu.`).
- **Dominios de Nivel Superior (TLDs - _Top-Level Domains_):** Administrados a nivel global por la **ICANN**. Se dividen en dos categorías principales:
    1. **gTLDs (genéricos):** `.com`, `.edu`, `.org`, `.net`, `.gov`, `.biz`, entre otros.
    2. **ccTLDs (por país):** Códigos geográficos de dos letras basados en la norma ISO 3166 (ej. `.py`, `.us`, `.uk`, `.nl`).
- **Dominios de Segundo Nivel y Subdominios:** Constituyen las ramas intermedias y hojas del árbol. Por ejemplo, en el nombre `cs.uchicago.edu`, `uchicago` es el dominio de segundo nivel registrado bajo `.edu`, mientras que `cs` es un subdominio administrado internamente por el departamento de informática de esa universidad.
- **Delegación y límites:** Cada dominio tiene autoridad para administrar y delegar subdominios por debajo de él (ejemplo: `.edu.py` bajo el ccTLD `.py`). Una vez registrado un dominio de segundo nivel, la organización propietaria puede crear nuevos subdominios sin solicitar permisos adicionales. Los nombres siguen divisiones organizativas y no coinciden necesariamente con las fronteras de las redes físicas o LANs.

![[Pasted image 20261025124343.png|528]]
##### Cadena de Administración del Árbol
La administración del espacio de nombres se distribuye en una cadena de tres niveles jerárquicos:
1. **ICANN (_Internet Corporation for Assigned Names and Numbers_):** Organización internacional que supervisa el sistema global de nombres y delega la gestión de cada TLD.
2. **Registry (Registro):** Entidad que administra un TLD específico. Por ejemplo, _Verisign_ administra el TLD `.com`, mientras que el _NIC Paraguay_ administra el ccTLD `.py`.
3. **Registrar (Registrador Acreditado):** Empresas autorizadas por ICANN o por el _Registry_ de un país para comercializar y registrar dominios de segundo nivel directamente a los usuarios (ej. GoDaddy, Namecheap).
##### Aspectos Clave del Árbol de Dominios
El funcionamiento operativo del árbol se rige por varios principios fundamentales:
- **Nombres Absolutos vs. Relativos:**
    - **Nombre de Dominio Absoluto / FQDN (_Fully Qualified Domain Name_):** Especifica la ruta completa hasta la raíz y termina técnicamente con un punto (ej. `eng.cisco.com.`).
    - **Nombre Relativo:** No incluye el punto final y requiere ser interpretado dentro de un contexto local para determinar su significado unívoco.
- **Inserción en Ramas Genéricas o de País:** Los dominios pueden integrarse bajo la rama genérica o de país. Por ejemplo, en Paraguay existen dominios de segundo nivel espejo como `edu.py` o `com.py` bajo el TLD `.py`.
- **Presencia en Múltiples TLDs:** No existe restricción para que una misma entidad registre el mismo nombre en distintos TLDs para proteger su marca o presencia global (ej. `sony.com`, `sony.net` y `sony.nl`).
- **Regla de Delegación y Autonomía:**
    - Para crear un nuevo dominio, se debe solicitar permiso al administrador del dominio superior directo (ej. solicitar `unsd.edu` al gestor del dominio `.edu`).
    - Una vez registrado el dominio, la entidad obtiene **autonomía total para generar subdominios** (ej. `cs.unsd.edu`) sin requerir autorización de los niveles superiores del árbol.
- **Límites Organizativos, No Físicos:** La estructura del árbol responde a divisiones administrativas y no a la topología física de la red. Varios departamentos situados en el mismo edificio compartiendo una misma red LAN pueden tener dominios DNS distintos, mientras que equipos geográficamente distantes pueden pertenecer al mismo dominio.
#### 7.1.4 Consultas, Respuestas y Registros de Recursos
##### Consultas y Usos Especiales del DNS
- **Propósito de la Consulta:** La función fundamental del DNS es asignar nombres de dominio a sus correspondientes registros de recursos. El cliente o _stub resolver_ envía una solicitud a su resolutor local, el cual realiza búsquedas iterativas hasta obtener la respuesta.
- **Consulta de Registro A:** Es el tipo de consulta más frecuente y solicita la asignación de un nombre de dominio a una dirección IPv4 de 32 bits.
- **Listas Negras Basadas en DNS (DNSBL):** Además de mapear nombres a direcciones IP, las consultas DNS se utilizan para consultar bases de datos como las **DNSBL**, que rastrean direcciones IP asociadas a _spammers_ o _malware_. Para consultar una IP en una DNSBL (como la lista de políticas `pbl.spamhaus.org`), el cliente invierte los octetos de la dirección IP y los antepone al nombre de la lista negra.
##### Ampliaciones y Mejoras en las Consultas
Con el crecimiento de la red y las exigencias de rendimiento y seguridad, se introdujeron extensiones para sofisticar las consultas:
- **EDNS Client Subnet (EDNS0 CS):** En esta extensión, el resolutor recursivo local incluye la subred IP del cliente original (_stub resolver_) en la consulta enviada al servidor autoritativo. Esto permite al servidor de nombres autoritativo identificar la ubicación geográfica o topológica del cliente y devolver la dirección IP de un servidor réplica cercano, lo cual es fundamental para el funcionamiento de las Redes de Distribución de Contenidos (CDNs).
##### Estructura del Registro de Recurso (La 5-tupla)
Los datos devueltos en las respuestas DNS se organizan en **registros de recursos**. Aunque internamente se codifican en formato binario, en texto se representan mediante una estructura de cinco campos o **5-tupla**:

$[{DomainName} | {Timetolive} | {Class} |{Type} | {Value}]$

1. **Domain_name (Nombre de Dominio):** Indica el dominio al que aplica el registro y funciona como la clave principal de búsqueda en la base de datos.
2. **Time_to_live (TTL):** Expresa la estabilidad del registro mediante la cantidad de segundos que una entrada puede ser conservada en una memoria caché remota antes de ser descartada. Para datos estables se asignan valores elevados (como 86400 segundos = 1 día), mientras que para información volátil o balanceo de carga se usan valores reducidos (como 60 segundos).
3. **Class (Clase):** Especifica la clase de red; para el tráfico de Internet adopta prácticamente siempre el valor `IN`.
4. **Type (Tipo):** Define el tipo de registro y la estructura de los datos contenidos.
5. **Value (Valor):** Es la carga útil o contenido del registro, el cual varía entre enteros de 32/128 bits, nombres de dominio o cadenas ASCII según el tipo.
##### Principales Tipos de Registros de Recursos

|Tipo|Significado|Valor y Descripción|
|:--|:--|:--|
|**SOA**|_Start of Authority_|Especifica la fuente principal de información sobre la zona, correo del administrador, número de serie y temporizadores.|
|**A**|Dirección IPv4|Almacena una dirección IPv4 de 32 bits correspondiente a una interfaz del host.|
|**AAAA**|Dirección IPv6|Almacena una dirección IPv6 de 128 bits (_quad A_).|
|**NS**|_Name Server_|Especifica el nombre del servidor de nombres autoritativo para el dominio o subdominio.|
|**MX**|_Mail Exchange_|Indica el nombre del host dispuesto a aceptar correo electrónico para el dominio, junto con un valor numérico de prioridad.|
|**CNAME**|_Canonical Name_|Define un alias que apunta al nombre de dominio canónico o verdadero (por ejemplo, permitiendo que `www.cs.uchicago.edu` apunte a `hnd.cs.uchicago.edu`).|
|**PTR**|_Pointer_|Apunta a otro nombre de dominio y se utiliza para **búsquedas inversas** (asociar una dirección IP a su nombre).|
|**SRV**|Servicio|Identifica el host y puerto para un servicio determinado en el dominio, generalizando la función del registro MX.|
|**SPF**|_Sender Policy Framework_|Codifica qué máquinas o direcciones IP tienen permitido enviar correo saliente del dominio para combatir el spam falsificado.|
|**TXT**|Texto|Diseñado para texto ASCII descriptivo; actualmente almacena información estructurada legible por máquina, principalmente datos SPF.|
##### Registros para la Seguridad (DNSSEC)
Dado que el DNS original carecía de autenticación y permitía la manipulación de registros en tránsito, se desarrollaron las extensiones de seguridad **DNSSEC** (RFC 3833).
En DNSSEC, los registros del mismo tipo para un dominio se agrupan en conjuntos denominados **RRSET** (_Resource Record Set_) y se firman criptográficamente mediante la clave privada de la zona autoritativa. Para gestionar las firmas y claves públicas, se introducen dos tipos de registros adicionales:

1. **RRSIG (Resource Record Signature):** Almacena la firma digital calculada sobre un RRSET, generada con la clave privada del servidor autoritativo.
2. **DNSKEY:** Almacena la clave pública utilizada para verificar las firmas RRSIG de los conjuntos RRSET; esta clave pública se encuentra firmada a su vez por la clave privada del dominio padre para establecer la cadena de confianza.
#### Zona DNS 
Zonas DNS se divide en zonas delegadas dentro de la jerarquía de dominios. Esto permite que distintas organizaciones o suborganizaciones gestionen sus propios registros, manteniendo el sistema distribuido, escalable y tolerante a fallos.

[[2 DNS seccion 7.1.5 hasta 7.1.8]]