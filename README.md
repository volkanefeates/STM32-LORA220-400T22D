# STM32-LORA220-400T22D


Bu proje, EBYTE E220 serisi (ör. LORA220-400T22D) modüllerini STM32 (HAL) ile yapılandırmak ve geçici ayarlarını değiştirmek için hazırlanmış bir örnek kütüphaneyi içerir.

Bu repodaki örnek kodlar şunları yapar:

- Modülün mevcut ayarlarını UART üzerinden okur.
- Kodu içindeki `lora_module.settings` yapısını kullanarak yeni (geçici) ayarları modüle yazar.
- Ayarların RAM'e yazıldığını doğrular.

Hangi pinleri bağlamanız gerektiği (genel rehber):

- LORA Modül Pinleri (tipik): VCC (3.3V), GND, TX, RX, M0, M1, AUX
- STM32 (örnek):
  - LORA TX -> MCU USART1_RX (ör. PA10)
  - LORA RX -> MCU USART1_TX (ör. PA9)
  - LORA VCC -> 3.3V (mutlaka 3.3V)
  - LORA GND -> GND
  - LORA M0 -> MCU GPIO (ör. GPIOB, M0_Pin)
  - LORA M1 -> MCU GPIO (ör. GPIOB, M1_Pin)
  - LORA AUX -> MCU GPIO input (AUX_GPIO_Port, AUX_Pin)

Not: `main.c` içinde modül için kullanılan UART: `huart1` (bu kodda modül haberleşmesi için USART1 kullanılıyor). Log/debug için `huart2` (115200) kullanılır.

Önerilen (örnek) pin eşlemesi — board'a göre değişebilir, lütfen `main.h` veya CubeMX (ioc) dosyanızdaki `M0_Pin`, `M1_Pin`, `AUX_Pin` tanımlarını kontrol edin:

- USART1: PA9 = TX, PA10 = RX (modül <-> MCU seri bağlantısı için)
- USART2: PA2 = TX, PA3 = RX (debug/log için; kod örneğinde huart2 115200)
- M0, M1: PBx (kodda GPIOB kullanılıyor; hangi PBx olduğunu `main.h` tanımları belirler)
- AUX: herhangi bir giriş pini (kodda `AUX_GPIO_Port` ile tanımlıdır)

CubeMX / STM32 Ayar Önerileri:

1. USART1'i (Asynchronous) etkinleştir, Baudrate 9600, 8N1.
2. USART2'yi (Asynchronous) etkinleştir, Baudrate 115200 (debug/log).
3. GPIOB üzerindeki M0 ve M1 pinlerini: Output, Push-Pull, No Pull, Low Speed olarak ayarla.
4. AUX pinini: Input, No Pull olarak ayarla.
5. Kod jenerasyonu yap ve proje dosyalarındaki `M0_Pin`, `M1_Pin`, `AUX_Pin` tanımlarını not et.

Kısa kullanım notları (kod tarafı):

- `E220_Init(&lora_module, &huart1, M0_GPIO_Port, M0_Pin, M1_GPIO_Port, M1_Pin, AUX_GPIO_Port, AUX_Pin);`
  — Bu fonksiyona modülün UART handle'ı ve M0/M1/AUX pin port/pin tanımları verilir.
- Mod değiştirme (config <-> normal) için `E220_SetMode(dev, MODE_3_CONFIG)` vb. kullanılır. Kod, AUX pininin HIGH olmasını bekleyerek (handshake) çalışır.
- `E220_LoadSettingsFromModule` modülden (kalıcı ayarları da dahil) ayarları okur ve `dev->settings` içine doldurur.
- `E220_SaveSettingsToRAM` ile `dev->settings` içeriği geçici (RAM) ayar olarak modüle yazılır (kodda parça parça yazma yapılır).

Önemli notlar / Troubleshooting:

- Besleme gerilimini kesinlikle 3.3V ile sağlayın; 5V vermeyin.
- UART hatlarını çapraz bağlayın: MCU_TX -> MODULE_RX, MCU_RX -> MODULE_TX.
- AUX pini modülün işlem durumunu gösterir; kod AUX HIGH beklerken modülün hazır olduğunu belirtir.
- Eğer `E220_LoadSettingsFromModule` hata veriyorsa: önce fiziksel bağlantıları, güç ve UART baud/parity ayarlarını kontrol edin.

## English — Project summary and wiring guide

This repository contains an example HAL-based STM32 library to read and modify settings of the EBYTE E220 family (for example LORA220-400T22D). The sample code reads current module settings via UART, writes temporary settings into RAM, and verifies them.

Quick mapping (typical):

- LORA module pins: VCC (3.3V), GND, TX, RX, M0, M1, AUX
- MCU (example):
  - MODULE TX -> MCU USART1_RX (e.g. PA10)
  - MODULE RX -> MCU USART1_TX (e.g. PA9)
  - MODULE VCC -> 3.3V
  - MODULE GND -> GND
  - M0 -> MCU GPIO (e.g. GPIOB, M0_Pin)
  - M1 -> MCU GPIO (e.g. GPIOB, M1_Pin)
  - AUX -> MCU GPIO input (AUX_GPIO_Port, AUX_Pin)

Important code details found in this repo:

- The library uses `huart1` for E220 module communication. `huart1` BaudRate is set to 9600 in `MX_USART1_UART_Init`.
- `huart2` is used for debug logs and is set to 115200.
- GPIO initialization in `MX_GPIO_Init` enables `GPIOB` and configures `M0_Pin|M1_Pin` as outputs and `AUX_Pin` as input.

Suggested CubeMX / STM32 peripheral settings:

1. Enable USART1 (Asynchronous), BaudRate=9600, 8N1. Assign TX/RX pins to your board (e.g. PA9/PA10).
2. Enable USART2 (Asynchronous) for debugging (115200) and assign pins (e.g. PA2/PA3).
3. Configure M0 and M1 pins as GPIO Output Push-Pull, No Pull, Low speed (they are toggled in code to change module mode).
4. Configure AUX pin as GPIO Input (No Pull) and connect to the module AUX.

How the software uses the pins and flow:

- M0/M1 combinations select module mode (see `E220_SetMode`):
  - M0=0, M1=0 -> MODE_0_NORMAL
  - M0=1, M1=0 -> MODE_1_WOR_TX
  - M0=0, M1=1 -> MODE_2_WOR_RX
  - M0=1, M1=1 -> MODE_3_CONFIG (configuration mode)
- After changing mode pins the code waits for AUX to be HIGH to ensure the module is ready before sending config commands.

Practical wiring checklist:

1. Connect module VCC to 3.3V and GND to GND.
2. Cross-connect UART: module TX -> MCU RX, module RX -> MCU TX.
3. Connect M0 and M1 to MCU GPIOs (defined as `M0_Pin`, `M1_Pin` in generated `main.h`).
4. Connect AUX to an input pin defined as `AUX_Pin`/`AUX_GPIO_Port`.
5. Verify the pin defines in your `main.h` (CubeMX generated) and, if needed, update code or README accordingly.

If you want, I can:

- Insert a simple wiring diagram image (if you provide one) or ASCII art mapping for a specific STM32 board.
- Update the README with exact pin numbers if you paste `main.h` or your CubeMX `.ioc` pin assignments.

---


