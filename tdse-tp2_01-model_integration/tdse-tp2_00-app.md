# Explicación del código fuente APP segun Gemini

El código adjunto implementa un planificador de tareas en un entorno bare-metal disparado por eventos temporales, apoyándose en interrupciones del SysTick y en el contador de ciclos por hardware (DWT) para medir con precisión el rendimiento de cada tarea.

**Evolución de Variables desde el Inicio hasta el Bucle Principal**

| Variable | Inicialización (`app_init` / `app_it_init`) | Comportamiento en Bucle Principal (`app_update` y callbacks) | Unidad de Medida |
| --- | --- | --- | --- |
| `g_app_tick_cnt` | Se inicializa en `0` deshabilitando interrupciones. | Incrementa en `1` automáticamente por hardware cada vez que se dispara `HAL_SYSTICK_Callback`. Decrementa en `1` dentro de `app_update()` al procesarse un ciclo de tiempo. | Adimensional (Contador de Ticks) |
| `index` | Itera de `0` a `TASK_QTY - 1` ejecutando las funciones de inicio para las 3 tareas del sistema. | Recorre iterativamente las tareas de `0` a `TASK_QTY - 1` secuenciando su ejecución e instrumentación.| Adimensional (Índice de array) |
| `g_app_runtime_us` | No se afecta durante la fase de inicialización inicial. | Se reinicia a `0` al inicio de cada ciclo de ejecución y acumula el tiempo individual (`LET`) de todas las tareas ejecutadas en esa iteración. | Microsegundos (µs) |
| `task_dta_list[index].NOE` | Inicializada al valor de la macro `TASK_X_NOE_INI` (`0`). | Suma `1` tras la actualización de cada tarea, indicando la cantidad total de veces que se ha ejecutado. | Adimensional (Contador numérico) |
| `task_dta_list[index].LET` | Inicializada al valor de la macro `TASK_X_LET_INI` (`0`). | Se sobrescribe obteniendo el tiempo transcurrido desde el hardware DWT (`cycle_counter_get_time_us()`), representando la demora de la última pasada de la tarea. | Microsegundos (µs) |
| `task_dta_list[index].BCET` | Inicializada en `TASK_X_BCET_INI` (`1000`). | Registra el "Mejor Caso": si el tiempo actual (`LET`) es estrictamente menor al récord guardado en `BCET`, este último se actualiza con el nuevo valor mínimo. | Microsegundos (µs) |
| `task_dta_list[index].WCET` | Inicializada en `TASK_X_WCET_INI` (`0`). | Registra el "Peor Caso": si el tiempo de ejecución actual (`LET`) supera el máximo guardado en `WCET`, este asume el nuevo récord. | Microsegundos (µs) |

**Impacto de usar LOGGER_INFO() en la medición de rendimiento**

La macro `LOGGER_INFO()` funciona deshabilitando las interrupciones del microcontrolador (`__asm("CPSID i")`), ejecutando el formateo de strings en memoria mediante `snprintf`, transmitiendo el mensaje con `logger_log_print_`, y finalmente rehabilitando las interrupciones (`__asm("CPSIE i")`).

Si se invoca esta macro dentro de una rutina `task_x_update`:

* **Impacto sobre `task_dta_list[index].WCET`:** Mientras el registro bloquea la CPU y realiza operaciones lentas, el contador interno de hardware (DWT) seguirá acumulando ciclos de reloj en segundo plano. Como resultado, el tiempo reportado para esa pasada (`LET`) sufrirá un pico enorme. Dado que el sistema siempre captura y retiene el peor escenario histórico (`WCET < LET`), el perfil del "Peor Caso" (`WCET`) quedará arruinado y artificialmente elevado de forma permanente.


* **Impacto sobre `g_app_runtime_us`:** Como esta variable totaliza linealmente todo el tiempo insumido por las tareas en un ciclo (`g_app_runtime_us += task_dta_list[index].LET`), heredará directamente esa inflación momentánea de ciclos provocada por el log. Durante el ciclo exacto donde se ejecutó la impresión, reportará un tiempo total de ejecución que no representará el flujo de control puro, sino la lentitud de la propia herramienta de depuración.


### Depuración del proyecto STM32

Mediante la depuración, analizamos los valores de **task_dta_list[index]**, segun la tabla:

| Expression       | Type         | Value | Unit |
|------------------|--------------|-------|------|
|`task_dta_list[0]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `31688` | dimensionless |
| --`LET`      | `uint32_t`   | `4` | uS |
| --`BCET`     | `uint32_t`   | `4` | uS |
| --`WCET`     | `uint32_t`   | `5` | uS |
|`task_dta_list[1]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `31688` | dimensionless |
| --`LET`      | `uint32_t`   | `3` | uS |
| --`BCET`     | `uint32_t`   | `3` | uS |
| --`WCET`     | `uint32_t`   | `5` | uS |
|`task_dta_list[2]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `31688` | dimensionless |
| --`LET`      | `uint32_t`   | `2` | uS |
| --`BCET`     | `uint32_t`   | `2` | uS |
| --`WCET`     | `uint32_t`   | `4` | uS |

