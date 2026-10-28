# Resolucion de nombres 7.1.5
#### Concepto y Servidores de Nombres
El proceso de buscar un nombre y encontrar su dirección de red se denomina **resolución de nombres**. Para evitar un punto único de fallo y distribuir la carga, la base de datos DNS se divide en **zonas no solapadas**, asociadas a servidores de nombres:

- **Servidor Primario:** Obtiene los registros de recursos de la zona directamente de un archivo almacenado en su disco local.
- **Servidores Secundarios:** Replican periódicamente la información desde el servidor primario.
- **Registros Autoritativos vs. en Caché:** Si el dominio consultado cae dentro de la jurisdicción directa del servidor de nombres, este devuelve **registros autoritativos**, los cuales provienen de la entidad gestora y son correctos por definición. En cambio, las respuestas obtenidas de memorias caché locales corresponden a consultas pasadas y corren el riesgo de estar desactualizadas.
#### Búsqueda Recursiva vs. Búsqueda Iterativa
Cuando se requiere resolver un dominio remoto que no está en caché ni en la zona local, intervienen dos modalidades de consulta:
- **Consulta Recursiva:** El emisor (_stub resolver_) le exige al servidor local (_local recursive resolver_) que gestione la búsqueda completa y le entregue únicamente la respuesta final sin respuestas parciales.
- **Consultas Iterativas:** Como los servidores superiores (como los raíz) están demasiado ocupados para hacer búsquedas completas, devuelven respuestas parciales o referencias al siguiente nivel jerárquico. El _local resolver_ asume el trabajo duro ejecutando una serie de consultas iterativas paso a paso a través de la jerarquía.
#### Los Servidores de Nombres Raíz (_Root Name Servers_)
Cuando no existe información previa en la caché local, la búsqueda remota debe comenzar necesariamente en la cima de la jerarquía:


- Existen **13 servidores raíz** lógicos, nombrados de `a.root-servers.net` a `m.root-servers.net`.
- No son equipos aislados, sino infraestructura fuertemente replicada en múltiples ubicaciones geográficas globales que utiliza **enrutamiento _anycast_** para dirigir el tráfico a la instancia más cercana, aumentando la fiabilidad y el rendimiento.
- Todo servidor de nombres local mantiene configurada la lista de direcciones IP de los servidores raíz para cargarla en caché al iniciar el sistema.
#### Ejemplo Detallado del Proceso de Búsqueda (En 10 Pasos)

El material ilustra la resolución iterativa completa cuando el host `filts.cs.vu.nl` desea obtener la dirección IP de `noise.cs.uchicago.edu`:
![[Pasted image 20261025130207.png|491]]

1. **Paso 1:** La aplicación en `filts.cs.vu.nl` invoca al _stub resolver_, el cual envía una consulta UDP (con nombre, tipo `A` y clase `IN`) a su servidor de nombres local (`cs.vu.nl`).
2. **Paso 2:** El servidor local (`cs.vu.nl`) no posee la información y envía la consulta iterativa a un **servidor de nombres raíz** (`a.root-servers.net`).
3. **Paso 3:** El servidor raíz no conoce la IP exacta, pero devuelve una respuesta parcial con los registros NS y A del servidor para el **TLD `.edu`** (`a.edu-servers.net`).
4. **Paso 4:** El servidor local reenvía la consulta completa al servidor del TLD `.edu`.
5. **Paso 5:** El servidor `.edu` responde indicando cuál es el servidor de nombres autoritativo del dominio `uchicago.edu`.
6. **Paso 6:** El servidor local envía la consulta al servidor de nombres de `uchicago.edu`.
7. **Paso 7:** Como el departamento de informática administra de forma autónoma su propia zona, el servidor de `uchicago.edu` responde redirigiendo al servidor específico de `cs.uchicago.edu`.
8. **Paso 8:** El servidor local consulta al servidor de nombres autoritativo de `cs.uchicago.edu`.
9. **Paso 9:** El servidor autoritativo de `cs.uchicago.edu` localiza el registro A y devuelve la dirección IP final (`128.135.24.19`).
10. **Paso 10:** El servidor de nombres local reenvía la dirección IP obtenida al cliente solicitante (`filts.cs.vu.nl`) y la guarda temporalmente en su memoria caché.
# Sección 7.1.6: Manos a la obra con el DNS (Exploración Práctica)

Esta sección aborda el uso de herramientas de línea de comandos para inspeccionar y diagnosticar de forma directa el funcionamiento del protocolo DNS, permitiendo verificar los registros de recursos, los tiempos de respuesta y el trazado de la resolución.
#### 1. Herramienta principal en UNIX / Linux / macOS: `dig` (_Domain Information Groper_)
- **Propósito:** Es la herramienta de exploración DNS estándar en sistemas basados en UNIX. Permite realizar consultas arbitrarias a cualquier servidor de nombres específico y mostrar la respuesta completa en texto.
- **Ejemplo de uso práctico:**
    
    ```
    dig ns @a.edu-servers.net cs.uchicago.edu
    ```
    
    Este comando envía una consulta solicitando los servidores de nombres (`NS`) de `cs.uchicago.edu` directamente al servidor autoritativo del TLD `.edu` (`a.edu-servers.net`). Permite reproducir manualmente los pasos individuales de una resolución iterativa (por ejemplo, el paso 4 del ejemplo de resolución en 10 pasos).
#### 2. Herramientas en entornos Microsoft Windows

- **`nslookup [dominio]`:** Herramienta interactiva y de línea de comandos clásica para realizar consultas de nombres de dominio y comprobar direcciones IP o tipos de registros.
- **Utilidades del sistema (`ipconfig` y `netsh`):**
    - `ipconfig /all`: Muestra las direcciones IP de los servidores DNS asignados a las interfaces de red del equipo local.
    - `ipconfig /flushdns`: Vacía y purga la memoria caché local de DNS guardada por el _stub resolver_ del sistema operativo.
    - `netsh dnsclient`: Permite configurar y administrar los parámetros del cliente DNS desde la consola de Windows.
- **Cmdlets de PowerShell:**
    - `Resolve-DnsName`: Comando moderno de PowerShell equivalente a `dig` o `nslookup` para consultar registros DNS.
    - `Show-DnsServerCache`: Permite inspeccionar las entradas almacenadas en la caché del servidor o cliente de nombres.
# Sección 7.1.7: Privacidad DNS

Esta sección analiza las vulnerabilidades de privacidad asociadas al diseño original del DNS y las innovaciones recientes desarrolladas para proteger el tráfico de consultas de los usuarios.
#### 1. El problema histórico de la falta de privacidad
- **Tráfico en texto plano:** Históricamente, las consultas y respuestas DNS no contaban con ningún tipo de cifrado, transmitiéndose en texto claro sobre UDP o TCP (puerto 53).
- **Espionaje e inspección:** Cualquier intermediario en el camino de la red —como administradores de red local, proveedores de servicios de Internet (ISPs), operadores de redes Wi-Fi públicas o atacantes (_eavesdroppers_)— podía registrar y analizar todo el tráfico DNS de un usuario.
- **Exposición de datos sensibles:** Las consultas DNS revelan los hábitos de navegación, intereses personales e investigaciones médicas (por ejemplo, consultas a `webmd.com`).
- **Impacto en el Internet de las Cosas (IoT):** En hogares inteligentes, los dispositivos IoT (como cámaras de seguridad o monitores de sueño) emiten consultas DNS continuas que pueden identificar de forma única la presencia de dispositivos específicos y los patrones de actividad de los ocupantes dentro de la vivienda.
#### 2. Migración hacia el Transporte Cifrado (DoT y DoH)
Para prevenir el espionaje pasivo y la manipulación de datos, se desarrollaron protocolos de transporte cifrado entre el cliente (_stub resolver_) y el resolutor recursivo local:

- **DoT (_DNS over TLS_):** Encapsula los mensajes DNS tradicionales dentro de un canal seguro cifrado con **TLS** utilizando el puerto dedicado `853`.
- **DoH (_DNS over HTTPS_):** Encapsula las consultas DNS dentro de sesiones HTTP/2 o HTTP/3 sobre **HTTPS** (puerto `443`). Al camuflarse como tráfico web común, dificulta que terceros o cortafuegos puedan bloquear o diferenciar las búsquedas DNS.
#### 3. Implicaciones operativas y el debate de centralización
- **Pérdida de visibilidad para los ISPs:** Al estar cifrado el tráfico, los ISPs ya no pueden inspeccionar las consultas de sus clientes. Esto protege la privacidad del usuario, pero invalida mecanismos de red tradicionales administrados por el ISP, como el bloqueo de sitios maliciosos, el control de infecciones de _malware_ y los controles parentales basados en DNS.
- **Riesgo de centralización y TRR (_Trusted Recursive Resolvers_):**
    - Al integrar DoH directamente en los navegadores web, la resolución DNS se traslada del sistema operativo al navegador.
    - Puesto que el mercado de navegadores está dominado por pocos actores (como Google), las configuraciones por defecto determinan qué empresa o resolutor público (como Google `8.8.8.8` ) termina concentrando y observando el tráfico DNS global.
    - Esto motivó el establecimiento de políticas de **TRR (_Trusted Recursive Resolvers_)**, diseñadas para regular y seleccionar resolutores recursivos de confianza que operen bajo estrictas garantías de privacidad mediante DoT o DoH.
#### 4. Privacidad de Próxima Generación: Oblivious DNS (ODNS / ODoH)
Ni DoT ni DoH resuelven el problema de forma absoluta, ya que el operador del resolutor recursivo local todavía puede ver simultáneamente la dirección IP del usuario y el nombre de dominio consultado. Para eliminar este punto de confianza único, surgieron **ODNS (_Oblivious DNS_)** y **ODoH (_Oblivious DoH_)**:
![[Pasted image 20261027152547.png|589]]
![[Pasted image 20261027152542.png]]

1. **El Cliente:** Cifra la consulta DNS de origen y la envía al resolutor recursivo local.
2. **El Resolutor Recursivo Local:** Ve la dirección IP del cliente para encaminar el paquete, pero **no puede descifrar la consulta** (desconoce qué sitio web se está buscando).
3. **El Servidor Autoritativo / Destino ODNS:** Recibe la consulta, la descifra y la resuelve, pero **desconoce la dirección IP del cliente** que la originó.

_Resultado:_ Ninguna entidad intermedia o servidor posee al mismo tiempo la identidad del cliente (su IP) y el contenido de su consulta (el dominio solicitado), logrando la desvinculación completa de los datos.
# Seccion 7.1.8
La sección **7.1.8 ("Contención y disputas por los nombres / Disputas por nombres")** analiza cómo los nombres de dominio trascendieron su función técnica original de traducir direcciones IP para convertirse en **activos comerciales, culturales y políticos de gran valor económico**.
#### El Valor Comercial y la Especulación de Dominios
- **Activos digitales cotizados:** Los nombres de dominio se han convertido en propiedades digitales extremadamente valiosas.
- **Saturación de `.com`:** La elevada demanda comercial derivó en la ocupación de prácticamente todas las palabras comunes del idioma inglés, así como sus errores ortográficos habituales, nombres de productos, plantas y animales dentro del gTLD `.com`.
- **Monetización de ccTLDs por países:** Ciertos países lograron capitalizar estratégicamente su código geográfico de dos letras. 
#### Debates Políticos y Criterios de Elegibilidad en la ICANN
Como organismo supervisor global, la **ICANN** enfrenta constantes presiones al decidir sobre la creación o regulación de nuevos dominios de nivel superior (gTLDs):

- **El debate del TLD `.xxx`:** La creación de este dominio para la industria de contenidos para adultos requirió **varios años de deliberaciones y demandas judiciales**. El conflicto enfrentó a quienes buscaban concentrar dicho material en un único dominio para facilitar el bloqueo mediante filtros parentales frente a quienes se oponían por motivos éticos o morales.
- **Definición de restricciones (El caso `.pro`):** Diseñado originalmente para "profesionales cualificados", generó amplias controversias para determinar qué profesiones eran elegibles (mientras médicos y abogados eran admitidos sin discusión, surgió la duda de si aplicaba a fotógrafos autónomos, profesores de música, plomeros o artesanos).
#### Ciberocupación (_Cybersquatting_)
- **Concepto:** Es la práctica deliberada de registrar nombres de dominio —especialmente marcas reconocidas o corporativas que tardaron en ingresar al mundo digital— con la única intención de **revenderlos posteriormente a sus dueños o empresas interesadas a precios muy elevados**.
- **Regla de asignación general:** Por principio operativo básico, la adjudicación de nombres en los registradores se rige bajo el criterio de **"primer llegado, primer servido"** (_first-come, first-served_).
- **Infracciones y marcas registradas:** La regla de prioridad no aplica cuando existe fraude directo o violación de marcas registradas (_trademarks_). En estos casos, las empresas recurren a políticas formalizadas de arbitraje (como la política UDRP de la ICANN) para recuperar el dominio registrado con mala fe.
[[3 Correo Electronico]]
