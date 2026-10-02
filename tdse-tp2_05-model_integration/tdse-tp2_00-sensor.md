# Explicación del código fuente SYSTEM, SENSORS segun Gemini

El código implementa un sistema no bloqueante impulsado por eventos para la lectura de sensores (botones) y la comunicación mediante una cola FIFO circular hacia una tarea de sistema.

## Funcionamiento General del Código

* **`task_sensor_attribute.h`**: Define las estructuras de datos del sensor (`task_sensor_cfg_t` y `task_sensor_dta_t`), sus estados (`ST_BTN_IDLE`, `ST_BTN_ACTIVE`) y sus eventos (`EV_BTN_UP`, `EV_BTN_DOWN`).
* **`task_system_attribute.h`**: Define los eventos del sistema (`EV_SYS_IDLE`, `EV_SYS_ACTIVE`) que son enviados por los sensores cuando detectan un cambio de estado.
* **`task_sensor.c`**: Contiene la máquina de estados finitos (FSM) que gestiona el sensor. Inicializa los datos con `task_sensor_init()` y los actualiza periódicamente mediante `task_sensor_update()`.
* **`task_system_interface.c`**: Implementa un buffer circular FIFO de 16 elementos (`event_task_system_queue_t`) para encolar (`put_event_task_system`) y desencolar (`get_event_task_system`) los eventos dirigidos al sistema.

---

## Comportamiento de `task_sensor_statechart(uint32_t index)`

1. **Lectura de Entrada Digital**: Lee el pin GPIO configurado (`HAL_GPIO_ReadPin`). Si el nivel lógico coincide con el valor `pressed`, asigna `event = EV_BTN_DOWN` (1); de lo contrario, asigna `event = EV_BTN_UP` (0).
2. **Evaluación de Estado**:
* **`ST_BTN_IDLE`**: Si `event == EV_BTN_DOWN`, llama a `put_event_task_system(signal_down)` (que envía `EV_SYS_ACTIVE`) y conmuta el estado a `ST_BTN_ACTIVE`.
* **`ST_BTN_ACTIVE`**: Si `event == EV_BTN_UP`, llama a `put_event_task_system(signal_up)` (que envía `EV_SYS_IDLE`) y conmuta el estado a `ST_BTN_IDLE`.
* **`default`**: Restablece los parámetros asignando `tick = DEL_BTN_MIN` (0), `state = ST_BTN_IDLE` y `event = EV_BTN_UP`.



---

## Evolución de Variables de la Tarea Sensor

* **`index`**: Toma únicamente el valor `0` ya que solo existe un sensor configurado en la lista (`SENSOR_CFG_QTY = 1`).
* **`task_sensor_dta_list[index].tick`**:
* **Unidad de medida**: Milisegundos (**mS**), definido en los comentarios de periodicidad del archivo.
* **Evolución**: Se inicializa implícitamente en `0` (sección BSS) y permanece en `0` durante la ejecución, ya que en el flujo principal de `task_sensor_statechart` no se incrementa (solo se reasigna a `0` en el bloque `default`).



| Etapa de Ejecución | `index` | `tick` [mS] | `state` | `event` | Acción / Transición |
| --- | --- | --- | --- | --- | --- |
| **Inicio (`task_sensor_init`)** | 0 | 0 | `ST_BTN_IDLE` (0) | `EV_BTN_UP` (0) | Configuración inicial de la FSM. |
| **Update (Botón Suelto)** | 0 | 0 | `ST_BTN_IDLE` (0) | `EV_BTN_UP` (0) | Sin cambio de estado ni evento generado. |
| **Update (Presión de Botón)** | 0 | 0 | Transiciona a `ST_BTN_ACTIVE` (1) | `EV_BTN_DOWN` (1) | Transición de estado y encola `EV_SYS_ACTIVE` (1). |
| **Update (Manteniendo Presionado)** | 0 | 0 | `ST_BTN_ACTIVE` (1) | `EV_BTN_DOWN` (1) | Se mantiene en estado activo sin encolar eventos. |
| **Update (Liberación de Botón)** | 0 | 0 | Transiciona a `ST_BTN_IDLE` (0) | `EV_BTN_UP` (0) | Transición de estado y encola `EV_SYS_IDLE` (0). |

---

## Evolución de Variables de la Cola (`event_task_system_queue`)

La cola FIFO almacena los eventos `task_system_ev_t`. Su tamaño total es de 16 elementos y el valor que representa una posición vacía es `EMPTY` (255).

| Evento de Sistema / Función | `head` | `tail` | `count` | `queue[0]` | `queue[1]` | `queue[2..15]` |
| --- | --- | --- | --- | --- | --- | --- |
| **Inicio (`init_event_task_system`)** | 0 | 0 | 0 | 255 | 255 | 255 |
| **Pulsación de Botón (`put_event_task_system(EV_SYS_ACTIVE)`)** | 1 | 0 | 1 | **1** (`EV_SYS_ACTIVE`) | 255 | 255 |
| **Liberación de Botón (`put_event_task_system(EV_SYS_IDLE)`)** | 2 | 0 | 2 | 1 | **0** (`EV_SYS_IDLE`) | 255 |
| **Lectura de 1 Evento (`get_event_task_system()`)** | 2 | 1 | 1 | **255** (`EMPTY`) | 0 | 255 |

Los índices `head` y `tail` avanzan de 0 a 15 de manera circular (`head = (head + 1) % 16`).
