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

El **agente de usuario** (conocido popularmente como **lector de correo** o MUA, _Mail User Agent_) es un programa que acepta una serie de comandos para redactar, recibir, responder y organizar mensajes, además de permitir la manipulación remota de los buzones de correo.

- **Ejemplos populares:** Google Gmail, Microsoft Outlook, Mozilla Thunderbird y Apple Mail.
- **Entorno de ejecución:** Se ejecuta localmente en el dispositivo del usuario (PC, laptop, smartphone) y solo funciona cuando el usuario lo inicia, a diferencia de los agentes de transferencia (MTA), que son procesos de servidor que se ejecutan continuamente en segundo plano.
#### Tipos de Interfaz

- **Gráficas y Táctiles (Modernas):** Basadas en menús, botones e iconos accionables mediante ratón o pantallas táctiles en dispositivos móviles.
- **Basadas en Texto (Clásicas):** Lectores más antiguos como _Elm_, _mh_ y _Pine_, que funcionan en consola y responden a comandos de un solo carácter desde el teclado.
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
