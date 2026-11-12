#### ¿Qué es una Transacción?

En los sistemas distribuidos y bases de datos, una **transacción** se define como una secuencia de operaciones realizada por un servidor sobre objetos o datos, la cual se garantiza que se ejecuta de manera **atómica**. Esto significa que el servidor asegura que la transacción se completa en su totalidad o no se ejecuta en absoluto, manteniendo los efectos parciales invisibles para otras transacciones concurrentes.

El ciclo de vida de una transacción es gestionado por un **coordinador**, el cual le asigna un identificador único (TID) y expone tres operaciones principales:

- **`openTransaction()`**: Inicia la transacción y devuelve un identificador de transacción (TID).
- **`closeTransaction()`**: Solicita la finalización de la transacción, resultando en la confirmación de los cambios (_commit_) o en el aborto de los mismos (_abort_).
- **`abortTransaction()`**: Cancela expresamente la transacción y elimina todos sus efectos.
### **Propiedades ACID de una Transacción**

Para garantizar que el manejo de datos en un entorno distribuido sea confiable, toda transacción debe cumplir con las cuatro **propiedades ACID**:

1. **Atomicidad (_Atomicity_):** Garantiza una regla de "todo o nada". Si ocurre un error o fallo en cualquier paso del proceso, todos los cambios parciales realizados hasta el momento se revierten por completo.
2. **Consistencia (_Consistency_):** Asegura que la transacción lleve al sistema de un estado válido a otro estado también válido, respetando en todo momento las reglas e invariantes del sistema (por ejemplo, evitar saldos negativos en una cuenta).
3. **Aislamiento (_Isolation_):** Exige que cada transacción se ejecute de forma independiente sin interferencias de otras transacciones concurrentes. Los resultados intermedios no son visibles para otras operaciones hasta que la transacción se consuma por completo.
4. **Durabilidad (_Durability_):** Asegura que una vez que la transacción ha sido confirmada (_commit_), sus cambios permanecen guardados en almacenamiento permanente, incluso si el servidor se apaga o sufre una caída inmediatamente después.
#### Que es una sincronizacion simple?
La **sincronización simple (sin transacciones)** es un mecanismo para asegurar que las operaciones ejecutadas en nombre de diferentes clientes no interfieran entre sí en un entorno multihilo, evitando que se generen valores incorrectos o inconsistentes en los objetos compartidos.

Los aspectos clave de este mecanismo incluyen:
##### Exclusión mutua y operaciones atómicas

- **Interferencia en multihilo:** Cuando múltiples hilos acceden concurrentemente a métodos no diseñados para entornos multihilo, sus acciones pueden entremezclarse de forma arbitraria y alterar las variables de instancia de los objetos.
- **Bloqueo con `synchronized`:** En Java, la palabra clave `synchronized` garantiza que **solo un hilo pueda acceder a un objeto a la vez**. Al invocar un método sincronizado, el hilo bloquea el objeto; otros hilos que intenten acceder deben esperar a que dicho bloqueo sea liberado.
- **Ejecución atómica:** Esta separación en el tiempo garantiza que las variables de instancia se accedan de forma consistente. Las operaciones libres de interferencia por parte de otros hilos se consideran **operaciones atómicas**.
##### Coordinación entre hilos (`wait` y `notify`)

- **Colaboración de clientes:** En situaciones donde una operación requiere que otro hilo realice una acción previa (como en esquemas de _productor-consumidor_ o al liberar un recurso compartido), la exclusión mutua por sí sola no basta.
- **`wait()`:** Permite que un hilo suspenda su propia ejecución dentro de un método sincronizado y **libere de forma atómica su bloqueo**, dando paso a que otro hilo ejecute.
- **`notify()`:** Informa a los hilos en espera que los datos del objeto han cambiado, permitiéndoles activarse, adquirir un nuevo bloqueo y reanudar la ejecución.

### **Diferencia con el manejo de transacciones**

Mientras que la sincronización simple solo protege la integridad de las variables en la memoria volátil del servidor durante la ejecución, **las transacciones garantizan propiedades ACID completas**, asegurando además que los cambios se almacenen en memoria permanente y se puedan revertir en caso de caídas del servidor.

### **¿Qué es una Transacción Simple (Plana o _Flat Transaction_)?**

Una **transacción simple** (conocida también como **transacción plana** o _flat transaction_) es la forma básica de estructura transaccional en la cual una secuencia de operaciones se ejecuta a un **único nivel lineal**, sin divisiones internas.

Sus características clave incluyen:

- **Estructura plana sin jerarquías:** A diferencia de las _transacciones anidadas_ (las cuales contienen subtransacciones jerárquicas que pueden ejecutarse o fallar de forma independiente), una transacción simple abarca todas sus operaciones en un único bloque directo.
- **Manejo de fallos del tipo "todo o nada":** Si una sola operación dentro de la transacción simple falla en cualquier punto, toda la transacción debe abortar y reiniciarse por completo desde el principio.
- **Contraste con la sincronización simple:** El material distingue la transacción simple de la **sincronización simple sin transacciones** (basada únicamente en exclusión mutua de hilos con bloques `synchronized`, `wait()` y `notify()` de Java). Mientras la sincronización simple solo previene interferencias inmediatas entre hilos en memoria volátil, una transacción simple aplica las garantías **ACID** completas y asegura la recuperación del estado ante caídas del servidor.