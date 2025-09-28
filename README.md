# C_training_0

// embedded C on an STM32 microcontroller

Using printf in embedded C on an STM32 microcontroller typically involves retargeting the standard library's fputc function to send characters over a chosen peripheral, most commonly UART or the Serial Wire Viewer (SWV).
1. Retargeting printf via UART:
This method directs printf output to a Universal Asynchronous Receiver/Transmitter (UART) peripheral, which can then be read by a terminal emulator on a connected computer.
C

#include <stdio.h>
#include "stm32f4xx_hal.h" // Or relevant HAL library for your STM32 series

extern UART_HandleTypeDef huart2; // Declare your UART handle (e.g., for UART2)

// Override the _write function (or fputc for some toolchains)
int _write(int file, char *ptr, int len)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)ptr, len, HAL_MAX_DELAY);
    return len;
}

// In your main function or a suitable initialization function:
void init_printf_uart(void)
{
    // Initialize your UART peripheral (e.g., MX_USART2_UART_Init() if using CubeMX)
    // ...
}

// Example usage in your main loop:
int main(void)
{
    // ... other initializations ...
    init_printf_uart();

    while (1)
    {
        printf("Hello from STM32! Counter: %d\r\n", counter++);
        HAL_Delay(1000);
    }
}
2. Retargeting printf via Serial Wire Viewer (SWV):
This method uses the SWV feature of ARM Cortex-M microcontrollers to send debug messages over the SWD interface to a debugger's console. This is often preferred during development as it doesn't consume a UART peripheral.
C

#include <stdio.h>
#include "stm32f4xx_hal.h" // Or relevant HAL library for your STM32 series

// Override the _write function for SWV
int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++)
    {
        ITM_SendChar(*ptr++); // Send character via ITM
    }
    return len;
}

// Example usage in your main loop:
int main(void)
{
    // ... other initializations ...

    while (1)
    {
        printf("SWV Debug Message: %s\n", "Hello!");
        HAL_Delay(500);
    }
}
Important Considerations:
Include <stdio.h>: Always include this header file to use printf.
Toolchain Specifics: The exact function to override (_write or fputc) might vary slightly depending on your compiler and IDE (e.g., GCC, Keil, IAR).
Buffer Management: For performance or memory constraints, consider disabling I/O buffering for stdout using setvbuf(stdout, NULL, _IONBF, 0); if needed.
SWV Setup: When using SWV, ensure your debugger configuration (e.g., in STM32CubeIDE) has SWV enabled and the ITM data console is configured to receive output.
