Las redes definidas por software (**SDN**) representan un cambio radical en la arquitectura de red al desacoplar el control lógico del hardware físico.

>[!tip] El problema
>Tradicionalmente, el software que ejecuta los algoritmos de enrutamiento (como OSPF o BGP) ha estado **integrado verticalmente** con el hardware propietario de los routers. Esto impedía a los operadores modificar el comportamiento de los protocolos o innovar de forma ágil. Además, la ingeniería de tráfico se realizaba de manera indirecta y manual mediante la alteración de parámetros toscos de configuración que interactuaban de forma impredecible.

En el enrutamiento tradicional, cada router ejecuta localmente tanto su plano de datos (el hardware que reenvía paquetes a nivel de línea) como su plano de control (el software que corre protocolos distribuidos como OSPF o RIP para calcular las tablas de rutas).

>[!success] SDN rompe este esquema al **separar física y lógicamente ambos planos**:
>- **El plano de datos** se queda en los conmutadores (_switches_) físicos de la red, limitándose a realizar búsquedas rápidas en cabeceras de paquetes y ejecutar el reenvío según las instrucciones recibidas.
>- **El plano de control** se extrae de los dispositivos físicos y pasa a ejecutarse en un entorno de software externo.

# El Controlador: Lógicamente Centralizado

Esta separación permite que el plano de control funcione como un **programa de software lógicamente centralizado** (comúnmente llamado **controlador** o _controller_).

- **Programabilidad en alto nivel:** El controlador se ejecuta de forma independiente de los enrutadores y puede estar escrito en lenguajes de alto nivel como **Python, Java, Golang o C**.
- **Visión global y cálculo remoto:** A diferencia de los sistemas tradicionales donde los nodos deben cooperar para converger de forma lenta en una ruta, el controlador tiene un mapa completo de la red. Este programa calcula las rutas de manera óptima en nombre de todos los routers de la red y simplemente **inyecta las tablas de reenvío resultantes de forma remota**.

# Comunicación con el Plano de Datos (Interfaces "Southbound")

Para que la inteligencia centralizada pueda aplicar sus decisiones en el hardware real, se requiere un canal o protocolo de comunicación que actúe como interfaz. Históricamente, se han utilizado diversos enfoques:

- **RCP (Routing Control Platform - 2003):** Uno de los primeros desarrollos de control centralizado que utilizaba el propio protocolo BGP para dictar las rutas directamente con el fin de balancear tráfico y mitigar ataques DDoS.
- **Ethane (2007):** Empleó software centralizado para realizar autenticación y control de accesos de hosts en la red, aunque requería conmutadores físicos personalizados.
- **OpenFlow (2008):** Se consolidó como una interfaz estándar clave. Permite que el controlador escriba directamente en la memoria direccionable del switch (basada en TCAM). El conmutador físico simplemente consulta esta **tabla de coincidencia-acción** (_match-action table_) inyectada por el controlador central para saber qué hacer con cada paquete (reenviarlo por un puerto, descartarlo o enviarlo al propio controlador si no sabe cómo procesarlo).

# ¿Por qué centralizar el control? (Beneficios clave)
La centralización del plano de control ofrece ventajas disruptivas frente al control distribuido clásico:

- **Ingeniería de tráfico directa:** En lugar de realizar un ajuste manual, complejo e indirecto de los parámetros de la red (como modificar pesos de enlaces en OSPF esperando que el tráfico se redistribuya bien), el operador puede programar el controlador para que asigne rutas específicas de forma directa y predecible.
- **Innovación a la velocidad del software:** Elimina la dependencia de las actualizaciones lentas de hardware o de la tecnología propietaria de un fabricante (como Cisco o Juniper). Si el operador desea implementar un nuevo comportamiento de red, solo debe reprogramar el software del controlador.
- **Hardware simplificado y económico:** Al descargar la compleja CPU del router de tener que calcular rutas localmente (por ejemplo, ejecutando Dijkstra masivos), los dispositivos físicos pueden ser mucho más sencillos, rápidos y baratos (llamados _white boxes_ o conmutadores genéricos).
- **Uso de algoritmos avanzados:** Dado que el plano de control corre en servidores o centros de datos robustos, se pueden ejecutar algoritmos de optimización computacionalmente mucho más potentes e imposibles de implementar en enrutadores individuales estándar.
# Telemetría de red programable
Se considera una de las aplicaciones estrella de SDN. Supera las estadísticas agregadas tradicionales de IPFIX y el alto costo de almacenamiento de la captura masiva de paquetes:

- **INT (In-band Network Telemetry):** Permite que los paquetes de usuario recolecten y transporten información del estado de la red (como la latencia exacta experimentada en cada salto) a lo largo de su ruta.
- **Consultas MapReduce:** Permite a los operadores programar y particionar consultas de tráfico de alto nivel directamente en el hardware y software de conmutación.
- **Tráfico cifrado:** Ante la creciente encriptación en Internet, la telemetría programable ayuda a deducir de manera indirecta parámetros como la calidad de video analizando estadísticas físicas como el tamaño y tiempo entre llegadas de paquetes.

## Beneficios de SDN

- **Innovación veloz:** El desarrollo se produce a la velocidad del software en lugar de depender de actualizaciones de hardware.
- **Reducción de costos:** Permite simplificar y abaratar el equipamiento físico utilizando conmutadores más genéricos.
- **Algoritmos avanzados:** Facilita el uso de algoritmos de control de enrutamiento más complejos y exigentes computacionalmente.
- **Lanzamiento rápido:** Reduce drásticamente los ciclos de lanzamiento de nuevos servicios de red.

[[3 IPV4]]