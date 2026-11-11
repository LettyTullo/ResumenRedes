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

### **¿Qué es una Transacción Simple (Plana o _Flat Transaction_)?**

Una **transacción simple** (conocida también como **transacción plana** o _flat transaction_) es la forma básica de estructura transaccional en la cual una secuencia de operaciones se ejecuta a un **único nivel lineal**, sin divisiones internas.

Sus características clave incluyen:

- **Estructura plana sin jerarquías:** A diferencia de las _transacciones anidadas_ (las cuales contienen subtransacciones jerárquicas que pueden ejecutarse o fallar de forma independiente), una transacción simple abarca todas sus operaciones en un único bloque directo.
- **Manejo de fallos del tipo "todo o nada":** Si una sola operación dentro de la transacción simple falla en cualquier punto, toda la transacción debe abortar y reiniciarse por completo desde el principio.
- **Contraste con la sincronización simple:** El material distingue la transacción simple de la **sincronización simple sin transacciones** (basada únicamente en exclusión mutua de hilos con bloques `synchronized`, `wait()` y `notify()` de Java). Mientras la sincronización simple solo previene interferencias inmediatas entre hilos en memoria volátil, una transacción simple aplica las garantías **ACID** completas y asegura la recuperación del estado ante caídas del servidor.