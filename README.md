# STM32F407xx Bare-Metal Peripheral Drivers

A register-level C driver collection for the **STM32F407xx** family (ARM Cortex-M4). The project explores peripheral initialization, configuration, data transfer, and interrupt handling without using STM32 HAL.

## Implemented modules

| Module | Current scope |
|---|---|
| GPIO | Pin mode, output type, speed, pull-up/down, alternate function, EXTI-related configuration and handling |
| RCC | System/peripheral clock-frequency queries and PLL clock calculation |
| SPI | Master/slave configuration, polling transfers, interrupt-driven TX/RX, status flags and callbacks |
| I2C | Master/slave communication, polling and interrupt-driven transfers, ACK/repeated-start handling and error events |
| USART | Baud-rate configuration, polling and interrupt-driven TX/RX, status and error events |

> This is a learning and development codebase. The presence of an API does not imply that every mode, peripheral instance, or error path has been validated on hardware. See the limitations below and the test evidence added to the repository.

## Repository layout

```text
.
├── Inc/
│   ├── gpio.h
│   ├── i2c.h
│   ├── rcc.h
│   ├── spi.h
│   ├── usart.h
│   └── drivers.h
└── Src/
    ├── gpio.c
    ├── i2c.c
    ├── rcc.c
    ├── spi.c
    └── usart.c
```

The STM32 device-register header (`stm32f407xx.h`) is an external project dependency and is not included in this source archive. Add the appropriate CMSIS/device header and startup/linker configuration for your chosen STM32F407 board. Keep vendor-provided files and their licenses separate from original driver code.

## Design approach

- Direct peripheral-register access through memory-mapped register structures.
- Configuration and handle structures for peripheral setup and transfer state.
- Polling and interrupt-driven transfer APIs where implemented.
- Application callbacks for selected interrupt events.
- Peripheral clock and GPIO alternate-function setup are explicit parts of initialization.

## Build and use

1. Add `Inc/` to the compiler include path.
2. Add the required files from `Src/` to your STM32F407 project.
3. Provide the matching STM32F407 CMSIS/device header, startup code, linker script, and system clock setup.
4. Configure GPIO alternate functions and peripheral clocks according to the selected pins and peripheral instance.
5. Build for the exact STM32F407 part and test each peripheral on hardware.

No universal build command is provided yet because the uploaded source set does not include a complete standalone build system or board project.

## Validation status

Hardware validation should be documented per module. Add a table here with the exact board, peripheral instance, pin mapping, clock, test procedure, and result. Useful evidence includes logic-analyzer captures for SPI/I2C/UART and oscilloscope captures for timing and clock signals.

## Known review items

The current source snapshot needs a correctness and robustness pass before being presented as a production-ready library. Initial static-review findings include:

- **NVIC helper indexing:** the SPI and I2C IRQ helper implementations contain incorrect ISER/ICER register selection/indexing in some IRQ ranges. Audit the GPIO/USART copies too; use `IRQn / 32` and `IRQn % 32`, and validate the supported IRQ range.
- **SPI 16-bit buffer handling:** expressions such as `(uint16_t *)pBuffer++` increment the byte pointer before the cast takes effect. Use a typed temporary pointer or increment the original byte pointer by two. Also validate that the transfer length is compatible with the configured frame width.
- **SPI transfer completion:** full-duplex receive behavior, overrun handling, end-of-transfer sequencing, and timeout behavior need explicit tests. A transmit-only operation still receives data on the STM32 SPI peripheral.
- **USART peripheral clock control:** the current enable path does not cover every USART/UART instance declared by the device header, and the disable path is unfinished.
- **USART baud-rate configuration:** validate the baud-rate calculation for both oversampling modes and all supported APB clock configurations.
- **I2C robustness:** review every wait loop for a timeout, check error-flag clearing sequences against the reference manual, and test 1-byte, 2-byte, and multi-byte receive edge cases.
- **API consistency:** standardize return codes, const-correctness, naming, parameter validation, and documentation. Avoid silently blocking forever on a peripheral flag.

These are review findings from a static inspection of the supplied source snapshot, not a claim that every issue has been reproduced on hardware.

## Roadmap

- Add bounded timeouts and explicit error return codes to blocking operations.
- Complete and test IRQ mapping for all supported peripheral instances.
- Add DMA-based transfers where useful.
- Add unit tests for pure configuration/calculation helpers and hardware-in-the-loop tests for peripheral behavior.
- Publish pin maps, clock assumptions, test logs, and analyzer captures.

## License

Add a license only after confirming that all included source files are yours to license and that any external CMSIS/device files retain their original licensing and attribution.
