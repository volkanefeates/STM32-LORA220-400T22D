# STM32-LORA220-400T22D

Bu proje, EBYTE E220 serisi (ör. LORA220-400T22D) modüllerini STM32 (HAL) ile yapılandırmak ve geçici ayarlarını değiştirmek için hazırlanmış bir örnek kütüphaneyi içerir.

Bu repodaki örnek kodlar şunları yapar:

- Modülün mevcut ayarlarını UART üzerinden okur.
- Kodu içindeki `lora_module.settings` yapısını kullanarak yeni (geçici) ayarları modüle yazar.

# STM32 - EBYTE LORA220-400T22D (E220) — Kullanım & Bağlantı Rehberi

Bu repo, EBYTE E220 serisi (ör. LORA220-400T22D) radyo modülünü STM32 (HAL) ile hızlıca denemek ve modül ayarlarını (geçici/RAM'e yazma) değiştirmek için hazırlanmış bir örnek proje içerir.

İçindekiler

- `Inc/Lora_Settings.h` : Modül ayarları için yapı, enum ve fonksiyon prototipleri
- `Src/Lora_Settings.c` : UART üzerinden modül ile konuşma, okuma/yazma ve kayıt/parça-parça yazma rutinleri
- `Src/main.c` : Kütüphaneyi nasıl kullanacağınızı gösteren örnek uygulama

Hızlı özet

Bu proje ile:

- Modülün mevcut ayarlarını okuyabilirsiniz.
- `lora_module.settings` yapısını doldurup yeni (geçici) ayarları modüle yazabilirsiniz (RAM'e yazma).
- Yazılan ayarları doğrulamak için tekrar okuyabilirsiniz.

Önkoşullar

- STM32CubeIDE / STM32CubeMX veya HAL kullanan benzer bir yapı
- HAL UART sürücüsü proje içinde etkin ve konfigüre edilmiş olmalı (ör. `USART1`)
- Modül 3.3V ile beslenmelidir (5V vermeyin)

Donanım (bağlantı) — tipik

- LORA modül: VCC, GND, TX, RX, M0, M1, AUX
- Örnek STM32 eşlemesi (kendi kartınıza göre değiştirin):
  - MODULE TX -> MCU USART1_RX (ör. PA10)
  - MODULE RX -> MCU USART1_TX (ör. PA9)
  - MODULE VCC -> 3.3V
  - MODULE GND -> GND
  - M0 -> MCU GPIO (ör. GPIOB, `M0_Pin`)
  - M1 -> MCU GPIO (ör. GPIOB, `M1_Pin`)
  - AUX -> MCU GPIO input (ör. `AUX_Pin`)

Not: Projedeki örnek uygulama `huart1` ile modül iletişimi kurar (USART1, 9600) ve `huart2` debug için (115200) kullanılır.

Pin / CubeMX ayar önerileri

1. USART1 (modül iletişimi): Asynchronous, Baud=9600, 8N1; TX/RX pinlerini board'unuza göre atayın (ör. PA9/PA10).
2. USART2 (debug/log): Asynchronous, Baud=115200 (isteğe bağlı).
3. M0 ve M1 pinleri: GPIO Output, Push-Pull, No Pull, Low Speed.
4. AUX pini: GPIO Input, No Pull.
5. Kod üretildikten sonra `main.h` dosyasındaki `M0_Pin`, `M1_Pin`, `AUX_Pin` tanımlarını not edin; README'yi kesin pinlerle güncellemek için bu değerleri paylaşabilirsiniz.

Yazılım kullanımı (kısa)

- Başlatma: `E220_Init(&lora_module, &huart1, M0_GPIO_Port, M0_Pin, M1_GPIO_Port, M1_Pin, AUX_GPIO_Port, AUX_Pin);`
- Mod değiştirme: `E220_SetMode(&lora_module, MODE_3_CONFIG)` gibi fonksiyonlar kullanılabilir. Kod, mod değişikliğinin ardından AUX pininin HIGH olmasını bekler (handshake).
- Ayar okuma: `E220_LoadSettingsFromModule(&lora_module)`
- RAM'e yazma (geçici): `E220_SaveSettingsToRAM(&lora_module)` — bu fonksiyon `dev->settings` içeriğini modüle parça parça gönderir.

„RAM'e yazma" (geçici ayarlar) hakkında kısa bilgi

- Bu proje örneklerinde yapılan yazma işlemi RAM'e (volatile) yazmadır; güç kesilince bu ayarlar kaybolur.
- İngilizce teknik terimler: "Write to RAM", "Temporary settings" veya "Volatile settings".
- Kalıcı (permanent) ayarlar için modülün dokümantasyonunda belirtilen non‑volatile (flash/EEPROM) kaydetme komutunu kullanmanız gerekir — bu proje örneğinde RAM'e yazma (temporary) gösterilmiştir.

Pratik kontrol listesi (ilk deney)

1. Güç: Modüle 3.3V verildiğinden emin olun.
2. UART: MCU TX -> MODULE RX, MCU RX -> MODULE TX şeklinde çapraz bağlantı yapın.
3. M0/M1/AUX: Doğru port/pinlere bağlandıklarını doğrulayın. `main.h` içindeki tanımlarla uyuşmuyorsa README'yi güncelleyin.
4. Eğer `E220_LoadSettingsFromModule` hata veriyorsa: bağlantıları, güç ve UART baud/parity ayarlarını kontrol edin.

Örnek davranış (kodda görülenler)

- `E220_SetMode` fonksiyonu M0/M1 pinlerini ayarlar, 10 ms bekler ve sonra AUX HIGH olana kadar bekleyerek mod geçişini onaylar.
- `E220_ReadRegister` ve `E220_WriteTempRegister` UART üzerinden modül ile paket tabanlı iletişim yapar (komut/cevap). Okuma ve yazma işlemleri belirtilen timeout değerlerine tabidir.

Hata giderme (troubleshooting)

- Güç 3.3V mı? 5V vermeyin.
- UART kabloları çapraz mı bağlandı? (MCU_TX->MOD_RX, MCU_RX->MOD_TX)
- AUX pini beklenen durumda mı? Kod çoğu operasyondan önce AUX HIGH bekler.
- `main.h` içindeki pin tanımları ile fiziksel bağlantılar uyuşuyor mu?

Geliştirme notları ve değiştirilebilir alanlar

- `M0_Pin`, `M1_Pin`, `AUX_Pin` tanımları `main.h` içinde bulunur; kesin pinleri README'ye eklememi isterseniz o dosyayı paylaşın.
- Eğer isterseniz README'yi belirli bir STM32 kartına (Nucleo/Discovery model) göre kişiselleştiririm ve ASCII diyagram eklerim.

English — Quick reference

This repository demonstrates basic usage of the EBYTE E220 family with an STM32 using HAL. It exposes functions to change module modes, read registers, and write temporary settings to RAM.

Contents

- `Inc/Lora_Settings.h`: structures/enums and prototypes
- `Src/Lora_Settings.c`: UART register read/write and settings parsing
- `Src/main.c`: example usage and a small test flow

Quick wiring (typical)

- MODULE TX -> MCU RX (USART1_RX)
- MODULE RX -> MCU TX (USART1_TX)
- MODULE VCC -> 3.3V
- MODULE GND -> GND
- M0/M1 -> MCU GPIO outputs (used to switch mode)
- AUX -> MCU GPIO input (module ready indicator)

CubeMX / STM32 setup (suggested)

1. Enable USART1 for module comms (9600, 8N1).
2. Enable a second UART for debug if needed (115200).
3. Configure M0/M1 as outputs (push-pull) and AUX as input (no pull).

Temporary vs Permanent writes

- The provided library writes settings to RAM (temporary). These are volatile and lost on power cycle. To persist settings across power cycles you must use the module's persistent-save command (not covered here).

If you want help adapting the README to a specific STM32 board or want an ASCII wiring diagram, paste your `main.h` or tell me your board model and I'll update the README accordingly.

License

See the `LICENSE` file in this repository.
