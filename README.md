# ESP32-S3 Zephyr Display Examples

**[한국어 버전](README_kr.md)**

An eight-lab series that walks through driving displays on an **ESP32-S3-DevKitC-1** under **Zephyr RTOS** - starting from a plain I2C bus scan, through a character LCD and monochrome OLEDs on both I2C and SPI, and ending with two SPI color TFTs. Every lab is built to stand on its own: wiring, build steps, and code are explained in full each time, so you can jump into any single lab without having done the others first.

## Lab list

| Lab | Topic | Bus | What it covers | Docs |
| --- | --- | --- | --- | --- |
| 01 | I2C Bus Scanner | I2C0 | Scans I2C0 at boot and prints an `i2cdetect`-style address grid - the diagnostic tool reused throughout the whole series | [KR](01_I2C_bus_scanner/readme_kr.md) · [EN](01_I2C_bus_scanner/readme.md) |
| 02 | I2C LCD | I2C0 | PCF8574 I2C backpack driving a 16x2 HD44780 character LCD | [KR](02_I2C_LCD_LAB/readme_kr.md) · [EN](02_I2C_LCD_LAB/readme.md) |
| 03 | OLED SSD1306 (I2C) | I2C0 | SSD1306 128x64 monochrome OLED over I2C, using Zephyr's display subsystem | [KR](03_OLED_SSD1306_I2C/readme_kr.md) · [EN](03_OLED_SSD1306_I2C/readme.md) |
| 04 | SPI Basics | SPI2 | A SPI loopback self-test (MOSI wired back to MISO) with no display hardware - the baseline every later SPI lab in this series reuses | [KR](04_SPI_basics/readme_kr.md) · [EN](04_SPI_basics/readme.md) |
| 05 | OLED SSD1306 (SPI) | SPI2 | The same SSD1306 panel as Lab 03, this time over SPI instead of I2C - a direct side-by-side comparison of the two buses | [KR](05_OLED_SSD1306_SPI/readme_kr.md) · [EN](05_OLED_SSD1306_SPI/readme.md) |
| 06 | TFT ST7789V3 | SPI2 | A color SPI TFT (ST7789V3) driven through a custom mipi-dbi-style devicetree binding | [KR](06_TFT_ST7789V3/readme_kr.md) · [EN](06_TFT_ST7789V3/readme.md) |
| 07 | Nokia 5110 Display | SPI2 | The classic Nokia 5110 (PCD8544) monochrome LCD over SPI | [KR](07_Nokia5110_display/readme_kr.md) · [EN](07_Nokia5110_display/readme.md) |
| 08 | TFT ST7735 | SPI2 | A second color SPI TFT (ST7735), the final lab of the series | [KR](08_TFT_ST7735/readme_kr.md) · [EN](08_TFT_ST7735/readme.md) |

Each lab folder is laid out the same way:

```
NN_LAB_NAME/
├── readme.md        English write-up (GitHub renders this automatically when you open the folder)
├── readme_kr.md      Korean write-up
└── lab/               lab code (src/, boards/*.overlay, CMakeLists.txt, prj.conf, sample.yaml, ...)
```

Lab 06 additionally ships `troubleshooting_kr.md` / `troubleshooting_en.md`, documenting the real debugging trail hit while bringing that display up.

## Requirements

- Board: **ESP32-S3-DevKitC-1**
- Framework: **Zephyr RTOS**, build target `esp32s3_devkitc/esp32s3/procpu`
- Toolchain: `west` (Zephyr's meta-tool), a working Zephyr SDK / west workspace

## Building & running any lab

```powershell
west build -p always -b esp32s3_devkitc/esp32s3/procpu .\NN_LAB_NAME\lab\
west flash
west espressif monitor
```

For example, to build and flash Lab 06:

```powershell
west build -p always -b esp32s3_devkitc/esp32s3/procpu .\06_TFT_ST7789V3\lab\
west flash
west espressif monitor
```

## Conventions shared across labs

- **I2C0** (used by Labs 01-03): SDA = **GPIO8**, SCL = **GPIO9** - matches the ESP32-S3 Arduino framework's default I2C pins, so existing wiring from the non-OS curriculum can be reused as-is.
- **SPI2** (used by Labs 04-08): pins are reused from lab to lab wherever possible; see each lab's own doc for its exact wiring table.
- **GPIO controller labels**: `&gpio0` covers GPIO0-31, `&gpio1` covers GPIO32-53. Strapping pins (GPIO0/3/45/46) and the USB-JTAG pins (GPIO19/20) are avoided throughout the series.
- **Custom devicetree bindings** (Labs 04, 06, 07, 08 each define their own): property names follow Zephyr's official `mipi-dbi-spi` binding naming (`reset-gpios`, `dc-gpios`) where the lab defines its own binding; a lab built on an official Zephyr binding (e.g. Lab 03/05's `solomon,ssd1306-*`) uses that binding's real property names instead (e.g. `data-cmd-gpios`, not `dc-gpios`).
- **`SPI_DT_SPEC_GET(node_id, operation_)`** takes exactly two arguments on this project's Zephyr checkout - a third trailing argument is a build error, not an optional flag.

## File layout

```
zephyr_display/
├── README.md                          this file
├── README_kr.md
├── 01_I2C_bus_scanner/
│   ├── readme.md
│   ├── readme_kr.md
│   └── lab/
│       ├── src/main.c
│       ├── boards/esp32s3_devkitc_esp32s3_procpu.overlay
│       ├── CMakeLists.txt
│       ├── prj.conf
│       └── sample.yaml
├── 02_I2C_LCD_LAB/
├── 03_OLED_SSD1306_I2C/
├── 04_SPI_basics/
├── 05_OLED_SSD1306_SPI/
├── 06_TFT_ST7789V3/        (+ troubleshooting_kr.md / troubleshooting_en.md)
├── 07_Nokia5110_display/
└── 08_TFT_ST7735/
```

**[한국어 버전](README_kr.md)**
