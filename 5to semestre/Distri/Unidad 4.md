La **Unidad 4: Objetos Distribuidos e Invocación Remota** aborda cómo se comunican y cooperan los programas ejecutados en procesos y computadores distintos dentro de una aplicación distribuida. Para evitar que el desarrollador gestione manualmente el paso de mensajes a bajo nivel, la capa de **middleware** proporciona abstracciones de mayor nivel como **invocaciones remotas y eventos**.

El middleware aporta varias ventajas clave:

- **Transparencia de ubicación:** El cliente que invoca una operación no distingue si esta se ejecuta en su mismo proceso o en uno remoto.
- **Independencia del transporte, SO y hardware:** Funciona sobre cualquier red, sistema operativo y arquitectura de computación mediante procesos de empaquetado y desempaquetado de datos.
- **Soporte multilenguaje:** Permite construir aplicaciones cuyos componentes están escritos en distintos lenguajes de programación.

Existen **tres paradigmas de invocación remota** graduados de menor a mayor nivel de abstracción:

1. **Protocolos Petición-Respuesta**.
2. **Llamada a Procedimiento Remoto (RPC)**.
3. **Invocación a Método Remoto (RMI)**.

---

### **1. PROTOCOLOS PETICIÓN-RESPUESTA**

Es el modo fundamental de bajo nivel para ejecutar una operación remota y sirve de base para RPC y RMI.

#### **A. Características y Funcionamiento**

- **Sincronía y fiabilidad:** En el caso convencional, la comunicación cliente-servidor es **síncrona**, ya que el cliente se bloquea esperando la respuesta del servidor. Es **fiable** porque la respuesta del servidor actúa implícitamente como un acuse de recibo para el cliente.
- **Variante asíncrona:** Se utiliza cuando el cliente prefiere no bloquearse y recuperar los resultados más tarde.

#### **B. Modelo de Fallos y Tolerancia**

- **Fallos por omisión y temporización:** Se abordan implementando un tiempo límite de espera (_timeout_) en la operación de invocación (`doOperation()`). Ante un _timeout_, el sistema puede retornar un indicador de fallo o reintentar la petición de forma repetida.
- **Fallos Bizantinos y peticiones duplicadas:** Si un mensaje se duplica en la red o el cliente reintenta la operación, el servidor puede recibir peticiones repetidas:
    - Si la operación es **idempotente** (produce el mismo efecto final sin importar cuántas veces se ejecute), se puede reejecutar sin problemas.
    - Si **no es idempotente**, el servidor debe mantener un **histórico (_history_)** con los resultados de las operaciones anteriores (asociado a un identificador único `id_cliente + id_peticion`) para retransmitir la respuesta guardada sin reejecutar la operación.

#### **C. Estilos de Protocolos RPC**

Dependiendo de la necesidad de confirmación y retorno de datos, existen tres estilos:

1. **Protocolo Petición (R - _Request_):** Intercambia 1 solo mensaje. Se usa cuando el procedimiento no devuelve ningún valor y el cliente no requiere confirmación de ejecución.
2. **Protocolo Petición-Respuesta (RR - _Request-Reply_):** Intercambia 2 mensajes. La respuesta del servidor sirve como confirmación de la petición.
3. **Protocolo Petición-Respuesta-Confirmación de la respuesta (RRA - _Request-Reply-Acknowledge_):** Intercambia 3 mensajes. El cliente envía un acuse de recibo final para informarle al servidor que ya recibió la respuesta y que puede borrarla de su histórico.

#### **D. Ejemplo Clave de Protocolo RR: HTTP (_Hypertext Transfer Protocol_)**

HTTP es un ejemplo práctico del estilo Petición-Respuesta (RR) sobre el protocolo TCP.

- **Pasos de la interacción básica:** 1) El cliente solicita conexión TCP al puerto del servidor; 2) El cliente envía el mensaje de petición; 3) El servidor envía la respuesta; 4) Se cierra la conexión.
- **Soporte de funcionalidades:** Permite negociación de contenido (idioma, tipo de medio) y autenticación mediante credenciales.
- **Métodos HTTP principales:**
    - `GET`: Solicita el recurso indicado por la URL. Si se refiere a datos los devuelve; si es un programa, lo ejecuta y devuelve su salida.
    - `HEAD`: Idéntico a GET pero no devuelve el cuerpo de los datos, solo los encabezados (metadatos como tamaño o fecha de modificación).
    - `POST`: Envía un bloque de datos en el cuerpo para ser procesado por un recurso (formularios, publicar en tablones, insertar registros).
    - `PUT`: Guarda los datos enviados en la URL especificada (crea o reemplaza el recurso).
    - `DELETE`: Solicita borrar el recurso especificado.
    - `OPTIONS`: Solicita la lista de métodos HTTP permitidos en esa URL.
    - `TRACE`: El servidor devuelve el mismo mensaje recibido (usado para depuración).
- **Códigos de Estado HTTP:**
    - `100–199`: Respuestas informativas.
    - `200–299`: Respuestas satisfactorias (ej. `200 OK`).
    - `300–399`: Redirecciones (ej. `301 Moved Permanently`).
    - `400–499`: Errores del cliente (ej. `400 Bad Request`, `403 Forbidden`, `404 Not Found`, `408 Request Timeout`, `409 Conflict`).
    - `500–599`: Errores del servidor (ej. `500 Internal Server Error`, `504 Gateway Timeout`).

---

### **2. LLAMADA A PROCEDIMIENTO REMOTO (RPC)**

RPC representa un gran avance al extender la abstracción convencional de llamada a procedimiento a un entorno distribuido, logrando un alto grado de **transparencia de distribución**.

#### **A. Interfaces y Paso de Parámetros**

- **Imposibilidad de variables compartidas:** Los módulos en procesos separados no pueden acceder directamente a las variables de otros procesos. Las interfaces definen los procedimientos o métodos disponibles.
- **Paso de parámetros:**
    - **Entrada:** Se empaquetan en el mensaje de petición y se pasan al procedimiento del servidor.
    - **Salida:** Se devuelven en el mensaje de respuesta.
    - **Punteros:** **Los punteros de un proceso no son válidos en el proceso remoto**, por lo que NO se pueden pasar punteros como argumentos ni como valores de retorno.
- **Lenguajes de Definición de Interfaces (IDL):** Proveen una notación independiente del lenguaje de programación para especificar interfaces remotas.

#### **B. Arquitectura y Componentes de RPC**

Para lograr la transparencia, el middleware utiliza los siguientes elementos:

1. **Programa Cliente:** Hace la llamada local al procedimiento.
2. **Procedimiento de Resguardo del Cliente (_Client Stub_):** Enmascara la llamada remota haciéndose pasar por el procedimiento real. Empaqueta los argumentos (_marshalling_) en un mensaje y los entrega al módulo de comunicación.
3. **Módulo de Comunicación:** Transmite los mensajes de petición y respuesta entre ambos nodos.
4. **Distribuidor (_Dispatcher_):** Recibe el mensaje en el servidor y selecciona el _stub_ o procedimiento correspondiente.
5. **Procedimiento de Resguardo del Servidor (_Server Stub_):** Desempaqueta los argumentos (_unmarshalling_) del mensaje y llama al procedimiento de servicio real.
6. **Procedimiento de Servicio (_Service Procedure_):** Ejecuta la lógica local en el servidor y retorna el resultado.

#### **C. Semánticas de Invocación (Tolerancia a Fallos)**

Según las medidas combinadas de reintento de mensaje, filtrado de duplicados y retransmisión de respuestas, se obtienen **tres semánticas de invocación**:

|Semántica|Retransmisión de Peticiones|Filtrado de Duplicados|Reejecución u Opción de Respuesta|Descripción / Riesgos|
|:--|:-:|:-:|:-:|:--|
|**Tal vez (_Maybe_)**|No|No procede|No procede|El cliente no reintenta. La operación puede ejecutarse una vez o ninguna. No hay garantías.|
|**Al menos una vez (_At-least-once_)**|Sí|No|Reejecuta procedimiento|Se reintenta la petición. Se garantiza que el método se evalúa al menos una vez, pero si no es idempotente puede ejecutarse varias veces y causar errores.|
|**Como máximo una vez (_At-most-once_)**|Sí|Sí|Retransmite respuesta del histórico|Se reintentan peticiones pero el servidor filtra duplicados y retransmite la respuesta guardada en el histórico sin reejecutar.|

---

### **3. INVOCACIÓN A MÉTODO REMOTO (RMI)**

RMI es la extensión natural de RPC hacia el mundo de la **Programación Orientada a Objetos Distribuidos**. Permite que un objeto invoque métodos en objetos ubicados potencialmente en otros procesos o computadores.

#### **A. Conceptos Fundamentales del Modelo de Objetos Distribuidos**

1. **Invocaciones Locales vs. Remotas:** Las invocaciones locales ocurren entre objetos dentro del mismo proceso; las remotas (_RMI_) cruzan los límites del proceso o la red.
2. **Objeto Remoto:** Es aquel objeto capaz de recibir invocaciones remotas.
3. **Interfaz Remota:** Especifica cuáles métodos de un objeto remoto están disponibles para ser invocados remotamente. A diferencia de RPC, en RMI **se pueden pasar objetos como argumentos y resultados**.
4. **Referencia a Objeto Remoto:** Es un **identificador único global** a lo largo de todo el sistema distribuido que permite referenciar a un objeto remoto específico. Puede pasarse como argumento o resultado en otras llamadas RMI.

#### **B. Arquitectura e Implementación de RMI**

El sistema RMI se estructura en capas y módulos cooperantes:

1. **Proxy (Cliente):**
    - Actúa como la representación local del objeto remoto en el proceso cliente.
    - Hace transparente la invocación empaquetando el identificador del método y los argumentos (_marshalling_), y enviándolos mediante el módulo de comunicación.
2. **Módulo de Referencia Remota:**
    - Traduce entre referencias a objetos locales y remotas.
    - Mantiene la **Tabla de Objetos Remotos** en cada proceso:
        - Una entrada para cada **objeto remoto** implementado en ese servidor.
        - Una entrada para cada **proxy local** instanciado en ese cliente.
3. **Módulo de Comunicación:**
    - Transporta los mensajes utilizando el protocolo petición-respuesta. El mensaje contiene: `tipoMensaje`, `idPeticion`, `refObjetoRemoto` e `idMetodo`.
4. **Distribuidor (_Dispatcher_ - Servidor):**
    - Obtiene la referencia local desde la Tabla de Objetos Remotos y selecciona el **Esqueleto (_Skeleton_)** de la clase del objeto invocado.
5. **Esqueleto (_Skeleton_ - Servidor):**
    - Desempaqueta los argumentos del mensaje, invoca el método real sobre el objeto remoto en el servidor, empaqueta el resultado devuelto y lo envía de regreso.

---

### **4. RESUMEN EJECUTIVO PARA MEMORIZACIÓN RÁPIDA**

Para recordar la Unidad 4 en un examen, memoriza esta estructura en 4 niveles:

- **Nivel 1: Tres Paradigmas:**
    1. Petición-Respuesta (Bajo nivel / Base)
    2. RPC (Basado en Procedimientos)
    3. RMI (Basado en Objetos)
- **Nivel 2: Tres Estilos de Protocolo:**
    - **R** (Request)
    - **RR** (Request-Reply, ej. HTTP)
    - **RRA** (Request-Reply-Acknowledge)
- **Nivel 3: Tres Semánticas de Invocación:**
    - **Tal vez (_Maybe_):** 0 o 1 ejecución (sin reintentos).
    - **Al menos una vez (_At-least-once_):** 1 o más ejecuciones (reintentos sin filtro).
    - **Como máximo una vez (_At-most-once_):** Exactamente 0 o 1 ejecución (reintentos + filtro + histórico).
- **Nivel 4: Flujo de Componentes RMI / RPC:**
    - **Cliente:** Programa → Proxy / Client Stub.
    - **Capa Intermedia:** Módulo de Comunicación + Módulo de Referencia Remota (Tabla de Objetos).
    - **Servidor:** Dispatcher → Skeleton / Server Stub → Objeto / Procedimiento Real.

