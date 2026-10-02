Example: tdse-tp2_01-model_integration

 Description:
 Bare Metal - Event-Triggered Systems (ETS)
 App - retarget_printf_to_Console
 Project for STM32 Project (STM32CubeIDE Version: 1.19.0 & STM32Cube FW_F1 V1.8.6)

  SystemCoreClock     => 64MHz (15.625nS)
  SysTick Rate Hertz  => 1000 ticks per second (1mS)

  app.c (app.h)
   Endless loops, which execute tasks with fixed computing time. This 
   sequential execution is only deviated from when an interrupt event occurs.
   Cyclic Executive (Update by Time Code, period = 1mS)
    
    app_it.c (app_it.h)
	 Utilities for "Callbacks"

    task_sensor.c (task_sensor.h, task_sensor_attribute.h) 
     Non-Blocking Code (Update by Time Code, period = 1mS) -> Sensor Modeling

    task_system.c (task_system.h, task_system_attribute.h) 
     Non-Blocking Code (Update by Time Code, period = 1mS) -> System Modeling

      task_system_interface.c (task_system_interface.h)
       Non-Blocking Code

    task_actuator.c (task_actuator.h, task_actuator_attribute.h) 
     Non-Blocking Code (Update by Time Code, period = 1mS) -> Actuator Modeling

      task_actuator_interface.c (task_actuator_interface.h)
       Non-Blocking Code

  logger.h (logger.c)
   Utilities for Retarget "printf" to Console

  dwt.h
   Utilities for Mesure "clock cycle" and "execution time" of code
  
  systick.c (systick.h) 
   Utilities for delay "microseconds"

  Special connection requirements:
   There are no special connection requirements for this example.

Build procedures:
Visit the Getting started with STM32: STM32 step-by-step at 
"https://wiki.st.com/stm32mcu/wiki/STM32StepByStep:Getting_started_with_STM32_:_STM32_step_by_step"
to get started building STM32 Projects.