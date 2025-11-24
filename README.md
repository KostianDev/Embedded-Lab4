**Embedded Lab 4 - ADC -> TIM4 PWM LEDs**

STM32F407 project that reads three potentiometers (PA3, PA4, PA5) with ADC1 and controls three on-board LEDs using TIM4 PWM channels. Firmware is generated with STM32CubeMX and built with the supplied Makefile.

**Hardware**
- **MCU**: `STM32F407VGT6`
- **Potentiometers (ADC inputs)**: PA3 (ADC1_IN3), PA4 (ADC1_IN4), PA5 (ADC1_IN5)
- **LEDs (TIM4 PWM outputs)**: PD12 - `LED_RED` (TIM4_CH1 in this project mapping), PD13 - `LED_YELLOW` (TIM4_CH2), PD14 - `LED_BLUE` (TIM4_CH3)
- **Debug/programming**: SWD (ST-LINK)

**What this lab does**
- Reads three analog inputs (potentiometers) and updates LED brightness proportionally using PWM on TIM4.
- ADC is configured to sample 3 regular channels (IN3/IN4/IN5). DMA (circular) is used to transfer ADC samples to memory at low CPU cost.
- A small integer IIR (exponential moving average) filter smooths ADC values to reduce visible flicker at low brightness.

**Implementation notes**
- ADC configuration: `ScanConvMode = ENABLE`, `NbrOfConversion = 3`, `DMAContinuousRequests = ENABLE`, `EOCSelection = ADC_EOC_SEQ_CONV`.
- DMA: `DMA2_Stream0` configured in circular mode, peripheral-to-memory, half-word alignment. CubeMX generated `MX_DMA_Init()` and linked DMA to ADC in `HAL_ADC_MspInit`.
- Buffer: `volatile uint16_t adc_dma_buf[3]` - DMA fills this buffer with the latest three ADC samples in order: index 0 -> IN3 (PA3), 1 -> IN4 (PA4), 2 -> IN5 (PA5).
- Smoothing: simple integer IIR filter (filtered = (filtered*(N-1)+sample)/N) implemented in `main.c` (default `N = 6`). A small deadzone disables PWM for very small ADC values to avoid flicker.
- PWM: TIM4 channels 1..3 used. PWM is started in `main.c` and compare values are updated from filtered ADC readings.

**Build**
Run in project root:
```bash
make
```
Artifacts are placed under `build/` (for example `build/Lab4.bin`, `build/Lab4.hex`).

**Flash**
Use your preferred SWD programmer. Examples:

STM32CubeProgrammer CLI:
```bash
STM32_Programmer_CLI -c port=SWD -w build/Lab4.bin 0x08000000 -v -rst
```

ST-LINK (stlink-tools):
```bash
st-flash --reset write build/Lab4.bin 0x08000000
```

**Run**
- After reset the firmware starts ADC+DMA and PWM. Turn each potentiometer to change the corresponding LED brightness.
