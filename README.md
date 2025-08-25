# STM32 Accelerometer Movement Detection
Real-time 3-axis accelerometer monitoring with STM32, using DMA for ADC, offset calibration, averaging, and UART printf for easy debugging.

This project demonstrates a simple movement detection system using a 3-axis accelerometer connected to an STM32 microcontroller. The system reads analog accelerometer data via multiple ADC channels with DMA and determines movement directions.

## Features

- **3-axis Accelerometer Reading:** Measures X, Y, Z axes using ADC channels 0, 1, and 2.
- **DMA-based ADC Data Acquisition:** ADC readings are handled by DMA for efficient, non-blocking data transfer.
- **Offset Calibration:** Initial readings are used to determine offsets for each axis to correct for sensor biases.
- **Averaging:** Multiple samples are taken and averaged to smooth out noise.
- **Movement Detection:** Detects movement directions: Right, Left, Forward, Backward, Up, Down, or No Movement.
- **UART Output for Debugging:** Movement direction is printed to a serial terminal using UART and `printf`.

## Hardware Setup

- STM32 microcontroller (e.g., STM32F103)
- 3-axis accelerometer with analog output ADXL335
- ADC channels 0, 1, and 2 connected to X, Y, Z outputs of the accelerometer
- UART connection for debugging and output

## How It Works

### ADC + DMA
The accelerometer signals are read continuously by the ADC with DMA, storing results in a `uint16_t adc_value[3]` array. DMA ensures efficient data transfer without blocking the main loop.

### Offset Calibration
Each axis has a predefined offset (`X_offset`, `Y_offset`, `Z_offset`) calculated from initial readings to correct for stationary sensor bias.

### Averaging Samples
To reduce noise, `N` samples are taken for each axis, summed, and averaged:
```c
for (int i = 0; i < N; i++) {
    sum_x += ((float)(adc_value[0] - X_offset) / 4095.0f) * 3.3f / 0.3f;
    sum_y += ((float)(adc_value[1] - Y_offset) / 4095.0f) * 3.3f / 0.3f;
    sum_z += ((float)(adc_value[2] - Z_offset) / 4095.0f) * 3.3f / 0.3f;
    HAL_Delay(2);
}
vx = sum_x / N;
vy = sum_y / N;
vz = sum_z / N;
```
### UART & printf
- UART is configured at 115200 baud.
- The standard printf function is redirected to transmit data via UART.
- This allows sending messages to a serial terminal.
- Using UART + printf is a common and convenient method for debugging and monitoring sensor outputs in real time.
- Movement messages are printed every 500 ms.
The STM32 communicates with a NodeMCU via UART for debugging and monitoring.

### Wiring for UART

- STM32 **TX** → NodeMCU **TX**  
- STM32 **RX** ← NodeMCU **RX**  

> ⚠️ Important: Connect the **RESET** pin of the NodeMCU to **GND**.  
> This ensures the NodeMCU’s processor stays in reset mode, allowing it to act only as a UART-to-USB bridge without interfering with your STM32 communication.

### Monitoring

- Open a serial terminal (e.g., **RealTerm**) on your computer.  
- Set the baud rate to **115200**.  
- You can now view messages sent from STM32 using `printf()` over UART.
- 
### Periodic Check
- The check_movement() function is called in the main loop every 500 ms to update movement status.
- This provides regular updates without blocking other code.

### Notes
- Thresholds and offsets may need adjustment depending on the accelerometer and power supply.
- Z-axis output is influenced by gravity; consider this when interpreting up/down movement.
- Averaging improves stability but adds a small response delay.
- UART + printf is a standard method for debugging microcontroller projects.
