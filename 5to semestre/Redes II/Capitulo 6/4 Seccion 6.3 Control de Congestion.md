Aborda la gestión de la carga en la red desde la perspectiva de la capa de transporte. Aunque la congestión ocurre físicamente en los enrutadores de la capa de red, es provocada por el volumen de datos que inyecta la capa de transporte. Por ello, el control de congestión es una **responsabilidad combinada** entre ambas capas, donde la única solución efectiva es que la entidad de transporte reduzca la velocidad a la que envía sus paquetes.
# Asignación deseable de ancho de banda (Sección 6.3.1)

El objetivo de un algoritmo de control de congestión no es solo evitar el colapso de la red, sino encontrar una asignación de ancho de banda para las entidades de transporte que cumpla con tres criterios fundamentales: **eficiencia**, **equidad** y **convergencia rápida**.

>[!info] Explicacion de cada uno:
>- **Eficiencia y Potencia:** Busca maximizar la tasa de entrega útil de paquetes (_goodput_) manteniendo el retardo bajo. Si la carga ofrecida se aproxima demasiado a la capacidad máxima de la red, las ráfagas de tráfico llenan los búferes de los routers y provocan pérdidas de paquetes y demoras exponenciales. Esto puede derivar en un **colapso de congestión**, donde los emisores retransmiten continuamente datos sin lograr trabajo útil.
>- **Equidad Máxima-Mínima (_Max-Min Fairness_):** Es el criterio utilizado para dividir el ancho de banda entre flujos que compiten por los mismos enlaces. Una asignación es justa en sentido máximo-mínimo si **no es posible aumentar el ancho de banda de un flujo sin reducir el de otro flujo que tenga una asignación igual o menor**. En la práctica, este criterio garantiza que ninguna conexión se quede sin ancho de banda (_starvation_).
>- **Convergencia:** Debido a que las conexiones se abren, se cierran y cambian su demanda en el tiempo, la asignación ideal es dinámica. El algoritmo de control debe **converger rápidamente** al punto de operación óptimo y rastrearlo de forma estable sin oscilar drásticamente alrededor de él.
# Regulación de la tasa de envío (Sección 6.3.2)

Para ajustar la velocidad de transmisión, la capa de transporte debe distinguir entre dos problemas que causan la pérdida de datos pero requieren soluciones opuestas:

1. **Control de flujo:** La velocidad se limita porque el **receptor es lento** y no tiene espacio en sus búferes.
2. **Control de congestión:** La velocidad se limita porque la **red intermedia es lenta** o está sobrecargada.
#### Señales de retroalimentación

Para saber cuándo frenar, la capa de transporte recopila señales de la red. Estas señales pueden ser explícitas (como bits **ECN** marcados por los routers en las cabeceras o mensajes de estrangulamiento) o implícitas (medición del aumento en el tiempo de ida y vuelta **RTT** o la detección de **pérdida de paquetes**).
#### La ley de control AIMD (_Additive Increase, Multiplicative Decrease_)

Chiu y Jain (1989) demostraron que ante señales de congestión binarias, la única ley de control que garantiza la **convergencia hacia una asignación equitativa y eficiente** es **AIMD**:

- **Incremento Aditivo (AI):** Cuando la red no está congestionada, los usuarios aumentan linealmente su tasa de envío a un ritmo suave (con una pendiente constante a 45° hacia la línea de eficiencia) para sondear el ancho de banda libre sin desestabilizar el sistema.
- **Decremento Multiplicativo (MD):** Al detectar congestión, los usuarios reducen drásticamente su tasa de envío en una proporción o porcentaje (apuntando al origen en el espacio de fase).
- Las alternativas como incrementar o reducir de forma puramente multiplicativa o aditiva (AIAD, MIMD, MIAD) no logran converger al punto óptimo de la red.
#### Aplicación en Internet y compatibilidad

En Internet, el protocolo **TCP implementa AIMD de manera indirecta** ajustando el tamaño de una **ventana de congestión (\(cwnd\))** mediante un reloj impulsado por los acuses de recibo (ACKs). El mecanismo presenta un sesgo natural: las conexiones con un RTT más corto reciben confirmaciones más rápido y hacen crecer su ventana con mayor velocidad que las conexiones lejanas. Debido a que TCP domina el tráfico en la red, se exige que cualquier nuevo protocolo de transporte sea **compatible con TCP (_TCP-friendly_)** para evitar que acapare el ancho de banda al competir con flujos TCP estándar.
# Cuestiones inalámbricas (Sección 6.3.3)

Los protocolos de control de congestión de la capa de transporte asumen tradicionalmente que cualquier pérdida de paquetes es el síntoma de un router saturado. Sin embargo, en los **enlaces inalámbricos**, la mayoría de las pérdidas se deben a **errores de transmisión de radio o ruidos en el canal** y no a congestión en la red.

Si un protocolo como TCP confunde un error de canal inalámbrico con congestión, reduce erróneamente su ventana de envío, lo que provoca una **degradación severa e innecesaria del rendimiento** en redes móviles o Wi-Fi.

