# Prácticas de Vulnerabilidades Hardware con ChipWhisperer

Este repositorio contiene el material práctico y de laboratorio para el estudio de **Vulnerabilidades Hardware**, centrado en técnicas de **Análisis por Canales Laterales** (*Side-Channel Analysis* - SCA) y técnicas de **Inyección de Fallos** (*Fault Injection Attacks* - FIA), utilizando la plataforma de bajo coste **ChipWhisperer-Nano** (NewAE Technology).

El material está estructurado en una serie de cuadernos interactivos en formato **Jupyter Notebook** (`.ipynb`), redactados y adaptados didácticamente en español, acompañados del código fuente de firmware en C y utilidades necesarias para reproducir los ataques sobre microcontroladores reales (ARM Cortex-M0 en STM32).

---

## 📋 Tabla de Contenidos

1. [Estructura del Repositorio](#-estructura-del-repositorio)
2. [Objetivos de las Prácticas](#-objetivos-de-las-prácticas)
   - [Práctica 0: Introducción a Jupyter Notebooks](#práctica-0-introducción-a-jupyter-notebooks)
   - [Práctica 1: Conectando el Hardware](#práctica-1-conectando-el-hardware)
   - [Práctica 2: Diferencias de Consumo de Potencia entre Instrucciones](#práctica-2-diferencias-de-consumo-de-potencia-entre-instrucciones)
   - [Práctica 3: Análisis de Consumo para Evadir Contraseña](#práctica-3-análisis-de-consumo-para-evadir-contraseña)
   - [Práctica 4: Introducción a la Inyección de Fallos por Voltaje](#práctica-4-introducción-a-la-inyección-de-fallos-por-voltaje)
3. [Requisitos de Hardware y Software](#-requisitos-de-hardware-y-software)
4. [Flujo de Trabajo y Modo de Uso](#-flujo-de-trabajo-y-modo-de-uso)
5. [Créditos y Referencias](#-créditos-y-referencias)
6. [Aviso Ético y Responsabilidad](#-aviso-ético-y-responsabilidad)

---

## 📂 Estructura del Repositorio

El repositorio se organiza de la siguiente manera:

```text
.
├── 0.Introduccion_a_Jupyter_Notebooks.ipynb
├── 1.Conectando_el_Hardware.ipynb
├── 2.Diferencias_consumo_potencia_(HARDWARE).ipynb
├── 2.Diferencias_consumo_potencia_(PRINCIPAL).ipynb
├── 3.Analisis_de_consumo_para_evadir_password_(HARDWARE).ipynb
├── 3.Analisis_de_consumo_para_evadir_password_(PRINCIPAL).ipynb
├── 4.Introduccion_a_la_inyeccion_de_fallos_por_voltaje.ipynb
├── Firmwares/              # Código fuente en C, headers y Makefiles para el microcontrolador víctima
├── Helper_Scripts/         # Cuadernos auxiliares y scripts complementarios
└── img/                    # Diagramas, esquemas de hardware y capturas ilustrativas
```

> [!NOTE]
> **Convención `(HARDWARE)` y `(PRINCIPAL)`:**  
> En las prácticas 2 y 3, el trabajo se desacopla en dos cuadernos para mantener el código modular y limpio:
> - **`(HARDWARE)`**: Contiene la inicialización del osciloscopio, la selección de la plataforma (`CWNANO`), la compilación y carga del firmware en el target, y la definición de las funciones de captura de trazas.
> - **`(PRINCIPAL)`**: Es el cuaderno de trabajo del estudiante. Ejecuta internamente el cuaderno `(HARDWARE)` al inicio mediante la directiva `%run` y se concentra en la lógica de análisis de trazas, graficado, algoritmos de detección y explotación de la vulnerabilidad.

---

## 🎯 Objetivos de las Prácticas

A continuación se detallan los objetivos pedagógicos y contenidos técnicos de cada una de las prácticas incluidas:

### Práctica 0: Introducción a Jupyter Notebooks
📁 **Archivo:** `0.Introduccion_a_Jupyter_Notebooks.ipynb`

* **Objetivo general:** Adquirir soltura en el manejo del entorno interactivo de Jupyter Notebooks aplicado al análisis de señales y ejecución de código científico en Python.
* **Objetivos específicos:**
  * Comprender la ejecución interactiva y no secuencial de celdas de código.
  * Utilizar celdas mágicas de Bash (`%%bash`) para interactuar con la línea de comandos y el comando mágico `%run` para invocar scripts externos.
  * Representar datos y formas de onda mediante `matplotlib`, tanto en modo estático como dinámico/interactivo (`%matplotlib notebook`, `fig.canvas.draw()`) dentro de bucles de adquisición.
  * Integrar widgets visuales (`ipywidgets`) para monitorizar barras de progreso y estado durante la captura de trazas.
  * Manejar atajos de teclado esenciales para la inspección de funciones (`Shift+Tab`) y documentación interactiva.

---

### Práctica 1: Conectando el Hardware
📁 **Archivo:** `1.Conectando_el_Hardware.ipynb`

* **Objetivo general:** Aprender a configurar y controlar la placa de adquisición **ChipWhisperer-Nano** y establecer comunicación con el microcontrolador objetivo (*target*).
* **Objetivos específicos:**
  * Reconocer la arquitectura de la placa ChipWhisperer-Nano (módulo capturador/glitcher integrado junto a microcontrolador víctima STM32F0).
  * Inicializar el osciloscopio digital y la interfaz de comunicación mediante la API de Python de ChipWhisperer (`cw.scope()`, `cw.target()`).
  * Compilar firmwares en lenguaje C para la plataforma (`PLATFORM=CWNANO`) y programar la memoria flash del objetivo con `cw.program_target`.
  * Comprender el protocolo de comunicación **SimpleSerial** (versiones 1 y 2 con codificación binaria COBS y comprobación CRC) y el uso de comandos serie directos (*raw serial*).
  * Configurar disparadores de captura por hardware (*triggers*) sincronizados con pines GPIO del microcontrolador víctima.
  * Capturar y almacenar trazas analógicas de consumo eléctrico (`scope.arm()`, `scope.capture()`, `scope.get_last_trace()`, `cw.capture_trace()`).
  * Aplicar buenas prácticas para liberar y desconectar los recursos USB del hardware (`scope.dis()`, `target.dis()`).

---

### Práctica 2: Diferencias de Consumo de Potencia entre Instrucciones
📁 **Archivos:** `2.Diferencias_consumo_potencia_(HARDWARE).ipynb` y `2.Diferencias_consumo_potencia_(PRINCIPAL).ipynb`

* **Objetivo general:** Comprender los fundamentos físicos del Análisis Simple de Potencia (SPA - *Simple Power Analysis*), observando cómo las instrucciones ejecutadas por la CPU generan variaciones mensurables en el consumo eléctrico.
* **Objetivos específicos:**
  * Capturar trazas de potencia correspondientes a la ejecución de distintos bloques de código ensamblador/C en el firmware (`simpleserial-base-lab2`).
  * Identificar y comparar las formas de onda producidas por:
    * Traza "vacía" de control (ruido base y estado inactivo).
    * Instrucciones sencillas (`nop`, operaciones lógicas y aritméticas básicas).
    * Bucles de control e instrucciones de salto condicional/incondicional (*loops* y *branches*).
    * Instrucciones de coste computacional elevado (multiplicaciones de 32 bits, divisiones enteras, accesos a memoria).
  * Realizar ingeniería inversa del flujo de ejecución del procesador a partir de la firma visual dejada en la señal de corriente.

---

### Práctica 3: Análisis de Consumo para Evadir Contraseña
📁 **Archivos:** `3.Analisis_de_consumo_para_evadir_password_(HARDWARE).ipynb` y `3.Analisis_de_consumo_para_evadir_password_(PRINCIPAL).ipynb`

* **Objetivo general:** Explotar fugas de información por canales laterales de tiempo y potencia (*Timing & Power Analysis*) para vulnerar un mecanismo de autenticación por contraseña.
* **Objetivos específicos:**
  * Analizar la vulnerabilidad inherente a las rutinas de comparación de cadenas no constantes (comprobación carácter a carácter con salida prematura o *early exit* en `basic-passwdcheck.c`).
  * Comprender cómo la coincidencia de caracteres válidos prolonga el tiempo de ejecución y modifica el perfil temporal de consumo.
  * Recopilar trazas de potencia variando los caracteres introducidos y utilizar una traza de referencia nula para aislar diferencias de señal.
  * Identificar los puntos temporales clave (*Points of Interest* - POIs) donde se procesa cada carácter de la contraseña.
  * Automatizar en Python un ataque que deduce la clave secreta carácter a carácter mediante una estrategia *divide y vencerás*, reduciendo la complejidad computacional de un ataque por fuerza bruta exponencial ($\mathcal{O}(|\Sigma|^L)$) a una complejidad lineal ($\mathcal{O}(|\Sigma| \cdot L)$).
  * Depurar problemas típicos del ataque (desalineación de trazas, gestión de buffers en comunicaciones serie, mantenimiento de prefijos adivinados).

---

### Práctica 4: Introducción a la Inyección de Fallos por Voltaje
📁 **Archivo:** `4.Introduccion_a_la_inyeccion_de_fallos_por_voltaje.ipynb`

* **Objetivo general:** Entender el principio físico y la aplicación práctica de los ataques por inyección de fallos mediante perturbaciones transitorias de alimentación (*Voltage Glitching*).
* **Objetivos específicos:**
  * Comprender cómo una caída ultracorta y controlada en la tensión de alimentación ($V_{CC}$) viola los tiempos de propagación de los biestables digitales, provocando la omisión de instrucciones (*instruction skipping*) o corrupción de registros sin destruir el componente.
  * Configurar los parámetros de inyección de fallos en ChipWhisperer-Nano: retraso respecto al trigger (`ext_offset`) y duración de la perturbación (`repeat`).
  * Implementar mecanismos de recuperación y reset automático por hardware (`reboot_flush`) ante cuelgues o bloqueos del microcontrolador.
  * Utilizar `cw.GlitchController` para automatizar el barrido paramétrico y representar en mapas 2D las zonas de operación normal, reinicio forzado (*reset/crash*) y fallo exitoso (*glitch*).
  * Ejecutar dos ataques prácticos:
    1. **Corrupción de bucles de cálculo:** Provocar que un bucle de $50 \times 50$ iteraciones en `simpleserial-glitch.c` calcule un valor erróneo distinto de 2500 sin provocar el reinicio del sistema.
    2. **Evasión de verificación de contraseña:** Inyectar un fallo en el instante exacto en que se evalúa la bifurcación condicional de autenticación, consiguiendo acceso autorizado sin suministrar la contraseña válida.

---

## 🛠 Requisitos de Hardware y Software

### Hardware
* **Placa ChipWhisperer-Nano** (NewAE Technology Inc.) con cable micro-USB.
  * Microcontrolador objetivo (*target*): **STM32F0** (ARM Cortex-M0).
  * Osciloscopio integrado (hasta 10 MS/s) y circuito MOSFET de corte de $V_{CC}$ para glitching.

### Software
* **Sistema Operativo:** Linux, macOS o Windows (con soporte para controladores USB de ChipWhisperer).
* **Python:** 3.8 o superior.
* **Paquetes de Python:**
  * `chipwhisperer` (versión 5.x o superior)
  * `jupyter` / `notebook`
  * `matplotlib`
  * `numpy`
  * `ipywidgets`
* **Cadena de herramientas (Toolchain) de compilación cruzada:**
  * `arm-none-eabi-gcc`, `arm-none-eabi-binutils`, `make` (necesarios para compilar el firmware de los targets en `Firmwares/`).

---

## 🚀 Flujo de Trabajo y Modo de Uso

1. **Instalación del entorno:**  
   Asegúrate de contar con Python y el paquete `chipwhisperer` instalado (consulta la [documentación oficial de instalación](https://chipwhisperer.readthedocs.io/en/latest/installation.html)).

2. **Conexión del hardware:**  
   Conecta la placa ChipWhisperer-Nano al puerto USB de tu ordenador.

3. **Lanzar Jupyter:**  
   Abre una terminal en la raíz de este repositorio y ejecuta:
   ```bash
   jupyter notebook
   ```

4. **Secuencia de realización recomendada:**
   * Comienza por `0.Introduccion_a_Jupyter_Notebooks.ipynb` para familiarizarte con las herramientas interactivas.
   * Continúa con `1.Conectando_el_Hardware.ipynb` para verificar la detección de la placa, la compilación de firmwares y la comunicación serie.
   * Abre `2.Diferencias_consumo_potencia_(PRINCIPAL).ipynb` para estudiar el Análisis Simple de Potencia.
   * Abre `3.Analisis_de_consumo_para_evadir_password_(PRINCIPAL).ipynb` para llevar a cabo el ataque diferencial contra la comprobación de contraseña.
   * Abre `4.Introduccion_a_la_inyeccion_de_fallos_por_voltaje.ipynb` para realizar las prácticas de *Voltage Glitching*.

> [!IMPORTANT]
> Recuerda ejecutar siempre las celdas finales de desconexión (`scope.dis()` y `target.dis()`) al finalizar cada práctica para liberar la interfaz USB y evitar bloqueos en el sistema operativo.

---

## 📚 Créditos y Referencias

Los cuadernos y firmwares de este repositorio están basados en el material formativo oficial desarrollado por **[NewAE Technology Inc.](https://www.newae.com/)** y creado originalmente por **Colin O'Flynn** y el equipo del proyecto de código abierto **ChipWhisperer**.

Este material ha sido traducido, revisado didácticamente y adaptado específicamente al hardware **ChipWhisperer-Nano** en el marco de la docencia universitaria de la asignatura de **Vulnerabilidades Hardware**.

A continuación se detalla la correspondencia con los materiales originales del repositorio [newaetech/chipwhisperer-jupyter](https://github.com/newaetech/chipwhisperer-jupyter):

| Cuaderno en este repositorio | Material Original en ChipWhisperer | Enlace de Referencia |
| :--- | :--- | :--- |
| **`0.Introduccion_a_Jupyter_Notebooks.ipynb`** | `0 - Introduction to Jupyter Notebooks.ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/0%20-%20Introduction%20to%20Jupyter%20Notebooks.ipynb) |
| **`1.Conectando_el_Hardware.ipynb`** | `1 - Connecting to Hardware.ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/1%20-%20Connecting%20to%20Hardware.ipynb) |
| **`2.Diferencias_consumo_potencia_(HARDWARE).ipynb`** | `courses/sca101/Lab 2_1A - Instruction Power Differences (HARDWARE).ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/sca101/Lab%202_1A%20-%20Instruction%20Power%20Differences%20(HARDWARE).ipynb) |
| **`2.Diferencias_consumo_potencia_(PRINCIPAL).ipynb`** | `courses/sca101/Lab 2_1A - Instruction Power Differences (MAIN).ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/sca101/Lab%202_1A%20-%20Instruction%20Power%20Differences%20(MAIN).ipynb) |
| **`3.Analisis_de_consumo_para_evadir_password_(HARDWARE).ipynb`** | `courses/sca101/Lab 2_1B - Power Analysis for Password Bypass (HARDWARE).ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/sca101/Lab%202_1B%20-%20Power%20Analysis%20for%20Password%20Bypass%20(HARDWARE).ipynb) |
| **`3.Analisis_de_consumo_para_evadir_password_(PRINCIPAL).ipynb`** | `courses/sca101/Lab 2_1B - Power Analysis for Password Bypass (MAIN).ipynb` | [Ver original](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/sca101/Lab%202_1B%20-%20Power%20Analysis%20for%20Password%20Bypass%20(MAIN).ipynb) |
| **`4.Introduccion_a_la_inyeccion_de_fallos_por_voltaje.ipynb`** | `courses/fault101/Fault 2_1 - Introduction to Voltage Glitching.ipynb` <br>+ `courses/fault101/Fault 2_2 - Voltage Glitching to Bypass Password.ipynb` | [Fault 2_1](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/fault101/Fault%202_1%20-%20Introduction%20to%20Voltage%20Glitching.ipynb) <br> [Fault 2_2](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/courses/fault101/Fault%202_2%20-%20Voltage%20Glitching%20to%20Bypass%20Password.ipynb) |

### Recursos y Enlaces Adicionales
* [Documentación oficial de ChipWhisperer](https://chipwhisperer.readthedocs.io/)
* [Repositorio oficial de ChipWhisperer en GitHub](https://github.com/newaetech/chipwhisperer)
* [Wiki de NewAE](https://wiki.newae.com/)

---

## ⚖️ Aviso Ético y Responsabilidad

El material y las técnicas presentadas en este repositorio tienen fines estrictamente **educativos y de investigación en seguridad hardware**. El objetivo es dotar a estudiantes y profesionales de los conocimientos necesarios para comprender cómo se diseñan sistemas embebidos seguros y cómo implementar contramedidas efectivas frente a ataques por canales laterales y manipulación física de dispositivos.
