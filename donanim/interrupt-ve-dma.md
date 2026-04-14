# Interrupt ve DMA

## 🟢 Giriş Seviyesi

**S: Interrupt nedir?**

İşlemciye "şu an yaptığını bırak, şuna bak" diyen donanım veya yazılım sinyalidir. İşlemci mevcut görevi durdurur, ISR'ı (Interrupt Service Routine) çalıştırır, geri döner.

---

**S: ISR içinde neler yapılmalı, neler yapılmamalı?**

**Yapılmalı:**
- Kısa ve hızlı işlemler
- Flag set etmek
- Buffer'a veri yazmak

**Yapılmamalı:**
- Uzun süren döngüler
- `printf`, `malloc` gibi reentrancy sorunu çıkarabilecek fonksiyonlar
- Blocking bekleme

---

**S: DMA nedir?**

Direct Memory Access — veriyi CPU müdahalesi olmadan donanım ile bellek arasında taşıyan kontrolcüdür. CPU serbest kalır, başka işler yapabilir.

---

## 🟡 Orta Seviye

**S: Interrupt latency nedir, neyi etkiler?**

Interrupt sinyali ile ISR'ın ilk instruction'ının çalışması arasındaki süredir. Yüksek öncelikli görevler için kritiktir. Etkileyen faktörler:

- İşlemci mimarisi (Cortex-M3: ~12 cycle)
- Daha yüksek öncelikli interrupt'ların varlığı
- Cache miss
- Flash wait state

---

**S: Nested interrupt nedir?**

Bir ISR çalışırken daha yüksek öncelikli bir interrupt gelirse, mevcut ISR duraklatılır ve yeni ISR çalışır. Cortex-M'de NVIC bunu otomatik yönetir.

---

**S: DMA transfer tamamlandığında nasıl haberdar olunur?**

DMA tamamlanma interrupt'ı (Transfer Complete) kullanılır. ISR içinde flag kontrol edilir ve gerekli işlem yapılır.

---

## 🔴 İleri Seviye

**S: DMA kullanırken cache coherency sorunu neden oluşur?**

Cache'li sistemlerde DMA doğrudan RAM'e yazar fakat CPU cache'den okuyabilir. CPU güncel veriyi göremez.

Çözümler:
- Cache invalidate (CPU okumadan önce)
- Non-cacheable bellek bölgesi kullanımı
- MPU ile bölge ayarı

---

**S: Interrupt priority inversion nedir?**

Düşük öncelikli görev bir kaynağı tutarken, yüksek öncelikli görev o kaynağı bekliyorsa yüksek öncelikli görev fiilen bloke olur. RTOS'larda mutex priority inheritance ile çözülür.

---

**S: `__disable_irq()` ve `__enable_irq()` ne zaman kullanılır?**

Atomik işlemler için kritik bölge oluştururken. Paylaşılan veriye hem main loop hem interrupt'tan erişiliyorsa:

```c
__disable_irq();
shared_var++;   // atomik erişim
__enable_irq();
```

Uzun süre disable bırakmak interrupt latency'yi artırır, dikkatli kullanılmalı.
