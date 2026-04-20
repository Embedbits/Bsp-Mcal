# MCAL — Microcontroller Abstraction Layer

> Standardized software interface to STM32 internal peripherals, built for modularity, portability, and testability.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Supported Peripherals](#supported-peripherals)
- [Repository Structure](#repository-structure)
- [Dependencies](#dependencies)
- [Initialization Flow](#initialization-flow)
- [Design Guidelines](#design-guidelines)
- [Usage Example](#usage-example)
- [Future Enhancements](#future-enhancements)
- [License](#license)
- [Authors](#authors)

---

## Overview

The **Microcontroller Abstraction Layer (MCAL)** sits directly on top of the hardware registers and provides a clean, unified API for accessing STM32 internal peripherals. Higher-level software components interact with hardware through this layer — without any knowledge of low-level register details.

MCAL is part of the **BSP (Board Support Package)** and leverages **CMSIS** and **STM32 LL (Low-Layer)** drivers for register-level access.

Key design goal: **one peripheral, one interface**. For example, the I2C driver can internally manage DMA without requiring the user to configure the DMA module separately — all dependency handling is encapsulated within the driver itself.

---

## Architecture

```
┌─────────────────────────────────────┐
│         Application Layer           │
├─────────────────────────────────────┤
│              MCAL                   │
│  (Gpio, Usart, Rcc, I2C, SPI, ...)  │
├─────────────────────────────────────┤
│   RAL — Register Abstraction Layer  │
│     (CMSIS + STM32 LL Drivers)      │
├─────────────────────────────────────┤
│         STM32 Hardware              │
└─────────────────────────────────────┘
```

---

## Supported Peripherals

| Peripheral | Description |
|---|---|
| GPIO | General-Purpose Input/Output |
| USART / UART | Serial communication |
| SPI | Serial Peripheral Interface |
| I2C | Inter-Integrated Circuit (with optional DMA) |
| Timers | General-purpose and advanced timers |
| ADC / DAC | Analog-to-Digital / Digital-to-Analog conversion |
| NVIC / EXTI | Interrupt controller and external interrupts |
| RCC | Clock and reset control |
| Flash | Flash memory access |
| PWR | Power and system configuration |

---

## Repository Structure

Each peripheral has its own dedicated folder containing:

| File | Purpose |
|---|---|
| `<Peripheral>.c` | Driver implementation |
| `<Peripheral>.h` | Public API declarations |
| `<Peripheral>_Port.h` | Hardware-specific port configuration |
| `<Peripheral>_Types.h` | Type definitions and enumerations |

```
Mcal/
├── Gpio/
│   ├── Gpio.c
│   ├── Gpio.h
│   ├── Gpio_Port.h
│   └── Gpio_Types.h
├── Usart/
│   ├── Usart.c
│   ├── Usart.h
│   ├── Usart_Port.h
│   └── Usart_Types.h
├── Rcc/
│   ├── Rcc.c
│   ├── Rcc.h
│   ├── Rcc_Port.h
│   └── Rcc_Types.h
├── I2c/
├── Spi/
├── Adc/
└── ...
```

---

## Dependencies

| Dependency | Purpose |
|---|---|
| **CMSIS** | Core interface to ARM Cortex-M processor and system peripherals |
| **STM32 LL Drivers** | Lightweight, low-overhead register access without HAL abstraction |

Both are included from the **STM32Cube Firmware Package** and integrated in the BSP under the RAL layer.

---

## Initialization Flow

The recommended startup sequence:

```
1. NVIC  — Interrupt controller setup
2. RCC   — System clock and voltage configuration
3. GPIO  — Pin multiplexing and output configuration
4. USART / SPI / I2C / ... — Peripheral initialization
5. DMA / Timers / SysTick  — Optional, as needed
```

This sequence can be executed as part of the system startup or triggered on demand by the application.

---

## Design Guidelines

- All hardware-specific code is encapsulated within MCAL — no register access outside this layer
- Consistent naming convention: `<Module>_<Action>` (e.g., `Gpio_Init`, `Usart_Send`, `Rcc_SetClock`)
- Macros and inline functions are used where performance is critical
- No hardcoded register values — CMSIS/LL macros are used throughout
- Each peripheral driver is self-contained and manages its own internal dependencies (e.g., DMA)
- All public APIs are documented

---

## Usage Example

### GPIO Initialization

```c
static gpioHal_PinConfig_t gpioHal_GpioConfiguration[GPIOHAL_IO_CNT] =
{
    {
        .GpioId = GPIOHAL_IO_USR_BTN,
        .ConfigStruct = {
            .PortId        = GPIO_PORT_C,
            .PinId         = GPIO_PIN_13,
            .PinMode       = GPIO_PIN_MODE_INPUT,
            .PinPull       = GPIO_PIN_PULL_NONE,
            .PinSpeed      = GPIO_PIN_SPEED_LOW,
            .PinOutType    = GPIO_PIN_OUTPUT_PUSHPULL,
            .PinAltFunction = GPIO_ALT_FUNC_CNT,
            .PinActiveLevel = GPIO_PIN_LEVEL_HIGH
        }
    }
};

void GpioHal_Init(void)
{
    for (uint16_t i = 0u; i < GPIOHAL_IO_CNT; i++)
    {
        Gpio_Init(&gpioHal_GpioConfiguration[i].ConfigStruct);
    }
}
```

---

## Future Enhancements

- [ ] Integration testing framework
- [ ] Unit test coverage for each peripheral driver
- [ ] Support for additional STM32 families

---

## License

This project is licensed under the **Creative Commons Attribution–NonCommercial 4.0 International (CC BY-NC 4.0)**.

You are free to use, modify, and share this work for **non-commercial purposes**, provided appropriate credit is given.

See [LICENSE.md](LICENSE.md) for full terms or visit [creativecommons.org/licenses/by-nc/4.0](https://creativecommons.org/licenses/by-nc/4.0/).

---

## Authors

- **Mr.Nobody** — [embedbits.com](https://embedbits.com)

Contributions are welcome! Please open a pull request.

---

## 🌐 Useful Links

- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
- [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/)
- [CC BY-NC 4.0 License](https://creativecommons.org/licenses/by-nc/4.0/)
