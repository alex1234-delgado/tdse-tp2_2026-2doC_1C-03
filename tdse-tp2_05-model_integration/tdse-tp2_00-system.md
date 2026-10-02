# Explicación del código fuente SYSTEM, ACTUADOR segun Gemini

El código implementa un gestor de control intermediario orientado a eventos, no bloqueante y de ejecución periódica (cada 1 ms). La tarea principal del sistema (`task_system.c`) se encarga de desencolar eventos emitidos por sensores u otras fuentes a través de una cola circular FIFO (`event_task_system_queue`), procesarlos mediante una máquina de estados finitos (FSM), y transmitir los comandos resultantes hacia la tarea de actuadores mediante la función de interfaz `put_event_task_actuator`.

**Comportamiento de `task_system_normal_statechart(void)`**

1. **Lectura de eventos**: Consulta la cola mediante `any_event_task_system()`. Si existen eventos pendientes, extrae el siguiente evento mediante `get_event_task_system()`, lo guarda en `p_task_system_dta->event` y establece `p_task_system_dta->flag = true`.


2. **Evaluación de la máquina de estados**:
* **`ST_SYS_IDLE`**: Si `flag == true` y `event == EV_SYS_ACTIVE`, limpia la bandera (`flag = false`), envía la señal de activación al actuador invocando `put_event_task_actuator(EV_LED_ACTIVE, ID_LED_A)` y conmuta el estado a `ST_SYS_ACTIVE`.


* **`ST_SYS_ACTIVE`**: Si `flag == true` y `event == EV_SYS_IDLE`, limpia la bandera (`flag = false`), envía la señal de inactividad al actuador invocando `put_event_task_actuator(EV_LED_IDLE, ID_LED_A)` y conmuta el estado a `ST_SYS_IDLE`.


* **`default`**: Restablece los atributos del sistema a valores por defecto (`tick = 0`, `state = ST_SYS_IDLE`, `event = EV_SYS_IDLE`, `flag = false`).





---

**Evolución de Variables de la Tarea Sistema (`task_system_dta_list`)**

* **`index`**: Solo toma el valor `0`, ya que únicamente está configurado el modo `NORMAL` (`SYSTEM_DTA_QTY = 1`).


* **`task_system_dta_list[index].tick`**:
* **Unidad de medida**: Milisegundos (**mS**), derivado del período de ejecución especificado en los comentarios del código.


* **Evolución**: Inicia en `0` y se mantiene en `0` durante toda la ejecución, ya que el estado del sistema no incluye contadores de tiempo dentro de esta FSM (solo se fuerza a `DEL_SYS_MIN` (0) en el bloque `default`).





| Etapa de Ejecución | `index` | `tick` [mS] | `state` | `event` | `flag` | Transición / Acción |
| --- | --- | --- | --- | --- | --- | --- |
| **Inicio (`task_system_init`)**<br> | 0 | 0 | `ST_SYS_IDLE` (0) | `EV_SYS_IDLE` (0) | `false`<br> | Estado inicial configurado. |
| **Loop sin eventos en cola**<br> | 0 | 0 | `ST_SYS_IDLE` (0) | `EV_SYS_IDLE` (0) | `false`<br> | Permanece en reposo sin cambios. |
| **Recepción de `EV_SYS_ACTIVE**`<br> | 0 | 0 | `ST_SYS_ACTIVE` (1) | `EV_SYS_ACTIVE` (1) | `false`<br> | Cambia a `ST_SYS_ACTIVE` y encola `EV_LED_ACTIVE`. |
| **Recepción de `EV_SYS_IDLE**`<br> | 0 | 0 | `ST_SYS_IDLE` (0) | `EV_SYS_IDLE` (0) | `false`<br> | Cambia a `ST_SYS_IDLE` y encola `EV_LED_IDLE`. |

---

**Evolución de Variables de la Cola (`event_task_system_queue`)**

* **`i`**: Variable de iteración local empleada durante la función `init_event_task_system()` para recorrer la cola desde el índice `0` hasta el `15`.



| Evento / Función | `head` | `tail` | `count` | `queue[0]` | `queue[1]` | `queue[2..15]` |
| --- | --- | --- | --- | --- | --- | --- |
| **Inicio (`init_event_task_system`)**<br> | 0 | 0 | 0 | 255 (`EMPTY`) | 255 (`EMPTY`) | 255 (`EMPTY`)|
| **`put_event_task_system(EV_SYS_ACTIVE)`**<br> | 1 | 0 | 1 | **1** (`EV_SYS_ACTIVE`) | 255 | 255 |
| **`get_event_task_system()`**<br> | 1 | 1 | 0 | **255** (`EMPTY`) | 255 | 255 |
| **`put_event_task_system(EV_SYS_IDLE)`**<br> | 2 | 1 | 1 | 255 | **0** (`EV_SYS_IDLE`) | 255 |

---

**Evolución de Variables de Interfaz del Actuador (`task_actuator_dta_list`)**

* **`identifier`**: Representa el ID del actuador controlado, tomando la constante `ID_LED_A` (valor `0`).



| Acción originada en `task_system.c` | `identifier` | `task_actuator_dta_list[identifier].event` | `task_actuator_dta_list[identifier].flag` | Impacto en la Tarea Actuador |
| --- | --- | --- | --- | --- |
| **Inicio (`task_actuator_init`)**<br> | 0 (`ID_LED_A`) | `EV_LED_IDLE` (0) | `false`<br> | Estado inicial del LED apagado. |
| **Llamada `put_event_task_actuator(EV_LED_ACTIVE, ID_LED_A)**`<br> | 0 (`ID_LED_A`) | `EV_LED_ACTIVE` (1) | `true` | Habilita al actuador para encender el LED. |
| **Consumo del evento por el Actuador**<br> | 0 (`ID_LED_A`) | `EV_LED_ACTIVE` (1) | `false`<br> | La FSM del actuador procesa la activación y limpia el flag. |
| **Llamada `put_event_task_actuator(EV_LED_IDLE, ID_LED_A)**`<br> | 0 (`ID_LED_A`) | `EV_LED_IDLE` (0) | `true` | Habilita al actuador para apagar el LED. |
