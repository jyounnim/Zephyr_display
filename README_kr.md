# ESP32-S3 Zephyr 디스플레이 실습 시리즈

**[English version](README.md)**

**ESP32-S3-DevKitC-1** 보드에서 **Zephyr RTOS**로 다양한 디스플레이를 구동해보는 8개 실습 시리즈입니다. I2C 버스 스캔부터 시작해 문자 LCD, I2C/SPI 두 방식의 모노크롬 OLED를 거쳐, 마지막에는 SPI 컬러 TFT 2종까지 다룹니다. 각 실습은 독립적으로 진행할 수 있도록 배선/빌드/코드 설명을 매번 처음부터 전부 담아뒀습니다 — 다른 실습을 먼저 하지 않아도 어느 랩이든 바로 시작할 수 있습니다.

## 실습 목록

| 랩 | 주제 | 버스 | 내용 | 문서 |
| --- | --- | --- | --- | --- |
| 01 | I2C 버스 스캐너 | I2C0 | 부팅 시 I2C0을 스캔해 `i2cdetect` 스타일 주소 그리드를 출력 — 시리즈 전체에서 계속 재사용하는 진단 도구 | [KR](01_I2C_bus_scanner/readme_kr.md) · [EN](01_I2C_bus_scanner/readme.md) |
| 02 | I2C LCD | I2C0 | PCF8574 I2C 백팩으로 16x2 HD44780 캐릭터 LCD 구동 | [KR](02_I2C_LCD_LAB/readme_kr.md) · [EN](02_I2C_LCD_LAB/readme.md) |
| 03 | OLED SSD1306 (I2C) | I2C0 | Zephyr 디스플레이 서브시스템으로 128x64 SSD1306 모노크롬 OLED를 I2C로 구동 | [KR](03_OLED_SSD1306_I2C/readme_kr.md) · [EN](03_OLED_SSD1306_I2C/readme.md) |
| 04 | SPI 기초 | SPI2 | 디스플레이 없이 MOSI↔MISO를 직결한 SPI 루프백 자체 테스트 — 이후 모든 SPI 랩의 배선 기반 | [KR](04_SPI_basics/readme_kr.md) · [EN](04_SPI_basics/readme.md) |
| 05 | OLED SSD1306 (SPI) | SPI2 | 03번과 동일한 SSD1306 패널을 이번엔 SPI로 구동 — I2C/SPI 두 방식을 나란히 비교 | [KR](05_OLED_SSD1306_SPI/readme_kr.md) · [EN](05_OLED_SSD1306_SPI/readme.md) |
| 06 | TFT ST7789V3 | SPI2 | 커스텀 mipi-dbi 스타일 devicetree 바인딩으로 컬러 SPI TFT(ST7789V3) 구동 | [KR](06_TFT_ST7789V3/readme_kr.md) · [EN](06_TFT_ST7789V3/readme.md) |
| 07 | Nokia 5110 디스플레이 | SPI2 | 클래식 Nokia 5110(PCD8544) 모노크롬 LCD를 SPI로 구동 | [KR](07_Nokia5110_display/readme_kr.md) · [EN](07_Nokia5110_display/readme.md) |
| 08 | TFT ST7735 | SPI2 | 두 번째 컬러 SPI TFT(ST7735) — 시리즈의 마지막 실습 | [KR](08_TFT_ST7735/readme_kr.md) · [EN](08_TFT_ST7735/readme.md) |

각 랩 폴더 구성은 모두 동일합니다:

```
NN_LAB_NAME/
├── readme.md        영문 문서 (GitHub에서 폴더를 열면 자동으로 렌더링됨)
├── readme_kr.md      한글 문서
└── lab/               랩 코드 (src/, boards/*.overlay, CMakeLists.txt, prj.conf, sample.yaml 등)
```

06번 랩은 추가로 `troubleshooting_kr.md` / `troubleshooting_en.md`를 갖고 있습니다 — 해당 디스플레이를 실제로 붙이면서 겪은 디버깅 과정을 정리한 문서입니다.

## 준비물

- 보드: **ESP32-S3-DevKitC-1**
- 프레임워크: **Zephyr RTOS**, 빌드 타겟 `esp32s3_devkitc/esp32s3/procpu`
- 툴체인: `west`(Zephyr 메타 도구), 동작하는 Zephyr SDK / west 워크스페이스

## 빌드 및 실행 (공통)

```powershell
west build -p always -b esp32s3_devkitc/esp32s3/procpu .\NN_LAB_NAME\lab\
west flash
west espressif monitor
```

예를 들어 06번 랩을 빌드/플래시하려면:

```powershell
west build -p always -b esp32s3_devkitc/esp32s3/procpu .\06_TFT_ST7789V3\lab\
west flash
west espressif monitor
```

## 전체 랩 공통 컨벤션

- **I2C0** (01~03번에서 사용): SDA = **GPIO8**, SCL = **GPIO9** — ESP32-S3 Arduino 프레임워크의 I2C 기본 핀과 동일해서, non-OS 커리큘럼에서 쓰던 배선을 그대로 재사용할 수 있습니다.
- **SPI2** (04~08번에서 사용): 가능한 한 이전 랩에서 쓰던 핀을 재사용합니다 — 정확한 배선표는 각 랩 문서를 참고하세요.
- **GPIO 컨트롤러 레이블**: `&gpio0`은 GPIO0~31, `&gpio1`은 GPIO32~53을 담당합니다. 스트래핑 핀(GPIO0/3/45/46)과 USB-JTAG 핀(GPIO19/20)은 시리즈 전체에서 피합니다.
- **커스텀 devicetree 바인딩** (04, 06, 07, 08번 각각 자체 바인딩 정의): 랩이 직접 바인딩을 정의하는 경우 Zephyr 공식 `mipi-dbi-spi` 바인딩의 프로퍼티 이름(`reset-gpios`, `dc-gpios`)을 그대로 따릅니다. 반면 Zephyr 공식 바인딩을 그대로 쓰는 랩(예: 03/05번의 `solomon,ssd1306-*`)은 그 바인딩의 실제 프로퍼티 이름(예: `dc-gpios`가 아니라 `data-cmd-gpios`)을 사용합니다.
- **`SPI_DT_SPEC_GET(node_id, operation_)`**는 이 프로젝트의 Zephyr 체크아웃 기준 인자가 정확히 2개입니다 — 3번째 인자를 추가로 넘기면 옵션이 아니라 빌드 에러가 됩니다.

## 파일 구성

```
zephyr_display/
├── README.md
├── README_kr.md                        본 문서
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

**[English version](README.md)**
