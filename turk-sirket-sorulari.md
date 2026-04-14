# Sektöre Göre Mülakat Soruları

Türkiye'deki embedded pozisyonlarında sektöre göre öne çıkan mülakat soruları ve beklenen cevap derinliği.

---

## Savunma Elektroniği

Radar, haberleşme, elektronik harp sistemleri. RTOS ve güvenilirlik ön planda.

**S: FreeRTOS'ta task önceliği nasıl belirlenir? Priority inversion'ı nasıl önlersin?**

Beklenen: Sadece kavramı bilmek değil, FreeRTOS'ta `configUSE_MUTEXES` ve priority inheritance mekanizmasını açıklayabilmek.

---

**S: Watchdog timer'ı anlatın. IWDG ile WWDG farkı nedir?**

```
IWDG (Independent Watchdog):
- LSI saatiyle çalışır, ana clock'tan bağımsız
- Yalnızca timeout tabanlı (belirli sürede beslenmezse reset)
- Yazılım tarafından durdurulamaz

WWDG (Window Watchdog):
- APB clock'a bağlı
- Hem erken hem geç besleme hatayı tetikler
- Belirli bir zaman penceresinde beslenmeli
- Zamanlama hassasiyeti gereken sistemlerde tercih edilir
```

---

**S: CRC hesaplaması nedir, firmware doğrulamada nasıl kullanılır?**

Beklenen: CRC32 polinomu, STM32'de CRC peripheral kullanımı, bootloader'da firmware bütünlüğü kontrolü.

---

**S: Stack overflow'u runtime'da nasıl tespit edersiniz?**

Beklenen: Stack canary pattern, MPU ile stack koruma, FreeRTOS'ta `uxTaskGetStackHighWaterMark()`.

---

## Savunma Yazılımı

Komuta kontrol sistemleri, simülasyon. Kod kalitesi ve standartlar önemli.

**S: MISRA-C nedir, neden kullanılır?**

Beklenen: Safety-critical sistemlerde deterministik davranış için C'nin tehlikeli özelliklerini kısıtlayan kural seti olduğunu bilmek. Dinamik bellek yasağı, özyineleme yasağı gibi kuralları örnekleyebilmek.

---

**S: Birim test (unit test) embedded sistemde nasıl yapılır?**

Beklenen: Unity/CMock gibi framework'leri bilmek, donanım bağımlılığını mock'lamak, CI ortamında çalıştırmak.

---

**S: `volatile` anahtar kelimesini ne zaman kullanırsınız? Kullanmayı unutursanız ne olur?**

Beklenen: ISR ile paylaşılan değişken, donanım register'ı, optimizasyon sorunu — somut bir örnek üzerinden açıklayabilmek.

---

**S: Deadlock senaryosu yazın ve çözümünü gösterin.**

```c
// Deadlock örneği:
// Task A: önce mutex1, sonra mutex2 alıyor
// Task B: önce mutex2, sonra mutex1 alıyor

// Çözüm: Her zaman aynı sırada al
// Task A ve Task B: önce mutex1, sonra mutex2
```

---

## Tüketici Elektroniği

Akıllı TV, beyaz eşya, küçük ev aletleri. Linux ve MCU karışık ekipler.

**S: Linux'ta device driver ile bare-metal driver arasındaki fark nedir?**

Beklenen: Kernel space / user space ayrımı, device tree, `probe` fonksiyonu, `read`/`write` file operations.

---

**S: I2C'de clock stretching nedir? Hangi durumlarda sorun çıkarır?**

Beklenen: Slave'in SCL'i LOW tutarak master'ı bekletttiğini, bazı master implementasyonlarının bunu desteklemediğini bilmek.

---

**S: Bir ürün fabrikadan çıkarken nasıl test edilir? Yazılım açısından ne düşünürsünüz?**

Beklenen: Production test firmware, UART üzerinden test komutları, self-test rutinleri, kalibrasyon değerlerinin flash'a yazılması.

---

**S: OTA (Over-The-Air) güncelleme nasıl tasarlanır?**

```
Temel bileşenler:
1. Dual bank flash (A/B partition)
2. Bootloader: hangi bank'tan boot edileceğini belirler
3. Uygulama: güncellemeyi indirir, pasif bank'a yazar
4. CRC / imza doğrulama
5. Başarılı boot sonrası aktif bank güncellenir
```

---

## IoT ve Akıllı Ev

Bağlantılı cihazlar, sensör ağları. Wireless protokoller ve güç optimizasyonu önemli.

**S: Wi-Fi ve BLE arasındaki farklar nelerdir? Hangi uygulamada hangisini seçersiniz?**

| | Wi-Fi | BLE |
|---|---|---|
| Menzil | ~50m | ~10m |
| Güç tüketimi | Yüksek | Çok düşük |
| Veri hızı | Yüksek | Düşük |
| Kullanım | Video, büyük veri | Sensör, kontrol |

---

**S: Bir sensör düğümünü bataryayla 2 yıl çalıştırmak istiyorsunuz. Nasıl tasarlarsınız?**

Beklenen: Duty cycle hesabı, sleep modları, periyodik uyandırma, peripheral'ların uykuda kapatılması, güç bütçesi tablosu.

---

**S: MQTT protokolü nedir, embedded cihazda nasıl kullanılır?**

Beklenen: Pub/sub modeli, broker, QoS seviyeleri, LWT (Last Will and Testament), MQTT over TLS.

---

## Gerçek Zamanlı ve Kritik Sistemler

Uçak, roket, endüstriyel kontrol. Yüksek güvenilirlik ve determinizm zorunlu.

**S: Hard real-time ile soft real-time arasındaki fark nedir?**

```
Hard real-time: Deadline kaçırılması sistem arızasıdır.
Örnek: Uçuş kontrol sistemi, endüstriyel pres kontrolü

Soft real-time: Deadline kaçırılması performans düşüşüdür.
Örnek: Video oynatma, kullanıcı arayüzü
```

---

**S: Interrupt latency'i etkileyen faktörler nelerdir ve nasıl minimize edilir?**

Beklenen: İşlemci pipeline durumu, flash wait state, daha yüksek öncelikli interrupt'ların varlığı, ISR kodu uzunluğu — her birini minimize etme yollarını bilmek.

---

**S: Fonksiyonel güvenlik (IEC 61508 / ISO 26262) biliyor musunuz?**

Beklenen: SIL/ASIL kavramları, redundancy, fault detection, safety case — kavramsal düzeyde bilgi yeterlidir, detay istenmez.
