# Capítulo 1: Empezando (Getting Started)

Este capítulo introduce las herramientas y flujos de trabajo básicos para programar en C y C++ en el entorno Linux.
#### **1. Edición de Código con Emacs**

- **GNU Emacs** es el editor más popular y completo en entornos GNU/Linux.
- **Formatos e Indentación automática:** Al abrir un archivo con extensión `.c`, `.h` (para C) o `.cpp`, `.hpp` (para C++), Emacs detecta automáticamente el lenguaje. Al presionar la tecla **Tab**, indenta el código de manera inteligente según su sintaxis.
- **Resaltado de Sintaxis:** Facilita la legibilidad y la detección de errores. Se activa globalmente agregando `(global-font-lock-mode t)` en el archivo de configuración `~/.emacs`.
- **Atajos básicos:** Abrir archivo (`C-x C-f`), Guardar (`C-x C-s`) y Salir (`C-x C-c`).

#### **2. Compilación con GCC y G++**

- **Diferenciación:** Se usa `gcc` para C y `g++` para C++. Siempre se debe usar `g++` para el paso de enlazado si el proyecto contiene partes en C++.
- **Banderas fundamentales del compilador:**
    - `-c`: Compila el código fuente a código objeto (genera archivos `.o`) sin realizar el enlace final.
    - `-o <nombre>`: Especifica el nombre del archivo ejecutable de salida.
    - `-I<directorio>`: Indica al preprocesador dónde buscar archivos de cabecera adicionales (headers).
    - `-D <MACRO>`: Define macros desde la línea de comandos (ej. `-D NDEBUG` para desactivar aserciones de depuración).
    - `-O2`: Activa el segundo nivel de optimización de código para entornos de producción.
    - `-g`: Incluye información de depuración necesaria para el uso de GDB.
    - `-L<directorio>` y `-l<nombre>`: Indican al enlazador dónde buscar las bibliotecas adicionales y con cuáles enlazarse (ej. `-lpam` enlaza con la biblioteca `libpam.a` o `.so`, eliminando automáticamente el prefijo `lib` y el sufijo).

#### **3. Automatización con GNU Make**

- Evita tener que compilar manualmente múltiples archivos cada vez que se hace un cambio.
- **Estructura del `Makefile`:** Se definen **objetivos (targets)**, **dependencias** y **reglas** para construirlos.
- **Regla de Oro:** La línea de comandos que contiene la regla de compilación **debe comenzar obligatoriamente con un carácter Tab (tabulador)**, de lo contrario `make` fallará.
- **Objetivo `clean`:** Es una convención estándar para limpiar archivos temporales u objetos construidos con el fin de recompilar todo desde cero.

#### **4. Depuración con GDB**

- **Preparación:** Requiere compilar el ejecutable con la opción `-g`.
- **Comandos clave de GDB:**
    - `run`: Ejecuta el programa.
    - `where` / `backtrace`: Muestra la pila de llamadas (_stack_) activa en el momento de un fallo (útil para diagnosticar errores de segmentación o _segmentation faults_).
    - `up`: Sube un nivel en la pila de llamadas para examinar el contexto del código que invocó la función que falló.
    - `print <variable>`: Imprime el valor actual de una variable.
    - `break <ubicación>`: Establece un punto de interrupción (_breakpoint_).
    - `next` y `step`: Avanzan en la ejecución línea por línea; `next` pasa por encima de las llamadas a funciones (_step over_), mientras que `step` ingresa dentro de ellas (_step into_).

# Capítulo 2: Escribiendo Buen Software en GNU/Linux

Este capítulo detalla las mejores prácticas y API necesarias para interactuar adecuadamente con el sistema operativo y escribir programas robustos y profesionales.

#### **1. Interacción con el Entorno de Ejecución**

- **Argumentos de línea de comandos:** Se acceden en la función `main` mediante `argc` (cantidad de argumentos) y `argv` (arreglo de cadenas de texto). El primer elemento, `argv`, es siempre el nombre del programa.
- **Uso de `getopt_long`:** Es la función estándar (de `<getopt.h>`) que simplifica el análisis (_parsing_) de opciones cortas (ej. `-h`) y largas (ej. `--help`). Permite usar variables globales del sistema como `optarg` (para capturar argumentos de opciones) y `optind` (para saber dónde comienzan los parámetros que no son opciones).
- **E/S Estándar y Redirección:**
    - Flujos estándar: `stdin` (fd 0), `stdout` (fd 1, con buffer) y `stderr` (fd 2, sin buffer).
    - Como `stdout` almacena datos en un búfer antes de imprimirlos en pantalla, se debe usar **`fflush(stdout)`** para forzar la salida inmediata en pantalla en momentos críticos.
    - La sintaxis `2>&1` en el shell une el canal de errores con el canal de salida estándar.
- **Códigos de salida:** Se especifican retornando un entero en `main` o con la llamada `exit()`. Por convención, `0` indica éxito y valores distintos de cero indican errores. El rango recomendado es de `0` a `127`.
- **El Entorno:** Colección de variables de tipo clave-valor (como `PATH` o `USER`). Se consultan individualmente con `getenv()` y se modifican con `setenv()` y `unsetenv()`. Alternativamente, se puede recorrer el entorno completo con la variable global `extern char** environ`.
- **Archivos Temporales:** Para evitar problemas de seguridad y colisión de nombres en `/tmp`, se deben evitar funciones obsoletas como `mktemp` o `tmpnam`.
    - Se recomienda **`mkstemp`**: genera un archivo único con permisos restrictivos (solo lectura/escritura para el dueño) a partir de una plantilla terminada en "XXXXXX". Es una excelente práctica llamar a `unlink()` sobre el archivo inmediatamente después de abrirlo; esto asegura que el archivo se eliminará del disco automáticamente cuando se cierre el descriptor o finalice el programa.
    - O bien **`tmpfile`**, que devuelve directamente un flujo `FILE*` de biblioteca estándar que ya está desvinculado (_unlinked_) y se borra solo al cerrarse.

#### **2. Programación Defensiva**

- **Uso de `assert`:** Macro de `<assert.h>` que evalúa expresiones lógicas y aborta dramáticamente el programa si la condición es falsa, ayudando a detectar errores internos tempranos. En producción, estas comprobaciones se pueden desactivar con la macro `NDEBUG` (compilando con `-DNDEBUG`). **Importante:** Jamás se deben colocar llamadas con efectos secundarios (como incremento de variables `++` o llamadas críticas a funciones) dentro de un `assert`, ya que si se desactiva `NDEBUG`, ese código no se ejecutará.
- **Manejo de Errores de Llamadas al Sistema:** Las llamadas al sistema pueden fallar por recursos agotados, falta de permisos o interrupciones de señales. En caso de fallo, almacenan el código de error en la variable global **`errno`** (de `<errno.h>`). Para informar estos errores, se utiliza `strerror(errno)` (devuelve la descripción del error) o `perror()` (la imprime directo en `stderr`).
- **Evitar fugas de recursos (leaks):** Si una función retorna anticipadamente debido a un fallo en una llamada intermedia, debe asegurarse de liberar toda la memoria dinámica asignada y cerrar todos los descriptores de archivos abiertos con anterioridad en esa misma función.

#### **3. Creación y Uso de Bibliotecas**

- **Bibliotecas Estáticas (`.a`):** Son colecciones de archivos objeto empaquetados con el comando `ar cr`. Al enlazar, solo se extraen las funciones requeridas por los objetos procesados hasta ese punto. Por lo tanto, las bibliotecas estáticas **deben colocarse siempre al final de la línea de comandos de compilación**.
- **Bibliotecas Compartidas (`.so`):** No copian su código en el ejecutable final, sino que se enlazan de forma dinámica en tiempo de ejecución.
    - **PIC (Position-Independent Code):** Los archivos objeto que las componen deben compilarse de manera independiente de su posición en memoria usando el parámetro **`-fPIC`**. El archivo final se enlaza con `gcc -shared -fPIC -o libtest.so test1.o test2.o`.
    - **Rutas de búsqueda:** En tiempo de ejecución, el sistema busca estas bibliotecas en `/lib` y `/usr/lib` por defecto. Rutas personalizadas pueden definirse en la compilación usando `-Wl,-rpath,<ruta>` o configurando la variable de entorno `LD_LIBRARY_PATH`.
- **Carga Dinámica en Tiempo de Ejecución (Plugins):** Permite cargar código de forma dinámica sobre la marcha. Se implementa mediante las funciones **`dlopen()`** (para cargar el archivo `.so`), **`dlsym()`** (para obtener un puntero a una función o variable de la biblioteca) y **`dlclose()`** (para descargarla). En C++, las funciones expuestas para `dlsym` deben declararse con `extern "C"` para evitar la alteración de nombres (_name mangling_) por parte del compilador.
