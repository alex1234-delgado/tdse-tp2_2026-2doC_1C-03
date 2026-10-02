# Explicación del código fuente MAIN segun Gemini

**Evolución de SystemCoreClock, SysTick y Flujo de Ejecución**

* **Inicio en ensamblador:** El ciclo de vida del programa comienza en la rutina `Reset_Handler` dentro del archivo de arranque. En esta primera etapa, el procesador llama a la función externa `SystemInit`. Durante esta ejecución inicial, la variable `SystemCoreClock` se inicializa con la frecuencia de reloj por defecto del microcontrolador (típicamente el oscilador interno, HSI).


* **Preparación de memoria:** Antes de entrar al código principal, el programa copia los inicializadores de la sección de datos (`.data`) desde la Flash hacia la SRAM. Posteriormente, llena con ceros el segmento `.bss`.


* **Transición a C:** Tras ejecutar `__libc_init_array`, el código ensamblador hace un salto hacia el punto de entrada de la aplicación, llamando a la función `main`.


* **Activación inicial del SysTick:** La primera instrucción dentro de `main()` es la ejecución de `HAL_Init()`. Esta función reinicia todos los periféricos e inicializa la interfaz Flash y el temporizador SysTick. En este instante, el hardware del SysTick comienza a funcionar basándose en la velocidad inicial de `SystemCoreClock`.


* **Reconfiguración del Reloj del Sistema:** A continuación, se llama a `SystemClock_Config()`. Esta función configura el oscilador HSI encendiéndolo y aplicándole un multiplicador PLL de 16, tomando como fuente la frecuencia del HSI dividida por dos (`RCC_PLLSOURCE_HSI_DIV2` y `RCC_PLL_MUL16`).


* **Actualización de SystemCoreClock:** La nueva configuración de frecuencias se aplica mediante la instrucción `HAL_RCC_ClockConfig()`. Como parte del funcionamiento interno de esta función, la variable global `SystemCoreClock` es actualizada para reflejar el nuevo reloj del sistema. Inmediatamente, la misma función reajusta los registros del hardware del SysTick para garantizar que sus interrupciones sigan ocurriendo en los intervalos correctos a la nueva velocidad.


* **Manejo de Interrupciones en segundo plano:** El hardware SysTick dispara interrupciones continuas que son capturadas en el archivo de interrupciones mediante la función `SysTick_Handler()`. Este manejador invoca a `HAL_IncTick()`, lo que actualiza la variable interna de la HAL encargada de contar los milisegundos transcurridos.


* **Ingreso al Loop Principal:** Tras configurar los puertos GPIO y USART2 (`MX_GPIO_Init` y `MX_USART2_UART_Init`), y luego de invocar `app_init()`, el código ingresa al bloque `while(1)`. Al alcanzar este punto de ejecución, `SystemCoreClock` posee su valor final estático configurado por el PLL, y el hardware de SysTick sigue operando y generando interrupciones en segundo plano de forma ininterrumpida mientras la aplicación llama repetidamente a `app_update()`.
