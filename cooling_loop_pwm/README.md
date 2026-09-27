# STM32 FreeRTOS & Hardware PWM Guide

Documentation for setting up STM32 IDE, installing FreeRTOS, and configuring hardware PWM. Includes sample programs and basics of FreeRTOS. 

---

## 1. Installing FreeRTOS on STM32

To smoothly upload the software to the STM3, we use the official **STM32CubeIDE**. Setting up cross-compiler with VScode is a bit tricky but it's doable. 

### Download and Install STMCubeIDE
1. Navigate to official download page [ST STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html).
2. Scroll down, select your OS and install
3. Just follow the installation steps and launch it

---
## 2. Project Initialization in STM32CubeMX

STMCubeMX handles the hardware peripheral initialization code generation. This guide uses the **NUCLEO-F446RE** for reference, but it's the same concept for any other STM board. Any code, setting or hardware pin specific to the F446RE will be mentioned and explained how to navigate it to your specific board. 

### 2.1 Selecting your board
1. Open STM32CubeMX (Not the IDE)
2. Select the *Board Selector Tab* 
3. In the *Commercial Part Number* Box Search for and select **NUCLEO-F446RE** or your specific board
4. Select start project. When prompted with the board project options menu, select the blue *unselect all* box, and click ok. It should load a pinout view with your selected board, and now we can configure the timer peripherals and installing RTOS

### 2.2  Configuring Peripherals and installing FreeRTOS

1. Click on the categories panel if it's not selected already and click on timers to expand the list. 
2. Click on TIM3, a new *TIM3 Mode and Configuration* menu will play. Set clock source to Internal clock. This makes it so it counts ticks from the microcontroller itself, which we will use in PWM.
3. Set Channel 1 to PWM Generation CH1. You can see PA6 light up green in the pinout view on the right hand side. The Timer we just set earlier is now wired to PA6, and allows for rapid changes between HIGH and LOW voltage. 
4. Below configuration settings, select the parameter settings tab. Click on Prescaler (PSC - 16 bits value) and change 0 to 83. Leave counter mode as up. Also change Counter Period (AutoReload Register - ARR) from 65535 to 999
5. On the far left panel under Categories, scroll down and click Middleware and Software Packs to expand it.
6. Scroll down until you find FreeRTOS and click on it. In the mode dropdown that appears at the top, select CMSIS_V1. FreeRTOS is now enabled.
7. Below configuration, select the tasks and Queues tab.
8. Double click the existing default task or click Add. (You may need to lengthen the configuration window to be able to see the default task)
9. Task Name: coolingTask (or any time you choose)
Priority: osPriorityLow:
Stack size: 128 (words)
Entry Function: StartDefaultTask
10. Click Ok. You've successfully added FreeRTOS task and are ready to export your project. 

### 2.3 Exporting Project 
1. Click the *Project Manager* tab along the top. 
2. Select your project name: (eg. cooling_loop_pwm)
3. Project Location: Choose your directory 
4. Toolchain/ IDE: Set the dropdown to STM32CubeIDE
5. On the left side of the project manager screen, click code generator
6. Check the box for "Generate peripheral initialization as a pair of '.c/.h' files per peripheral"
7. Click the blue *GENERATE CODE button in the top right corner. This might take a minute
8. When you recieve the success prompt, choose open project to launch it directly in STM32CubeIDE. 
9. Once you're in the IDE, you're now ready to start programming your STM32. 

---

## 3. Hardware Configuration (Nucleo-F446RE)

| Parameter | Configuration | Details |
| :--- | :--- | :--- |
| **Timer** | `TIM3` | General-purpose 16-bit timer |
| **Channel** | `Channel 1` | Configured as `PWM Generation CH1` |
| **GPIO Pin** | `PA6` | Morpho / Arduino header pin `D12` (Alternate Function AF2) |
| **Prescaler ($PSC$)** | `83` | Scales timer clock down (e.g., $84\text{ MHz} \rightarrow 1\text{ MHz}$ tick rate) |
| **Counter Period ($ARR$)** | `999` | $1000$ counts per cycle $\rightarrow 1\text{ kHz}$ base frequency |
| **Counter Mode** | `Up` | Counts $0 \rightarrow ARR$ |

## 4. Basics of FreeRTOS & Implementation

This section covers how FreeRTOS manages execution on the Nucleo-F446RE, why non blocking delays are required, and the task loop used to control hardware PWM.

Once the project opens in STM32CubeIDE, peripheral initialization calls and task loop logic must be placed inside designated `USER CODE BEGIN` and `USER CODE END` comment blocks. Any code outside these tags will be permanently overwritten if you regenerate code in CubeMX.

### 4.1 Peripheral Initialization 
Peripherals have to be started explicity. You must start the timer **after** the hardware init functions (`MX_TIM3_Init()`) and **before** the scheduler starts (`osKernelStart()`).

In `Core/Src/main.c`:

```c
/* USER CODE BEGIN 2 */
// Enable Timer 3 Channel 1 PWM output and start hardware counter
HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1);
/* USER CODE END 2 */ 
```

### 4.2 FreeRTOS Rules and the Task Loop
when using FreeRTOS the standard while(1) super loop is no longer nessesary. The RTOS scheduler takes over and runs tasks independently. 

Always use osDelay() after each task. osDelay() puts the task to sleep to X milliseconds, so the CPU doesn't get frozen and is free to let other tasks in the queue execute. 

### 4.3 Some Sample Task implementations
CubeMX creates your task function in `Core/Src/main.c`: Scroll down to the `StartDefaultTask` method to begin your code. 

Here is a sample breathing loop that ramps the PWM duty cycle up and down continuiously. 

```c
/* USER CODE BEGIN StartDefaultTask */
uint16_t duty = 0;
int16_t step = 10;

for(;;)
{
  // Write current duty cycle to TIM3 Channel 1 compare register
  __HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_1, duty);

  // Ramp duty cycle up and down between 0 and 1000
  duty += step;
  if (duty >= 1000) {
    duty = 1000;
    step = -10; // ramp back down
  } else if (duty <= 0) {
    duty = 0;
    step = 10;  // ramp back up
  }

  // Sleep for 15ms so other tasks can run while hardware handles PWM
  osDelay(15);
}
/* USER CODE END StartDefaultTask */ 
```
You can verify the result by connecting a jumper wire to PA6 pin to an LED that's in series with a 220 Ohm resistor. If setup correctly the LED will smoothly lower and increase the brightness. 













