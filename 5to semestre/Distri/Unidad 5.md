La **Unidad 5: Tiempo y Estados Globales** aborda uno de los problemas conceptuales más importantes de los sistemas distribuidos: la **ausencia de un reloj global físico** y de una **memoria compartida**.

Para poder coordinar actividades, determinar el orden de los eventos, detectar bloqueos o depurar fallos, los sistemas distribuidos deben recurrir a algoritmos de **sincronización de relojes físicos**, **relojes lógicos** y captura de **estados globales consistentes**.

---

### **1. CONCEPTOS FUNDAMENTALES: PROCESOS, EVENTOS Y RELOJES**

#### **A. Modelo de Procesos e Historias**

- **Colección de Procesos (\(\Omega\)):** El sistema distribuido está formado por un conjunto $\Omega = {p_1, p_2, \dots, p_N}$ de \(N\) procesos que se ejecutan en procesadores independientes sin memoria compartida.
- **Estado Local (\(s_i\)):** Es el conjunto de valores de todas las variables locales del proceso \(p_i\), pudiendo incluir recursos del sistema operativo local (como archivos abiertos).
- **Evento (\(e\)):** Ocurrencia de una única acción que realiza un proceso a medida que se ejecuta. Puede ser:
    1. Una acción de comunicación (`Envía` o `Recibe` un mensaje).
    2. Una acción de transformación de estado (cambia variables locales).
- **Orden Local (\(\to_i\)):** En un único proceso \(p_i\), la secuencia de eventos está estrictamente ordenada en forma lineal: \(e \to_i e'\) significa que \(e\) ocurrió antes que \(e'\) en \(p_i\).
- **Historia del Proceso (\(h_i\)):** La serie ordenada de todos los eventos que han tenido lugar en el proceso $p_i: h_i = \langle e_i^0, e_i^1, e_i^2, \dots \rangle$.

#### **B. Relojes Físicos y sus Imperfecciones**

- **Reloj Hardware (\(H_i(t)\)):** Contador basado en las oscilaciones de un cristal de cuarzo.
- **Reloj Software (\(C_i(t)\)):** Escala el valor del reloj hardware y añade una compensación para aproximar el tiempo real físico $t: C_i(t) = \alpha H_i(t) + \beta$.
- **Sesgo de Reloj (_Clock Skew_):** La diferencia instantánea entre las lecturas de tiempo de dos relojes cualesquiera.
- **Deriva de Reloj (_Clock Drift_):** Fenómeno por el cual los relojes cuentan el tiempo a ritmos ligeramente distintos debido a diferencias físicas en sus cristales, haciendo que sus lecturas se diverjan progresivamente.
    - _Relojes de cuarzo comunes:_ Tasa de deriva típica de \(10^{-6}\) seg/seg (1 segundo de diferencia cada 11,6 días).
    - _Relojes atómicos:_ Tasa de deriva de \(10^{-13}\), base del **TAI** (_Tiempo Atómico Internacional_).
- **UTC (_Tiempo Universal Coordinado_):** Estándar internacional de cronometraje basado en el tiempo atómico, al que se le insertan o borran "segundos de salto" periódicamente para mantenerse en sintonía con la rotación astronómica de la Tierra. Se difunde vía satélites y estaciones terrestres de radio (como la estación WWV).

---

### **2. SINCRONIZACIÓN DE RELOJES FÍSICOS**

Se requiere sincronizar los relojes físicos de los procesos para asociar marcas de tiempo reales a los eventos del sistema.

#### **A. Tipos de Sincronización**

- **Sincronización Externa:** Los relojes locales se sincronizan con una fuente externa de tiempo autorizada (UTC) con un límite de error \(D\):  
    $|S_i(t) - C_i(t)| < D$
- **Sincronización Interna:** Los nodos se sincronizan entre sí con una precisión \(D\), sin depender de una fuente externa de tiempo real:  
    \[|C_i(t) - C_j(t)| < D\]
- **Monotonicidad:** Propiedad esencial que impide que un reloj "retroceda" en el tiempo (\(t' > t \Rightarrow C(t') > C(t)\)). Si un reloj está adelantado, se debe desacelerar gradualmente su ritmo software en lugar de atrasar la hora bruscamente.

#### **B. Algoritmos de Sincronización Física**

1. **Método de Cristian (Probabilístico / Servidor Central):**
    
    - **Funcionamiento:** Un proceso cliente \(p\) solicita la hora a un **servidor de tiempo \(S\)** conectado a una fuente UTC. El cliente mide el tiempo total transcurrido de ida y vuelta (\(T_{round}\)) desde que envió la solicitud \(m_r\) hasta que recibió la respuesta \(m_t\).
    - **Estimación:** El cliente ajusta su reloj a \(t_p = t_{server} + \frac{T_{round}}{2}\), asumiendo que el tiempo de tránsito fue simétrico en ambas direcciones.
    - **Naturaleza probabilística:** Solo se logra el grado de precisión requerido si \(T_{round}\) es lo suficientemente corto.
2. **Algoritmo de Berkeley (Sincronización Interna / Maestro-Esclavo):**
    
    - **Funcionamiento:** No utiliza un servidor UTC externo. Elige un nodo como **Maestro (_Time Daemon_)** que consulta periódicamente la hora a los demás nodos (**Esclavos**).
    - **Cálculo:** El maestro mide los retardos de red, calcula un **promedio de tiempo** (descartando lecturas con diferencias anómalas) y le responde a cada esclavo indicándole el **ajuste relativo (+/-)** que debe aplicar individualmente para alinearse.
3. **NTP (_Network Time Protocol_):**
    
    - **Propósito:** Diseñado para distribuir el tiempo UTC a escala global en Internet sobre el protocolo **UDP**.
    - **Estructura en Estratos (_Strata_):** Organiza los servidores de tiempo en una subred de jerarquía lógica:
        - **Estrato 1:** Servidores primarios (conectados directamente a relojes atómicos o GPS).
        - **Estrato 2:** Servidores secundarios sincronizados directamente con los del Estrato 1.
        - **Estrato 3:** Servidores sincronizados con los del Estrato 2, y así sucesivamente.
    - **Objetivos:** Resistir pérdidas prolongadas de conectividad, soportar resincronizaciones frecuentes y proteger el servicio contra interferencias o ataques maliciosos.

---

### **3. TIEMPOS Y RELOJES LÓGICOS**

Dado que los relojes físicos no pueden sincronizarse perfectamente en un sistema distribuido, Leslie Lamport demostró que el tiempo físico no sirve para determinar el orden de cualquier par arbitrario de eventos. En su lugar, se emplea la **ordenación causal de eventos** (tiempo lógico).

#### **A. La Relación "Sucedió Antes" (\(\to\)) de Lamport**

Determina la relación de causalidad potencial entre dos eventos \(e\) y \(e'\) mediante tres reglas esenciales:

1. **Regla SA1 (Secuencia local):** Si \(e\) y \(e'\) ocurren dentro del mismo proceso \(p_i\) y \(e\) ocurre antes que \(e'\), entonces \(e \to e'\).
2. **Regla SA2 (Paso de mensajes):** Para cualquier mensaje \(m\), el evento de enviarlo siempre precede al evento de recibirlo: \(\text{envía}(m) \to \text{recibe}(m)\).
3. **Regla SA3 (Transitividad):** Si \(e \to e'\) y \(e' \to e''\), entonces \(e \to e''\).

- **Eventos Concurrentes (\(e \parallel e'\)):** Si no existe una relación de causa-efecto entre dos eventos (es decir, ni \(e \to e'\) ni \(e' \to e\)), se afirma que \(e\) y \(e'\) son concurrentes.

#### **B. Relojes Lógicos de Lamport**

Un reloj lógico de Lamport es un contador software mono-tónicamente creciente \(L_i\) que gestiona cada proceso \(p_i\).

- **Reglas de Actualización:**
    - **RL1:** Antes de que ocurra cualquier evento en \(p_i\), el proceso incrementa su contador: \(L_i = L_i + 1\).
    - **RL2:**
        - (a) Al enviar un mensaje \(m\), el proceso le adjunta la marca de tiempo \(t = L_i\).
        - (b) Al recibir el mensaje \((m, t)\), el proceso receptor actualiza su contador local tomando el máximo entre su valor actual y el recibido, e incrementándolo en uno: \(L_j = \max(L_j, t) + 1\).
- **Propiedad y Deficiencia Clave:**
    - Si \(e \to e' \Rightarrow L(e) < L(e')\) (**Cumple sentido directo**).
    - **Deficiencia:** Del hecho que \(L(e) < L(e')\) **NO se puede inferir** que \(e \to e'\).

#### **C. Relojes Totalmente Ordenados**

Para eliminar empates cuando eventos en procesos distintos obtienen la misma marca de tiempo \(L(e)\), se crea un **orden total** combinando el valor del reloj con el identificador del proceso: \((T_i, i)\). Se define que \((T_i, i) < (T_j, j)\) si y solo si \(T_i < T_j\), o si \(T_i = T_j\) siendo \(i < j\).

#### **D. Relojes Vectoriales (Mattern y Fidge)**

Diseñados para corregir la deficiencia de los relojes de Lamport y permitir deducir la causalidad en **ambas direcciones**.

- **Estructura:** Para un sistema de \(N\) procesos, cada proceso \(p_i\) mantiene un vector \(V_i\) de \(N\) enteros.
- **Reglas de Actualización (RV1–RV4):**
    1. **RV1:** Inicialmente, \(V_i[j] = 0\) para todo \(j = 1, \dots, N\).
    2. **RV2:** Justo antes de registrar un evento local, \(p_i\) incrementa su propio componente: \(V_i[i] = V_i[i] + 1\).
    3. **RV3:** Al enviar un mensaje, \(p_i\) incluye en él su vector completo \(t = V_i\).
    4. **RV4:** Al recibir un mensaje con marca \(t\), el proceso \(p_i\) combina elemento por elemento tomando el máximo: \(V_i[j] = \max(V_i[j], t[j])\) para todo \(j\), e incrementa su componente local \(V_i[i]\).
- **Propiedad Bidireccional:**  
    \[e \to e' \iff V(e) < V(e')\]  
    Si los vectores no se pueden comparar (\(V(e) \not\le V(e')\) y \(V(e') \not\le V(e)\)), se confirma de manera matemática que los eventos son concurrentes (\(e \parallel e'\)).

---

### **4. ESTADOS GLOBALES Y CORTES CONSISTENTES**

Un **estado global** representa una "fotografía" conjunta de los estados de todos los procesos y de los canales de comunicación en un momento dado.

#### **A. Aplicaciones de la Captura del Estado Global**

1. **Garbage Collection Distribuido:** Detectar objetos en memoria que ya no tienen referencias en ningún proceso ni en ningún mensaje en tránsito por la red.
2. **Detección Distribuida de Interbloqueos (_Deadlocks_):** Identificar ciclos en el grafo de "espera por" (_wait-for_) entre procesos.
3. **Detección de la Terminación Distribuida:** Verificar que un algoritmo distribuido ha finalizado cuando todos los procesos están pasivos y no quedan mensajes viajando por los canales.

#### **B. Cortes de Ejecución y Consistencia**

- **Historia Global (\(H\)):** Es la unión de las historias individuales de todos los procesos: \(H = h_1 \cup h_2 \dots \cup h_N\).
- **Corte (\(C\)):** Es un subconjunto de la historia global formado por la unión de prefijos de las historias de cada proceso. La **frontera del corte** son los últimos eventos incluidos en cada proceso.
- **Corte Consistente:** Un corte \(C\) se considera **consistente** si para cualquier evento \(e\) que pertenezca al corte, **todos los eventos que sucedieron antes que él (\(e' \to e\)) también están incluidos dentro del corte**.
- **Estado Global Consistente:** Es aquel estado que corresponde a un corte consistente. En él **no existen mensajes "huérfanos"** (mensajes recibidos en la frontera cuyo evento de envío no haya ocurrido antes del corte).

---

### **5. ALGORITMO DE INSTANTÁNEA DE CHANDY Y LAMPORT (1985)**

Permite determinar un **estado global consistente** de procesos y canales en plena ejecución sin necesidad de congelar el sistema.

#### **A. Supuestos del Algoritmo**

1. Ni los procesos ni los canales fallan; la comunicación es 100% fiable.
2. Los canales son unidireccionales y garantizan un orden de entrega **FIFO** (_First-In, First-Out_).
3. El grafo de la red está fuertemente conectado.
4. Cualquier proceso puede iniciar la instantánea en cualquier momento.
5. Los procesos continúan ejecutándose y transmitiendo mensajes normales mientras transcurre la instantánea.

#### **B. Mecanismo de Mensajes Marcadores (_Markers_)**

El algoritmo utiliza un mensaje de control especial llamado **Marcador (_Marker_)**, que fluye por los canales entre los mensajes normales. Se rige por dos reglas fundamentales:

1. **Regla de Recepción del Marcador (para el proceso \(p_i\) al recibirlo por el canal \(c\)):**
    
    - **Si \(p_i\) AÚN NO ha grabado su estado:** Guarda su estado local inmediatamente, registra el estado del canal \(c\) como un conjunto vacío (\(\emptyset\)), y activa la grabación de todos los mensajes que lleguen por los demás canales entrantes.
    - **Si \(p_i\) YA HABÍA grabado su estado previamente:** Registra el estado del canal \(c\) guardando todos los mensajes que han llegado por \(c\) desde el momento en que salvó su propio estado local.
2. **Regla de Envío del Marcador (para el proceso \(p_i\)):**
    
    - Inmediatamente después de guardar su estado local, \(p_i\) envía un mensaje **Marcador** por **cada uno de sus canales salientes** (antes de enviar cualquier otro mensaje de aplicación).

---

### **6. DEPURACIÓN DISTRIBUIDA Y EVALUACIÓN DE PREDICADOS**

La depuración distribuida analiza la secuencia de estados globales por los que atraviesa la aplicación (\(S_0 \to S_1 \to S_2 \dots\)) para detectar errores.

#### **A. Predicados de Estado Global (\(\Phi\))**

Un predicado es una función lógica que evalúa si una condición del sistema es verdadera o falsa sobre un estado global. Se caracterizan por:

- **Estabilidad:** Si el predicado toma valor Verdadero en un estado, **permanecerá Verdadero** en todos los estados alcanzables posteriores (ejemplo: interbloqueo o terminación).
- **Seguridad (_Safety_):** El predicado debe evaluar a Falso para cualquier estado alcanzable desde \(S_0\) (sirve para verificar que nunca se entre en estados anómalos o de error).
- **Vitalidad (_Liveness_):** El predicado evalúa a Verdadero en algún estado alcanzable desde \(S_0\) (garantiza que el sistema "progrese" hacia una situación necesaria).

#### **B. Monitorización y Evaluaciones de Certeza**

- **Monitorización Centralizada (Marzullo y Neiger):** Los procesos envían periódicamente sus estados a un **proceso monitor**, el cual construye una red de estados globales consistentes y evalúa las **linealizaciones** (rutas válidas entre estados).
- **Modos de Evaluación de Predicados:**
    - **Posiblemente \(\Phi\) (_Possibly \(\Phi\)_):** Significa que existe **al menos una linealización** (secuencia de estados) que pasa por un estado consistente \(S\) donde \(\Phi(S)\) es Verdadero.
    - **Sin Duda Alguna \(\Phi\) (_Definitely \(\Phi\)_):** Significa que para **TODAS las linealizaciones posibles**, el sistema obligatoriamente pasa por algún estado consistente \(S\) donde \(\Phi(S)\) es Verdadero.

---

### **7. ESQUEMA RESUMEN PARA MEMORIZACIÓN RÁPIDA**

Para repasar la Unidad 5 antes de un examen, memoriza esta estructura en 5 bloques:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       UNIDAD 5: TIEMPO Y ESTADOS GLOBALES                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ 1. SINCRONIZACIÓN FÍSICA                                                    │
│    • Externa (|Si - Ci| < D, con UTC) vs. Interna (|Ci - Cj| < D, sin UTC).  │
│    • Cristian: Servidor UTC + Medición Tround -> t_server + Tround/2.       │
│    • Berkeley: Demonio Maestro pide horas, promedia y manda ajustes (+/-).  │
│    • NTP: Subred jerárquica en estratos (1, 2, 3...) sobre UDP.             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 2. TIEMPO LÓGICO Y RELOJES                                                  │
│    • Lamport ("Sucedió antes" ->): RL1 (inc. local) y RL2 (max + inc.).     │
│      -> Ojo: e -> e' => L(e) < L(e') (Solo sentido directo).                │
│    • Orden Total: Desempata marcas idénticas mediante (Ti, i).              │
│    • Vectoriales (Mattern/Fidge): Vector Vi de N enteros.                   │
│      -> Validez total: e -> e' <=> V(e) < V(e'). Detecta concurrencia.      │
├─────────────────────────────────────────────────────────────────────────────┤
│ 3. ESTADOS GLOBALES Y CORTES                                                │
│    • Usos: Garbage Collection, Detección de Deadlocks y Terminación.        │
│    • Corte Consistente: Si incluye e, incluye TODO e' que ocurrió antes.    │
├─────────────────────────────────────────────────────────────────────────────┤
│ 4. ALGORITMO DE CHANDY Y LAMPORT (INSTANTÁNEA)                              │
│    • Canales FIFO y fiables. Usa mensajes Marcadores (Markers).             │
│    • Si no he grabado estado -> Grabo estado, vacíocanal c, transmito Marc  │
│      por salidas y grabo entradas de otros canales.                         │
│    • Si ya grabé estado -> Grabo en canal c lo recibido desde mi grabación. │
├─────────────────────────────────────────────────────────────────────────────┤
│ 5. DEPURACIÓN DISTRIBUIDA                                                   │
│    • Predicados Φ: Estabilidad, Seguridad (Safety) y Vitalidad (Liveness).  │
│    • Posiblemente Φ (existe 1 ruta que cumple) vs. Sin duda alguna Φ (todas │
│      las rutas cumplen).                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

