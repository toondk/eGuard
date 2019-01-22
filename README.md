# eGuard_octa

## About
The eGuard is a device which tracks the health of an electronic device by using sensors like an accelerometer, a light sensor, a temperature and humidity sensor, ...

While programming, we paid attention to the low power aspect which is needed in the world of IoT to let devices run several years on a single battery.

```sequence
Title: Workflow

1 -> 2: Drivers
2 -> 3: ISR
3 -> 4: Power Management
4 -> 5: Main Function


1 -> 2: TTN and TB
2 -> 3: Database
3 -> 4: Fingerprinting
4 -> 5: Final Setup

1 -> 3: GPS I2C
2 -> 4: GPS Parse
4 -> 5: GPS Send
```

## 1. Information

University of Antwerp
Course: I-IoT Low Power Embedded Systems
Professsor/teacher: Maarten Weyn, Michiel Aernouts, Mats De Meyer
Master Students: Thomas Verellen, Robin Janssens, Toon De Keyzer

## 2. Hardware

- Nucleo L496ZG
- Octa connect shield
- Octa connect Murata modem shield
- Octa connnect GPS shield

![Imgur: The magic of the Internet](https://imgur.com/fKzjctM)


## 3. Setup

### 3.1 Hardware

- Octa Board
  - Mount the Murata modem shield on P1
  - Mount the GPS shield on P2

- Backend
  - MongoDB
  - Fingerprinting
  - MQTT broker
  - The Things Network
  - Thingsboard

### 3.2 Configure
- Add your device's ABP keys to `template_keys.h` and rename this file to `keys.h`

### 3.3 Drivers
- This repository uses modified drivers. To install these correctly place the 
	following files in the correct folders:
	- Move the .h files in `drivers/include/` to `RIOTBASE/drivers/include/`
	- Move the folders with .c files in `drivers/drivers/` to `RIOTBASE/drivers/`

- Edit the `RIOTBASE/drivers/Makefile.dep` file and add the following code:
```
ifneq (,$(filter lsm303agr,$(USEMODULE)))
  FEATURES_REQUIRED += periph_i2c
endif

ifneq (,$(filter xm1110, $(USEMODULE)))
  USEMODULE += xtimer
  FEATURES_REQUIRED += periph_i2c
endif

ifneq (,$(filter tcs34725,$(USEMODULE)))
  FEATURES_REQUIRED += periph_i2c
endif
```

- Edit the `RIOTBASE/drivers/Makefile.include` file and add the following code:
```
ifneq (,$(filter lsm303agr,$(USEMODULE)))
  USEMODULE_INCLUDES += $(RIOTBASE)/drivers/lsm303agr/include
endif

ifneq (,$(filter xm1110,$(USEMODULE)))
  USEMODULE_INCLUDES += $(RIOTBASE)/drivers/xm1110/include
endif

ifneq (,$(filter tcs34725,$(USEMODULE)))
  USEMODULE_INCLUDES += $(RIOTBASE)/drivers/tcs34725/include
endif
```

- Edit the `RIOTBASE/boards/octa/include/periph_conf.h` file en replace the I2C-config with the following:
```
static const i2c_conf_t i2c_config[] = {
    {
        .dev            = I2C1,
        .speed          = I2C_SPEED_NORMAL,
        .scl_pin        = GPIO_PIN(PORT_B, 8),
        .sda_pin        = GPIO_PIN(PORT_B, 9),
        .scl_af         = GPIO_AF4,
        .sda_af         = GPIO_AF4,
        .bus            = APB1,
        .rcc_mask       = RCC_APB1ENR1_I2C1EN,
        .irqn           = I2C1_EV_IRQn
    },
    {
        .dev            = I2C2,
        .speed          = I2C_SPEED_NORMAL,
        .scl_pin        = GPIO_PIN(PORT_F, 0),
        .sda_pin        = GPIO_PIN(PORT_F, 1),
        .scl_af         = GPIO_AF4,
        .sda_af         = GPIO_AF4,
        .bus            = APB1,
        .rcc_mask       = RCC_APB1ENR1_I2C2EN,
        .irqn           = I2C2_EV_IRQn
    }
};
```
