# I2C, SPI ve UART

## 🟢 Giriş Seviyesi

**S: I2C, SPI ve UART arasındaki temel farklar nelerdir?**

| Özellik | UART | SPI | I2C |
|---|---|---|---|
| Hat sayısı | 2 (TX, RX) | 4 (MOSI, MISO, CLK, CS) | 2 (SDA, SCL) |
| Hız | Düşük-Orta | Yüksek | Orta |
| Cihaz sayısı | 2 (point-to-point) | Çoklu (her biri için CS) | Çoklu (adres ile) |
| Senkron mu? | Hayır | Evet | Evet |
| Donanım karmaşıklığı | Düşük | Orta | Düşük |

---

**S: I2C'de pull-up direnci neden gereklidir?**

I2C hatları open-drain'dir. Hiçbir cihaz aktif HIGH süremez, bu nedenle pull-up direnci hattı varsayılan olarak HIGH tutar. Cihazlar hattı sadece LOW'a çekebilir.

---

**S: UART'ta baud rate nedir?**

Saniyede iletilen sembol sayısıdır. İki cihazın baud rate'i eşleşmezse veri bozulur. Yaygın değerler: 9600, 115200.

---

## 🟡 Orta Seviye

**S: I2C'de ACK/NACK ne anlama gelir?**

Her byte iletiminden sonra alıcı:
- **ACK (0):** "aldım, devam et"
- **NACK (1):** "almadım veya dur"

Master NACK alırsa iletişimi durdurur.

---

**S: SPI'da CPOL ve CPHA ne anlama gelir?**

- **CPOL (Clock Polarity):** Saat sinyalinin boştaki seviyesi (0 = LOW, 1 = HIGH)
- **CPHA (Clock Phase):** Verinin hangi kenardan örneklendiği (0 = rising, 1 = falling)

4 farklı SPI modu oluşturur (Mode 0-3). Slave cihazın datasheeti hangi modun kullanılacağını belirtir.

---

**S: I2C clock stretching nedir?**

Slave, hazır olmadığında SCL hattını LOW'da tutarak master'ı bekletir. Master bu süre zarfında veri göndermez.

---

## 🔴 İleri Seviye

**S: I2C bus'ta arbitration nasıl çalışır?**

Birden fazla master aynı anda veri göndermeye çalışırsa:
- Her master kendi gönderdiği biti okur
- Gönderilen 1 ama okunan 0 ise başka bir master baskın gelmiş demektir
- Kaybeden master iletimi bırakır, kazanan devam eder

---

**S: UART'ta overrun hatası ne zaman oluşur?**

RX buffer dolmadan yeni veri gelirse overrun hatası oluşur. Embedded'da bu genellikle DMA veya interrupt latency sorununa işaret eder.

---

**S: Hangi protokolü ne zaman seçersin?**

- **UART:** Debug output, GPS, GSM modül — basit, yaygın
- **SPI:** Hız gerektiren uygulamalar — display, flash, ADC
- **I2C:** Çok sayıda düşük hızlı sensör — IMU, RTC, EEPROM
