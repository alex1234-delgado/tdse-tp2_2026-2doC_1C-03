# Explicación del código fuente ACTUATOR segun Gemini

El código implementa la gestión no bloqueante de un actuador (salida digital GPIO para un LED) impulsada por eventos mediante una máquina de estados finitos (FSM). La interfaz expone la función `put_event_task_actuator()`, la cual permite a otras tareas enviar comandos actualizando el evento correspondiente y marcando una bandera (`flag = true`); posteriormente, la función periódica `task_actuator_update()` procesa dichos eventos y modifica el estado lógico del pin GPIO del LED.

**Comportamiento de `task_actuator_statechart(uint32_t index)`**

1. **Obtención de punteros**: Asigna los punteros a las estructuras de configuración (`p_task_actuator_cfg`) y de datos (`p_task_actuator_dta`) según el parámetro `index`.


2. **Evaluación FSM**:
* **`ST_LED_IDLE`**: Si la bandera es verdadera (`flag == true`) y el evento recibido es `EV_LED_ACTIVE`, limpia la bandera (`flag = false`), escribe en el puerto GPIO la señal `led_on` y conmuta el estado a `ST_LED_ACTIVE`.


* **`ST_LED_ACTIVE`**: Si la bandera es verdadera (`flag == true`) y el evento recibido es `EV_LED_IDLE`, limpia la bandera (`flag = false`), escribe en el puerto GPIO la señal `led_off` y conmuta el estado a `ST_LED_IDLE`.


* **`default`**: Ante cualquier estado no válido, reinicia los parámetros a `tick = DEL_LED_MIN` (0), `state = ST_LED_IDLE`, `event = EV_LED_IDLE` y `flag = false`.





**Evolución de Variables de la Tarea Actuador (`task_actuator_dta_list`)**

* **`index`**: Vale `0`, pues la lista solo posee una estructura configurada (`ACTUATOR_CFG_QTY = 1`, referente a `ID_LED_A`).


* **`task_actuator_dta_list[index].tick`**:
* **Unidad de medida**: Milisegundos (**mS**), según la frecuencia de refresco configurada en el sistema.


* **Evolución**: Inicia en `0` y se mantiene siempre en `0`, debido a que esta FSM en particular no incrementa ni gestiona temporizadores de permanencia en estado.





| Etapa de Ejecución | `index` | `tick` [mS] | `state` | `event` | `flag` | Estado del LED / Acción |
| --- | --- | --- | --- | --- | --- | --- |
| **Inicialización (`task_actuator_init`)**<br> | 0 | 0 | `ST_LED_IDLE` (0) | `EV_LED_IDLE` (0) | `false`<br> | Apagado (`led_off`) |
| **Loop sin nuevos eventos (`task_actuator_update`)**<br> | 0 | 0 | `ST_LED_IDLE` (0) | `EV_LED_IDLE` (0) | `false`<br> | Permanece apagado |
| **Recepción de evento activo**<br> | 0 | 0 | Transiciona a `ST_LED_ACTIVE` (1) | `EV_LED_ACTIVE` (1) | Pasa a `false` tras procesar | Encendido (`led_on`) |
| **Loop manteniéndose activo (`task_actuator_update`)**<br> | 0 | 0 | `ST_LED_ACTIVE` (1) | `EV_LED_ACTIVE` (1) | `false`<br> | Permanece encendido |
| **Recepción de evento de reposo**<br> | 0 | 0 | Transiciona a `ST_LED_IDLE` (0) | `EV_LED_IDLE` (0) | Pasa a `false` tras procesar | Apagado (`led_off`) |

**Evolución de Variables en la Interfaz (`put_event_task_actuator`)**

* **`identifier`**: Representa el actuador a modificar y toma la constante `ID_LED_A` (valor numérico `0`).

| Función / Operación | `identifier` | `task_actuator_dta_list[identifier].event` | `task_actuator_dta_list[identifier].flag` | Efecto en la FSM |
| --- | --- | --- | --- | --- |
| **Arranque del sistema (`task_actuator_init`)**<br> | 0 (`ID_LED_A`) | `EV_LED_IDLE` (0) | `false`<br> | Configuración inicial en reposo. |
| **Llamada `put_event_task_actuator(EV_LED_ACTIVE, ID_LED_A)**`<br> | 0 (`ID_LED_A`) | `EV_LED_ACTIVE` (1) | `true`<br> | Habilita a la FSM para encender el LED. |
| **Ejecución de `task_actuator_statechart(0)**`<br> | 0 (`ID_LED_A`) | `EV_LED_ACTIVE` (1) | `false`<br> | La FSM enciende el LED y consume el evento borrando la bandera. |
| **Llamada `put_event_task_actuator(EV_LED_IDLE, ID_LED_A)**`<br> | 0 (`ID_LED_A`) | `EV_LED_IDLE` (0) | `true`<br> | Habilita a la FSM para apagar el LED. |
| **Ejecución de `task_actuator_statechart(0)**`<br> | 0 (`ID_LED_A`) | `EV_LED_IDLE` (0) | `false`<br> | La FSM apaga el LED y consume el evento borrando la bandera. |
