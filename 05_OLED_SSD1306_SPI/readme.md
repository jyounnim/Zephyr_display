# 5. OLED SSD1306 (0.96") — SPI Mode

**[한국어 버전](readme_kr.md)**

## What You'll Learn in This Lab

You'll connect the **exact same chip (SSD1306)** used in Lab 3 (I2C mode), but this time in **SPI mode**. Zephyr's SSD1306 driver supports both I2C and SPI buses in a single driver (internally branching via `DT_ON_BUS(node_id, spi)`) — so the **application code is practically identical to Lab 3**, and only the overlay changes.

## What You'll Need

- 0.96" OLED, SSD1306, **7-pin SPI module** (VCC/GND/SCK/SDA(MOSI)/RES/DC/CS)

> ⚠️ The 4-pin I2C-only module used in Lab 3 will NOT work for this lab. Make sure your module breaks out the SPI-specific pins (especially DC and RES) before you start.

## Wiring — Reusing the Same SPI Bus as Lab 4

| Signal | ESP32-S3 Connection |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCK | GPIO12 |
| SDA(MOSI) | GPIO11 |
| CS | GPIO10 |
| DC | GPIO21 |
| RES | GPIO14 |

(MOSI=11, MISO=13, SCK=12 are the same pins used since the non-OS curriculum's SPI labs — the SSD1306 is write-only so MISO isn't actually used, but the bus itself is kept identical so it can be shared with other SPI devices)

## Folder Structure

```
Zephyr_display/
└── 05_OLED_SSD1306_SPI/
    ├── lab/
    │   ├── src/
    │   │   └── main.c
    │   ├── boards/
    │   │   └── esp32s3_devkitc_esp32s3_procpu.overlay
    │   ├── CMakeLists.txt
    │   ├── prj.conf
    │   └── sample.yaml
    ├── readme_kr.md
    └── readme.md
```

## Devicetree Overlay

```dts
&pinctrl {
    spim2_default: spim2_default {
        group1 {
            pinmux = <SPIM2_MISO_GPIO13>, <SPIM2_SCLK_GPIO12>;
        };
        group2 {
            pinmux = <SPIM2_MOSI_GPIO11>;
            output-low;
        };
    };
};

&spi2 {
    #address-cells = <1>;
    #size-cells = <0>;
    status = "okay";
    pinctrl-0 = <&spim2_default>;
    pinctrl-names = "default";
    cs-gpios = <&gpio0 10 GPIO_ACTIVE_LOW>;

    oled_spi: ssd1306@0 {
        compatible = "solomon,ssd1306";
        reg = <0>;
        spi-max-frequency = <4000000>;
        data-cmd-gpios = <&gpio0 21 GPIO_ACTIVE_HIGH>;
        reset-gpios = <&gpio0 14 GPIO_ACTIVE_LOW>;
        width = <128>;
        height = <64>;
        segment-offset = <0>;
        page-offset = <0>;
        display-offset = <0>;
        multiplex-ratio = <63>;
        segment-remap;
        com-invdir;
        prechargep = <0x22>;
    };
};

/ {
    chosen {
        zephyr,display = &oled_spi;
    };
};
```

## prj.conf

```
CONFIG_SPI=y
CONFIG_DISPLAY=y
CONFIG_CHARACTER_FRAMEBUFFER=y
CONFIG_SSD1306=y
CONFIG_HEAP_MEM_POOL_SIZE=16384
```

## Code — Nearly Identical to Lab 3

```c
#include <zephyr/kernel.h>
#include <zephyr/device.h>
#include <zephyr/drivers/display.h>
#include <zephyr/display/cfb.h>
#include <stdio.h>

#define DISPLAY_STACK_SIZE 2048
#define DISPLAY_PRIORITY   5

static void display_thread_entry(void *p1, void *p2, void *p3) {
    const struct device *dev = DEVICE_DT_GET(DT_CHOSEN(zephyr_display));

    if (!device_is_ready(dev)) {
        printk("DisplayThread: display device not ready\n");
        return;
    }

    if (display_set_pixel_format(dev, PIXEL_FORMAT_MONO10) != 0) {
        display_set_pixel_format(dev, PIXEL_FORMAT_MONO01);
    }

    if (cfb_framebuffer_init(dev)) {
        printk("DisplayThread: framebuffer init failed\n");
        return;
    }

    cfb_framebuffer_clear(dev, true);
    display_blanking_off(dev);

    printk("DisplayThread: ready (SPI mode)\n");

    int counter = 0;
    while (1) {
        char buf[32];
        snprintf(buf, sizeof(buf), "Count: %d", counter++);

        cfb_framebuffer_clear(dev, false);
        cfb_print(dev, "SSD1306 (SPI)", 0, 0);
        cfb_print(dev, buf, 0, 16);
        cfb_framebuffer_finalize(dev);

        k_sleep(K_SECONDS(1));
    }
}

K_THREAD_DEFINE(display_id, DISPLAY_STACK_SIZE, display_thread_entry,
                NULL, NULL, NULL, DISPLAY_PRIORITY, 0, 0);

int main(void) {
    printk("main: started, DisplayThread is running independently\n");
    return 0;
}
```

## Build & Run

```powershell
west build -p always -b esp32s3_devkitc/esp32s3/procpu .\05_OLED_SSD1306_SPI\lab\
west flash
west espressif monitor
```

## Run & Verify

- Confirm that "SSD1306 (SPI)" and a counter are displayed on the screen

## Observation Points — Side-by-Side Comparison with Lab 3

| | Lab 3 (I2C) | Lab 5 (SPI) |
|---|---|---|
| Number of signal lines | 2 (SDA/SCL) | 4 (SCK/MOSI/CS) + DC/RES |
| Overlay `compatible` | `solomon,ssd1306` (I2C binding) | `solomon,ssd1306` (SPI binding, same name) |
| Addressing | `reg = <0x3c>` (I2C address) | `reg = <0>` (SPI CS index) + `data-cmd-gpios`/`reset-gpios` added |
| **Application code (`main.c`)** | **Identical** | **Identical** |

**The key is that last row** — even though the bus is completely different, `main.c` is identical except for a single string ("(I2C)" → "(SPI)"). This is the **abstraction of Zephyr's driver model** that was repeatedly emphasized in Labs 3 and 4, actually at work. As long as you use the Display + CFB API, the application doesn't need to care whether the bus underneath is I2C or SPI.

## Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| Nothing shows on the screen | Verify you have the 7-pin SPI module (a 4-pin I2C-only module can't work at all) |
| `device_is_ready()` returns false | Check the `data-cmd-gpios`/`reset-gpios` polarity and the `cs-gpios` polarity (`GPIO_ACTIVE_LOW`) |
| Lab 4's loopback test passed, but the screen doesn't show anything | Lab 4 only verified the bus itself — the DC/RES pins were not part of that test, so check the wiring/polarity of these two pins separately |
| I2C mode (Lab 3) worked, but only SPI mode fails | If you enabled both `&i2c0` and `&spi2` in the same overlay, you can't set `chosen { zephyr,display = ...}` for both buses at once (only one is selected) — make sure only the SPI side is set as `chosen` in this lab |

## Next

Lab 6 (`06_TFT_ST7789V3`) covers another SPI-based display, the ST7789V3 color TFT — unlike the monochrome OLED you've used so far, you'll learn how to draw full-color graphics.
