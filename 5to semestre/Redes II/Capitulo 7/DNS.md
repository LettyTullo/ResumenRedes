# Capítulo 7: La Capa de Aplicación (Visión General)

La **Capa de Aplicación** es el nivel superior de la arquitectura de red donde residen las aplicaciones reales con las que interactúan los usuarios. Todas las capas inferiores existen exclusivamente para proporcionar servicios de transporte confiables o no confiables, pero no realizan el trabajo directo que requieren los usuarios.

Incluso en este nivel, existen **protocolos de apoyo** fundamentales que permiten el funcionamiento global de las aplicaciones. El más importante es el **Sistema de Nombres de Dominio (DNS)**, el cual asigna nombres de Internet a direcciones IP. A partir de este servicio de soporte se estructuran las aplicaciones reales de la red, como el correo electrónico, la _World Wide Web_ (WWW), la transmisión de multimedia (_video streaming_) y las redes de distribución de contenidos (CDNs y P2P).
# Sección 7.1: El Sistema de Nombres de Dominio (DNS)

El **DNS** (_Domain Name System_) es un esquema de nombres jerárquico implementado sobre una base de datos distribuida. Está especificado principalmente en las RFC 1034, RFC 1035 y RFC 2181.
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
    - **Minimización de QNAME:** Para proteger la privacidad del usuario y evitar revelar el nombre completo consultado (_FQDN_) a todos los servidores de la cadena, el _local resolver_ envía a cada nivel jerárquico únicamente la etiqueta del nombre estrictamente necesaria para continuar la delegación (por ejemplo, envía solo `uchicago.edu` al servidor del TLD `.edu`).
    - **Codificación 0x20:** Dado que los nombres en las consultas no distinguen entre mayúsculas y minúsculas, este mecanismo altera arbitrariamente las mayúsculas y minúsculas del nombre consultado (ej. `uCHicaGO.EDu`) para inyectar bits de entropía adicionales al ID de transacción de 16 bits, dificultando ataques de envenenamiento de caché como el ataque Kaminsky.
#### 7.1.3 El Espacio de Nombres DNS y su Jerarquía

El espacio de nombres está organizado como una estructura jerárquica en árbol, análoga al sistema postal:

- **Dominio Raíz:** Es el nodo superior del árbol y se representa formalmente por un punto (`.`).
- **FQDN (_Fully Qualified Domain Name_):** Es el nombre de dominio completo que inicia con la máquina específica y recorre la jerarquía hasta concluir formalmente en el punto raíz (por ejemplo, `www.cs.uchicago.edu.`).
- **Dominios de Nivel Superior (TLDs - _Top-Level Domains_):** Administrados a nivel global por la **ICANN**. Se dividen en dos categorías principales:
    1. **gTLDs (genéricos):** `.com`, `.edu`, `.org`, `.net`, `.gov`, `.biz`, entre otros.
    2. **ccTLDs (por país):** Códigos geográficos de dos letras basados en la norma ISO 3166 (ej. `.py`, `.us`, `.uk`, `.nl`).
- **Dominios de Segundo Nivel y Subdominios:** Constituyen las ramas intermedias y hojas del árbol. Por ejemplo, en el nombre `cs.uchicago.edu`, `uchicago` es el dominio de segundo nivel registrado bajo `.edu`, mientras que `cs` es un subdominio administrado internamente por el departamento de informática de esa universidad[6][7].
- **Delegación y límites:** Cada dominio tiene autoridad para administrar y delegar subdominios por debajo de él (ejemplo: `.edu.py` bajo el ccTLD `.py`). Una vez registrado un dominio de segundo nivel, la organización propietaria puede crear nuevos subdominios sin solicitar permisos adicionales. Los nombres siguen divisiones organizativas y no coinciden necesariamente con las fronteras de las redes físicas o LANs.

![[Pasted image 20261025124343.png|456]]
##### Cadena de Administración del Árbo
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

Cada dominio o _host_ tiene asociado un conjunto de **registros de recursos** que conforman la base de datos distribuida de DNS.

- **Estructura del Registro de Recurso (5-tupla):** Todos los registros se representan conceptualmente mediante la tupla: \[\text{Domain_name} \mid \text{Time_to_live} \mid \text{Class} \mid \text{Type} \mid \text{Value}\]
    
    1. **Domain_name:** Clave principal de búsqueda que indica a qué dominio aplica el registro.
    2. **Time_to_live (TTL):** Tiempo en segundos que el registro puede almacenarse en caché.
    3. **Class:** Indica la clase de red (para tráfico de Internet es casi siempre `IN`).
    4. **Type:** Especifica el tipo de registro y el formato de los datos contenidos.
    5. **Value:** Valor asociado (dirección IP, nombre de dominio o cadena de texto).
- **Principales Tipos de Registros:**
    

|Tipo|Significado|Valor y Descripción|
|:--|:--|:--|
|**SOA**|_Start of Authority_|Proporciona parámetros principales de la zona (administrador, número de serie, temporizadores).|
|**A**|Dirección IPv4|Dirección IP binaria de 32 bits de un _host_.|
|**AAAA**|Dirección IPv6|Dirección IP binaria de 128 bits de un _host_ (_quad A_).|
|**NS**|_Name Server_|Especifica el nombre del servidor de nombres autoritativo para el dominio o subdominio.|
|**MX**|_Mail Exchange_|Indica el nombre del servidor de correo dispuesto a recibir e-mails para el dominio, junto con su prioridad.|
|**CNAME**|_Canonical Name_|Define un alias que apunta al nombre de dominio canónico o verdadero.|
|**PTR**|_Pointer_|Apunta a un nombre de dominio; se utiliza para **búsquedas inversas** (asociar una dirección IP a su nombre).|
|**SRV**|Servicio|Generaliza el registro MX permitiendo especificar _host_ y puerto para servicios arbitrarios.|
|**SPF / TXT**|Texto|`SPF` define qué máquinas pueden enviar correo del dominio; `TXT` almacena texto ASCII legible por máquina (como políticas SPF).|

- **Usos especiales y extensiones:**
    
    - **DNSBL (_DNS-based Blacklist_):** Listas negras para rastrear direcciones IP asociadas con _spammers_ o _malware_. Se consultan invirtiendo los octetos de la IP e invocando a dominios especiales como `pbl.spamhaus.org`.
    - **EDNS0 CS (_Extended DNS Client Subnet_):** Permite al _local resolver_ transmitir la subred IP del cliente original al servidor autoritativo. Esto permite a las CDNs devolver la dirección IP de un servidor réplica geográficamente cercano al cliente.
- **Registros de Seguridad DNSSEC (RFC 3833):** DNSSEC añade autenticación mediante firmas digitales firmadas con criptografía de clave pública para evitar la manipulación de datos:
    
    1. **RRSIG:** Contiene la firma digital calculada sobre un conjunto de registros del mismo tipo (**RRSET**), firmada con la clave privada de la zona.
    2. **DNSKEY:** Almacena la clave pública de la zona para verificar los registros RRSIG; está firmada a su vez por la clave del dominio padre.

---

#### 7.1.5 Zonas DNS y Resolución de Nombres

Para garantizar la escalabilidad y tolerancia a fallos, la base de datos global de DNS se divide en **zonas delegadas**.

- **Archivo de Zona:** Es el archivo que almacena físicamente el conjunto de registros de recursos correspondientes a un dominio o subdominio.
- **Servidor Primario y Secundarios:** Cada zona cuenta con un **servidor de nombres primario** (que lee los datos desde su disco local) y uno o varios **servidores secundarios** (que replican periódicamente la información desde el servidor primario).

```
                    [Cliente filts.cs.vu.nl]
                               |
                        (1)    |    (10) Respuesta IP
                               v
                     [Local Resolver cs.vu.nl]
                       /       |       \
            (2) query /        |        \ (8) query
            (3) .edu v         | (5)     v (9) IP final
            [Servidor Raíz]    | uchicago.edu
                          (4)  |  (6) query
                        query  v  (7) cs.uchicago.edu
                     [Servidor TLD .edu] ---> [Servidor Autoritativo cs.uchicago.edu]
```

- **Pasos de la Resolución Iterativa (Ejemplo de 10 pasos):**
    
    1. El cliente `filts.cs.vu.nl` consulta a su _Local Resolver_ por el registro A de `noise.cs.uchicago.edu`.
    2. El _Local Resolver_ consulta a uno de los **Servidores Raíz**.
    3. El Servidor Raíz responde con una referencia al servidor del TLD `.edu`.
    4. El _Local Resolver_ consulta al servidor TLD `.edu`.
    5. El servidor `.edu` responde con la dirección del servidor de `uchicago.edu`.
    6. El _Local Resolver_ consulta al servidor de `uchicago.edu`.
    7. El servidor de `uchicago.edu` responde redirigiendo al servidor autoritativo de `cs.uchicago.edu`.
    8. El _Local Resolver_ consulta al servidor autoritativo de `cs.uchicago.edu`.
    9. El servidor autoritativo devuelve el registro A con la dirección IP final (`128.135.24.19`).
    10. El _Local Resolver_ reenvía la dirección IP al cliente.
- **Servidores Raíz:** Existen **13 servidores raíz** lógicos (denominados de `a.root-servers.net` a `m.root-servers.net`), los cuales están masivamente replicados en todo el mundo mediante enrutamiento _anycast_.
    

---

#### 7.1.6 a 7.1.8 Tendencias Modernas, Privacidad y Contención por Nombres

- **Cifrado de DNS (DoT / DoH):** Para evitar que terceros o ISPs puedan espiar o manipular las consultas, se ha migrado progresivamente desde UDP en texto plano hacia transportes cifrados como **DoT** (_DNS over TLS_) y **DoH** (_DNS over HTTPS_).
- **Resolutores Recursivos de Confianza (TRR):** Los navegadores modernos permiten redirigir todo el tráfico DNS del usuario hacia proveedores públicos de confianza (_Trusted Recursive Resolvers_) como Cloudflare o Google.
- **Oblivious DNS (ODNS / ODoH):** Propuesta avanzada para preservar la privacidad en la que el cliente cifra la consulta; el _local resolver_ observa la dirección IP del cliente pero no puede leer el dominio consultado, mientras que el servidor autoritativo resuelve el dominio pero desconoce la IP del cliente.
- **Contención por los Nombres:** Disputas políticas y comerciales administradas por ICANN relativas al control y creación de dominios de nivel superior (por ejemplo, los debates judiciales para la aprobación del gTLD `.xxx` o la definición de profesiones elegibles para el dominio `.pro`).

---

💡 ¿Te gustaría que practiquemos con alguna pregunta de examen sobre los **tipos de registros de recursos (A, MX, CNAME, PTR)** o sobre el rastreo paso a paso de una **resolución iterativa**?