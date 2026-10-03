### Depuración del proyecto STM32

Mediante la depuración, analizamos los valores de **task_dta_list[index]**, segun la tabla:

| Expression       | Type         | Value | Unit |
|------------------|--------------|-------|------|
|`task_dta_list[0]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `16654` | dimensionless |
| --`LET`      | `uint32_t`   | `10` | uS |
| --`BCET`     | `uint32_t`   | `10` | uS |
| --`WCET`     | `uint32_t`   | `11` | uS |
|`task_dta_list[1]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `16654` | dimensionless |
| --`LET`      | `uint32_t`   | `3` | uS |
| --`BCET`     | `uint32_t`   | `3` | uS |
| --`WCET`     | `uint32_t`   | `6` | uS |
|`task_dta_list[2]` | `task_dta_t` | `{...}` | - |
| --`NOE`      | `uint32_t`   | `16654` | dimensionless |
| --`LET`      | `uint32_t`   | `4` | uS |
| --`BCET`     | `uint32_t`   | `4` | uS |
| --`WCET`     | `uint32_t`   | `7` | uS |

