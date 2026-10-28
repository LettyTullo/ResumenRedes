#### Sección 7.2: Correo Electrónico (Visión General y Evolución)
- **Historia e Impacto:** El correo electrónico existe desde hace más de cuatro décadas y es considerado la _"killer app"_ original de Internet. Evolucionó del ámbito puramente académico en los años 80 hacia una adopción masiva global en los 90, manteniéndose hasta hoy como el pilar fundamental para la comunicación corporativa e interempresarial.
- **Evolución Técnica:** Los primeros sistemas consistían en simples protocolos de transferencia de archivos donde la primera línea indicaba la dirección del destinatario. Con el tiempo, evolucionó para soportar listas de distribución, formatos enriquecidos con adjuntos multimedia (MIME) e interfaces gráficas avanzadas que permiten el acceso desde computadoras y dispositivos móviles.
- **El problema del Spam:** La inmensa mayoría del tráfico diario (aproximadamente **9 de cada 10 mensajes**) corresponde a correo no deseado o _spam_. Esto obliga a que tanto los servidores como los programas lectores mantengan sistemas de filtrado continuo y listas de reputación para separar el correo legítimo de la basura.
#### Sección 7.2.1: Arquitectura y Servicios del Correo Electrónico

La arquitectura del sistema de correo electrónico se organiza en torno a dos subsistemas principales interconectados mediante **buzones** y un **formato de mensaje estandarizado**:
#### **A. Componentes de la Arquitectura**
1. **Agentes de Usuario (MUA — _User Agents_):**
    
    - Son las aplicaciones de lectura y composición con las que interactúa directamente el usuario (ej. Google Gmail, MS Outlook, Mozilla Thunderbird, Apple Mail).
    - **Funciones:** Permiten redactar, mostrar, responder, reenviar y clasificar mensajes en carpetas. También ejecutan tareas automáticas como preclasificación de _spam_ o configuración de respuestas automáticas ("de vacaciones").
    - **Entorno:** Se ejecutan localmente en los dispositivos de los usuarios (PC, laptop, smartphone) y no requieren estar activos de forma continua.
2. **Agentes de Transferencia de Mensajes (MTA — _Message Transfer Agents_):**
    
    - Son procesos de servidor que se ejecutan continuamente en segundo plano en las máquinas servidoras de correo.
    - **Funciones:** Mueven automáticamente los mensajes a través de la red desde el origen hasta el destino utilizando el protocolo **SMTP** (_Simple Mail Transfer Protocol_, RFC 822 / RFC 5321) e informan el estado de entrega o errores.
    - **Funciones avanzadas:** Gestionan listas de distribución masiva, copias carbón (Cc), copias ocultas (Bcc), prioridades y soporte para correo cifrado.
3. **Buzones de Correo (_Mailboxes_):**
    
    - Son repositorios alojados en los servidores de correo que almacenan los mensajes entrantes de cada usuario.
    - Permiten que un usuario consulte, organice o elimine sus mensajes de forma remota utilizando diferentes agentes de usuario desde múltiples dispositivos.
#### **B. Etapas del Flujo de Entrega**

El viaje de un mensaje desde el emisor hasta el receptor consta de tres pasos principales:

```
[MUA Remitente] --(1. Envío por correo)--> [MTA Emisor] --(2. Transferencia SMTP)--> [MTA Receptor / Buzón] --(3. Entrega Final)--> [MUA Receptor]
```

1. **Envío de correo (_Mail Submission_):** El agente de usuario (MUA) entrega el mensaje recién redactado a su servidor de correo local (MTA emisor).
2. **Transferencia de mensajes (_Message Transfer_):** El MTA emisor encamina y transfiere el mensaje a través de Internet hacia el MTA del destinatario mediante el protocolo SMTP.
3. **Entrega final (_Final Delivery / Mail Retrieval_):** El MUA del destinatario recupera o consulta los mensajes almacenados en su buzón del MTA receptor utilizando protocolos de acceso como IMAP, POP3 o interfaces Webmail.
#### **C. Estructura del Mensaje (Sobre vs. Contenido)**

Para permitir el transporte y la lectura adecuada, el sistema establece una distinción clara entre el **sobre** y el **contenido del mensaje**:

- **El Sobre (_Envelope_):** Encapsula el mensaje y contiene la información de control de transporte (dirección de destino, prioridad, nivel de seguridad). Es leído y procesado por los agentes de transferencia (MTAs) para realizar el enrutamiento.
- **El Mensaje / Contenido:** Se divide en dos partes:
    - **Cabecera (_Header_):** Contiene líneas de texto ASCII estructuradas para el agente de usuario, incluyendo campos esenciales como `Para:`, `De:`, `Cc:`, `Bcc:`, `Asunto:`, `Fecha:` y `Message-Id:`.
    - **Cuerpo (_Body_):** Contiene el texto o material destinado exclusivamente al receptor humano.
# Seccion 7.2.3 Agente de usuario

	![[Pasted image 20261028104806.png|468]] 
#### Elementos Principales de la Interfaz y Resumen del Buzón

Al iniciar el agente de usuario, este presenta un **resumen formateado del contenido del buzón** (por lo general ordenado por fecha de recepción, mostrando los más recientes primero):

- **Campos clave mostrados:** Extrae y da formato legible a los campos de cabecera `De:` (remitente), `Asunto:` (tema del mensaje) y `Recibido:` (fecha y hora).
- **Indicadores visuales e iconos:**
    - ✉️ **Sobre:** Indica mensaje no leído.
    - 📎 **Clip:** Indica que el mensaje contiene archivos adjuntos.
    - ❗ **Signo de exclamación:** Indica correo de alta prioridad/importancia definido por el remitente.
- **Vista previa (_Preview_):** Muestra los primeros caracteres del cuerpo del mensaje para ayudar al usuario a decidir si desea abrirlo, ignorarlo o marcarlo directamente como _spam_.
#### Disposición del Mensaje (_Message Disposition_)
Una vez que el usuario lee un mensaje, el agente de usuario le permite realizar las siguientes acciones:

1. **Borrar:** Elimina o traslada el mensaje a la papelera.
2. **Responder / Reenviar:** Crea un nuevo mensaje pre-llenando las direcciones del remitente original o enviándolo a un tercero.
3. **Archivar en carpetas:** Organiza los mensajes guardados en múltiples carpetas clasificadas por remitente, tema o categoría.
#### Procesamiento Automático y Filtrado
El agente de usuario puede realizar tareas automáticas en segundo plano sin intervención manual directa:
- **Detección y preclasificación de _Spam_:** Examina los campos y contenido del mensaje y, combinándolos con el historial de decisiones del usuario o firmas conocidas, separa el correo legítimo del correo basura hacia la carpeta de _spam_.
- **Respuestas automáticas:** Configuración de reglas especiales para enviar notificaciones automáticas ante ausencias (ej. _"Estoy de vacaciones y responderé a mi regreso"_).
# Seccion 7.2.3 Formato de mensajes 
#### Estructura General del Mensaje (RFC 5322 / RFC 822)
El formato estándar del correo en Internet proviene de la especificación RFC 822 original, actualizada posteriormente por el **RFC 5322**. Un mensaje de correo consta de tres componentes principales:
1. **Sobre primitivo (_envelope_):** Contiene la información de transporte necesaria para el encaminamiento por parte de los MTAs (como las direcciones reales de envío y entrega), según se define en el protocolo SMTP (RFC 5321).
2. **Campos de cabecera (_headers_):** Conjunto de líneas de texto ASCII ordenadas lógicamente con la estructura `Nombre-de-Campo: Valor` que contienen información de control.
3. **Línea en blanco:** Separa obligatoriamente el bloque de cabeceras del cuerpo del mensaje.
4. **Cuerpo (_body_):** Contenido libre destinado al destinatario humano.
#### Cabeceras Relacionadas con el Transporte
El RFC 5322 define un conjunto de cabeceras estándar orientadas a la entrega y trazabilidad del mensaje:
- **`Para:` (`To:`):** Dirección o direcciones de correo electrónico de los destinatarios principales.
- **`Cc:` (`Carbon copy`):** Destinatarios secundarios. Técnicamente no hay diferencia en la entrega respecto al campo `Para:`, pero establece una distinción psicológica para los receptores.
- **`CCO:` / `Bcc:` (`Blind carbon copy`):** Copia oculta. Esta línea se elimina automáticamente de todas las copias entregadas a los destinatarios principales y secundarios, permitiendo enviar copias a terceros de forma confidencial.
- **`De:` (`From:`):** Indica la persona o entidad que creó o redactó el mensaje (campo obligatorio).
- **`Remitente:` (`Sender:`):** Dirección del remitente real que transmite el mensaje. Puede omitirse si coincide con el campo `De:`, pero es útil si un asistente envía un correo a nombre de un ejecutivo.
- **`Recibido:` (`Received:`):** Línea añadida secuencialmente por cada servidor MTA a lo largo de la ruta. Incluye la identidad del servidor, la fecha y hora de recepción, sirviendo para depurar problemas de enrutamiento.
- **`Return-Path:`:** Añadida por el último MTA de la cadena para indicar la dirección exacta a la que deben enviarse los informes de error o rebotes.
#### Cabeceras de Control del Usuario y Cabeceras Privadas
Existen cabeceras adicionales utilizadas por los agentes de usuario:

- **`Date:`:** Fecha y hora de emisión del mensaje.
- **`Respuesta a:` (`Reply-To:`):** Dirección a la que deben dirigirse las respuestas. Se utiliza cuando el emisor o redactor no desea recibir las respuestas en su propia cuenta.
- **`Message-Id:`:** Identificador único generado automáticamente para enlazar mensajes en conversaciones y evitar entregas duplicadas.
- **`In-Reply-To:`** y **`Referencias:`:** Almacenan el `Message-Id` del mensaje original al que se responde para agrupar correos por hilos.
- **`Asunto:` (`Subject:`):** Breve resumen del contenido para la vista previa en el lector de correo.
- **Cabeceras Privadas (`X-`):** El estándar permite a los usuarios o aplicaciones inventar cabeceras personalizadas. Por convención, deben comenzar con la cadena **`X-`** para garantizar que nunca colisionen con futuras cabeceras oficiales estandarizadas.
#### MIME (_Multipurpose Internet Mail Extensions_)
Inicialmente, el correo electrónico solo admitía texto plano en inglés codificado en **ASCII de 7 bits** con un límite máximo de 1000 caracteres por línea. En la década de 1990, la expansión global de Internet hizo necesario enviar textos con acentos/diacríticos, alfabetos no latinos (ruso, hebreo, japonés) y archivos binarios (imágenes, audio, ejecutable, documentos).

Para solucionar esto sin romper la infraestructura de servidores existente, se creó **MIME** (RFC 2045, RFC 4288, RFC 4289). MIME añade estructura al cuerpo del mensaje e introduce **cinco nuevas cabeceras**:

1. **`Versión MIME:` (`MIME-Version:`):** Identifica la versión de MIME utilizada (ej. `1.0`). Si esta cabecera no está presente, se asume texto plano en ASCII.
2. **`Content-Description:`:** Cadena de texto legible que describe el contenido del mensaje.
3. **`Content-Id:`:** Identificador único del contenido.
4. **`Content-Transfer-Encoding:`:** Especifica el esquema mediante el cual se codificó el cuerpo para su transmisión.
5. **`Tipo de contenido:` (`Content-Type:`):** Especifica la naturaleza y formato del contenido.

#### Esquemas de Codificación y Tipos de Contenido (`Content-Type`)
#### Esquemas de Codificación (_Content-Transfer-Encoding_):
- **7-bit:** ASCII puro sin líneas superiores a 1000 caracteres.
- **8-bit:** Permite caracteres extendidos de 8 bits (valores 0–255) respetando el límite de longitud de línea.
- **Binary:** Permite datos binarios sin restricción de longitud de línea (ej. archivos ejecutables).
- **Base64:** Divide grupos de 24 bits de datos binarios en cuatro unidades de 6 bits, representando cada una mediante 64 caracteres ASCII imprimibles. Añade un ~33% de sobrecarga pero garantiza la transmisión segura sobre cualquier servidor tradicional.
- **Quoted-Printable:** Ideal para textos mayoritariamente ASCII con pocos caracteres especiales. Mantiene intactos los caracteres normales y codifica los especiales mediante su valor hexadecimal precedido de un signo igual (ej. la letra `ñ` se codifica como `=C3=B1`).
#### Tipos de Contenido (_Content-Type_):
Se definen mediante la sintaxis **`tipo/subtipo`** (administrados centralmente por la IANA):

- **`texto`:** `plano`, `html`, `xml`, `css`.
- **`imagen`:** `jpeg`, `gif`, `png`, `tiff`.
- **`audio`:** `mpeg`, `mp4`, `básico`.
- **`vídeo`:** `mpeg`, `mp4`, `quicktime`.
- **`aplicación`:** `pdf`, `zip`, `javascript`, `octet-stream`.
- **`multiparte` (`multipart`):** Permite construir mensajes complejos que contienen **múltiples archivos adjuntos de distintos tipos**, donde cada parte tiene sus propias cabeceras MIME individuales.
# Seccion 7.2.4
La **sección 7.2.4 ("Transferencia de mensaje")** explica la mecánica mediante la cual los mensajes de correo electrónico son transferidos entre los componentes de la red utilizando el protocolo **SMTP** (_Simple Mail Transfer Protocol_) y sus extensiones **ESMTP**.
#### El Rol de SMTP en la Arquitectura de Correo
El protocolo SMTP (especificado originalmente en la RFC 821 y actualizado en la RFC 5321) opera en dos fases de la transferencia:

1. **Envío de correo (_Mail Submission_):** Es el paso en el cual el agente de usuario (MUA) transmite un mensaje recién redactado hacia su servidor de correo o agente de transferencia de mensajes (MTA).
2. **Transferencia entre servidores (_Message Transfer_):** Es el proceso mediante el cual el MTA emisor entrega el mensaje al MTA receptor a través de Internet en un solo salto.
#### Funcionamiento Básico y Diálogo SMTP
SMTP es un protocolo sencillo basado en texto ASCII, lo que facilita su desarrollo, prueba y depuración (permitiendo incluso realizar pruebas manuales mediante la herramienta `telnet` al puerto 25 de un servidor).
El proceso de transferencia sigue un diálogo cliente-servidor sobre una conexión **TCP en el puerto 25**:
- **Establecimiento de la sesión:** El emisor (cliente) inicia una conexión TCP al puerto 25 del servidor receptor. El servidor responde primero enviando una línea de bienvenida con el código numérico `220` para anunciar su identidad y disponibilidad. Si el servidor no está preparado, el cliente cierra la conexión para reintentar más tarde.
- **Saludo inicial (`HELO` / `EHLO`):** El cliente se identifica enviando el comando `HELO` seguido del nombre de su dominio.
- **Identificación del sobre:**
    - El cliente emite `MAIL FROM: <direccion_emisor>` para indicar la dirección del remitente.
    - A continuación, transmite uno o varios comandos `RCPT TO: <direccion_destino>` para especificar a los destinatarios. Cada destinatario es aceptado o rechazado individualmente por el servidor.
- **Envío del contenido (`DATA`):**
    - El cliente envía el comando `DATA`.
    - El servidor responde con el código `354` indicando que puede comenzar la transmisión.
    - El cliente envía las cabeceras y el cuerpo del mensaje, y señala el final de la transmisión enviando una línea que contiene únicamente un solo punto (`.`).
- **Cierre de la conexión:** Tras confirmar la recepción exitosa con el código `250`, el cliente envía el comando `QUIT` y el servidor responde con el código `221` para cerrar la conexión TCP.
#### Limitaciones del SMTP Tradicional
A pesar de su eficacia, la versión básica de SMTP presenta deficiencias clave:
1. **Ausencia de autenticación:** El comando `MAIL FROM` no verifica la identidad real del emisor, lo que facilita la suplantación de identidad (_spoofing_) y la propagación de _spam_.
2. **Restricción a ASCII de 7 bits:** Solo admite texto ASCII estándar de 7 bits con un límite de 1000 caracteres por línea. La transmisión de archivos binarios o adjuntos exige codificaciones como Base64 de MIME, lo que incrementa el tamaño del mensaje y consume ancho de banda de forma ineficiente.
3. **Tráfico en texto claro:** Transmite la información sin cifrar, dejando los mensajes expuestos a la inspección por terceros durante su tránsito.
#### ESMTP (_Extended SMTP_)

Para resolver estas deficiencias, el estándar RFC 5321 formalizó el mecanismo de extensión denominado **ESMTP** (_Extended SMTP_):
- **Mecanismo de negociación:** El cliente envía el comando **`EHLO`** en lugar de `HELO`. Si el servidor soporta ESMTP, responde enumerando las palabras clave de las extensiones que admite. Si rechaza `EHLO`, el cliente asume que se trata de un servidor SMTP clásico y continúa de la forma habitual.
- **Extensiones destacadas de ESMTP:**
    - **`AUTH`:** Permite autenticar al usuario mediante credenciales (usuario y contraseña) antes de permitir el envío.
    - **`STARTTLS`:** Negocia el cambio a un canal seguro cifrado mediante TLS.
    - **`BINARYMIME` / `CHUNKING`:** Habilita la transmisión directa de datos binarios de 8 bits y el envío de mensajes extensos en bloques.
    - **`SIZE`:** Permite verificar el tamaño total del mensaje antes de intentar enviarlo.
    - **`UTF8SMTP`:** Añade soporte para direcciones de correo internacionalizadas.

#### Envío de Correo (_Mail Submission_) vs. Transferencia Física entre MTAs
La arquitectura moderna diferencia el envío de correo desde clientes de la transferencia entre servidores:
- **Envío por el Agente de Usuario (Puerto 587):** Dado que los agentes de usuario funcionan en dispositivos móviles o portátiles y los MTAs residen en servidores de empresas o ISPs, los servidores no permiten la retransmisión abierta (_open relay_) para evitar atraer _spammers_. Por ello, para el envío de correo se exige la extensión **`AUTH`** y se emplea el **puerto 587** (definido en la RFC 4409 para _Mail Submission_) o los puertos 465 (SMTPS) y 2525, en lugar del puerto 25.
- **Transferencia Física entre MTAs (Puerto 25):**
    1. El MTA emisor realiza una consulta DNS solicitando los registros **MX (_Mail Exchanger_)** del dominio de destino.
    2. El DNS devuelve una lista ordenada con los nombres y direcciones IP de los servidores de correo autorizados.
    3. El MTA emisor establece una conexión TCP al **puerto 25** de la dirección IP del servidor de destino.
    4. Transfiere el mensaje utilizando el diálogo SMTP.
    5. El MTA receptor deposita el mensaje en el **buzón (_mailbox_)** del destinatario para su posterior lectura.
#### Retransmision y entrega multiple de correo
Hay situaciones en las que un mensaje puede ser entregado varias veces:
Listas de correo: se recibe un mensaje para la lista. Luego se expande como un mensaje para cada miembro de la lista que se envía a las direcciones individuales de los miembros. 
Reenvíos o alias: Por ejemplo, Bob tiene la dirección bob@alum.mit.edu, pero quiere que todos los correos lleguen a bob@ee.uwa.edu. En este caso: 
1. Primero, el correo se envía al servidor de alum.mit.edu.
2. Luego, ese servidor reenvía el mensaje al servidor de ee.uwa.edu.au. Cada una de estas etapas es una entrega completa y separada en cuanto a los agentes de transferencia de correo se refiere.
# Seccion 7.2.5 Entrega Final

#### El Desafío de la Entrega Final y la Limitación de SMTP
- **Evolución del acceso:** En los inicios de Internet, el MUA y el servidor de correo (MTA) se ejecutaban localmente en el mismo ordenador; el MTA simplemente escribía los mensajes al final del archivo del buzón en disco y el MUA leía directamente dicho archivo.
- **El escenario moderno:** Actualmente, los usuarios necesitan acceder a su correo desde **múltiples dispositivos** (smartphones, laptops, PCs de oficina) y ubicaciones geográficas diversas.
- **¿Por qué no se utiliza SMTP para la entrega final?** SMTP es un protocolo basado exclusivamente en **push** (empujar datos conectándose a un servidor remoto que está continuamente encendido). No puede utilizarse para entregar correo al cliente final porque el dispositivo del usuario suele estar apagado, desconectado temporalmente o cambiando de dirección IP. Por ello, se requieren protocolos de gestión remota basados en **pull**.
#### POP3 (_Post Office Protocol, versión 3_ — RFC 1939)

- **Funcionamiento:** Es un protocolo de descarga antiguo y sencillo.
- **Mecánica:** Se conecta al servidor, **descarga los mensajes** al ordenador del cliente y normalmente los elimina del servidor.
- **Desventajas principales:**
    1. Dificulta el acceso sincronizado desde múltiples dispositivos (si un mensaje se descarga en la laptop, desaparece del servidor y ya no estará disponible en el teléfono).
    2. Riesgo de pérdida total de datos si el disco duro o dispositivo local del cliente sufre una avería.
#### IMAP (_Internet Message Access Protocol, versión 4_ — RFC 3501)
- **Concepto clave:** Diseñado para **manipular y gestionar el buzón de forma remota directamente en el servidor**.
- **Mecánica de funcionamiento:**
    - Escucha en el **puerto TCP 143**.
    - El cliente se conecta, establece un transporte seguro (TLS/STARTTLS), se autentica con sus credenciales y emite comandos IMAP (como `SELECT`, `FETCH`, `SEARCH`, `DELETE`, `STORE`).
- **Ventajas sobre POP3:**
    1. **Sincronización multidispositivo:** Los mensajes permanecen centralizados en el servidor, por lo que los cambios de estado (mensajes leídos, carpetas, marcadores) se reflejan en todos los dispositivos.
    2. **Descarga parcial:** Permite descargar solo las cabeceras o partes específicas de un mensaje (por ejemplo, el texto plano) sin descargar adjuntos pesados hasta que el usuario lo solicite explícitamente.
    3. **Búsqueda en servidor:** Ejecuta búsquedas complejas en el servidor para evitar descargar miles de correos a la máquina local.
#### Correo Web (_Webmail_)
- **Arquitectura:** Es una alternativa donde la Web funciona como interfaz del Agente de Usuario (ejemplos populares: **Google Gmail, Microsoft Hotmail/Outlook.com, Yahoo! Mail**).
- **Mecanismo de funcionamiento:**
    - El servidor de correo del proveedor recibe el correo entrante utilizando **SMTP tradicional en el puerto 25**.
    - El usuario accede y se autentica mediante un formulario web en un navegador (sobre **HTTP/HTTPS**).
    - El servidor genera dinámicamente páginas web (apoyadas en programas JavaScript en el cliente) que representan el contenido del buzón en tiempo real.
    - El envío de mensajes se realiza mediante peticiones web comunes (como HTTP POST), y es el servidor web el que inyecta el correo al sistema de transferencia SMTP tradicional.
- **Principal ventaja:** **Acceso universal** desde cualquier navegador web sin necesidad de instalar o configurar clientes de correo locales.

