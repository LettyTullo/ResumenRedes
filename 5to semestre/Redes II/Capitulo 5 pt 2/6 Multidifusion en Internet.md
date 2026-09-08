La **multidifusión (multicast)** es un modelo de entrega de paquetes donde un único emisor transmite un mensaje de forma simultánea a un **grupo específico de receptores interesados**, en lugar de a un solo destino (_unicast_) o a absolutamente todos los dispositivos de la red (_broadcast_).

Es el método ideal para aplicaciones multimedia en tiempo real, como transmisiones de video en vivo (IPTV), videoconferencias o juegos multijugador, ya que evita que el emisor tenga que duplicar y enviar el mismo archivo miles de veces, ahorrando un inmenso ancho de banda en la red.
Para entender su funcionamiento en Internet de manera sencilla, el proceso se apoya en tres pilares:
#### 1. Las Direcciones de Grupo (Clase D)
Para que los receptores puedan identificarse como miembros de un grupo común, la multidifusión utiliza un rango especial de direcciones IP IPv4 conocidas como **direcciones de Clase D** (que abarcan desde la `224.0.0.0` hasta la `239.255.255.255`).

- **Ámbito local:** El rango `224.0.0.0/24` está reservado exclusivamente para la red local (LAN). Aquí no se requiere enrutamiento; por ejemplo, la IP `224.0.0.1` se usa para enviar un mensaje a todos los hosts de la LAN, y la `224.0.0.2` a todos los routers locales.

- **Ámbito global:** Las demás direcciones de Clase D permiten agrupar a miembros que se encuentran en diferentes redes a lo largo de Internet, lo cual sí requiere el trabajo de los routers para mover los paquetes entre redes.

#### 2. ¿Cómo se une un dispositivo a un grupo? (IGMP)

Cuando una aplicación en tu computadora quiere recibir el contenido de un grupo de multidifusión, el host debe avisarle a su router local que desea unirse. Esto lo hace a través del protocolo **IGMP (Internet Group Management Protocol)**.

- El router multicast envía consultas periódicas (aproximadamente una vez por minuto) a todos los dispositivos de su LAN para saber a qué grupos pertenecen.

- Los dispositivos interesados responden indicando las direcciones de Clase D que quieren escuchar.

- Si el último dispositivo de la LAN abandona un grupo, el router se entera y deja de solicitar ese tráfico para no saturar la red local de forma innecesaria.

#### 3. El enrutamiento de los paquetes por la red (PIM)

Una vez que los routers saben dónde están los usuarios interesados, deben construir de forma dinámica un camino eficiente (un árbol de expansión sin bucles) para hacerles llegar los paquetes desde el emisor original.

El protocolo estándar y más utilizado hoy en día dentro de las redes de los operadores es **PIM (Protocol Independent Multicast)**, el cual funciona principalmente en dos modos según cómo estén distribuidos los receptores:

- **Modo Denso (PIM-DM):** Se utiliza cuando los miembros del grupo están muy concentrados por toda la red (por ejemplo, al distribuir archivos a muchos servidores en un centro de datos). El router asume que todos quieren el paquete, lo envía a todas partes y luego "poda" (_prune_) recursivamente las ramas de la red donde los routers vecinos le avisen mediante mensajes `PRUNE` que no tienen hosts interesados.

- **Modo Disperso (PIM-SM):** Se utiliza cuando los miembros están muy separados geográficamente en Internet (como los abonados de televisión de un ISP). En lugar de inundar la red, se utiliza un router central como **núcleo** (_core_ o punto de encuentro). Los routers de los usuarios interesados deben pedirle explícitamente al núcleo unirse al árbol, de modo que el tráfico solo viaja por las ramas que realmente lo solicitaron.