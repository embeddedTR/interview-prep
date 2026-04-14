# Bellek Tipleri

## 🟢 Giriş Seviyesi

**S: Flash, RAM ve EEPROM arasındaki fark nedir?**

| | Flash | RAM | EEPROM |
|---|---|---|---|
| Uçucu mu? | Hayır | Evet | Hayır |
| Yazma hızı | Yavaş | Hızlı | Çok yavaş |
| Silinme | Blok bazlı | - | Byte bazlı |
| Kullanım | Program kodu | Çalışma zamanı | Kalıcı veri |

---

**S: Stack ve Heap nedir?**

- **Stack:** Fonksiyon çağrıları, yerel değişkenler. LIFO yapısı. Boyutu derleme zamanında belirlenir.
- **Heap:** Dinamik bellek (`malloc`). Boyutu çalışma zamanında değişir.

Embedded sistemlerde heap kullanımı genellikle önerilmez — fragmentation ve belirsiz davranış riski vardır.

---

## 🟡 Orta Seviye

**S: Stack overflow nasıl tespit edilir?**

- Stack canary: Stack sonuna bilinen bir değer yazılır, periyodik olarak kontrol edilir
- MPU: Stack bölgesine yazma girişiminde fault üretilir
- RTOS'larda `uxTaskGetStackHighWaterMark()` ile kalan stack izlenir

---

**S: `.bss` ve `.data` segment farkı nedir?**

- `.data`: Başlangıç değeri olan global/statik değişkenler. Flash'tan RAM'e kopyalanır.
- `.bss`: Başlangıç değeri sıfır olan veya başlangıç değeri olmayan değişkenler. Startup kodu sıfırlar.

```c
int x = 5;    // .data
int y;        // .bss
```

---

**S: `const` değişkenler nerede tutulur?**

Flash'ta (ROM'da). `const` global değişkenler `.rodata` segmentine gider, RAM kullanmaz.

```c
const uint8_t table[256] = { ... };  // Flash'ta, RAM harcamaz
```

---

## 🔴 İleri Seviye

**S: Flash wear leveling nedir?**

Flash hücreleri sınırlı sayıda yazılabilir (NOR: ~100K, NAND: ~10K yazma). Wear leveling, yazma işlemlerini tüm sektörlere eşit dağıtarak ömrü uzatır. EEPROM emulasyonu yapan kütüphaneler bunu otomatik yönetir.

---

**S: Linker script'te `MEMORY` ve `SECTIONS` ne işe yarar?**

```ld
MEMORY {
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 512K
    RAM   (rwx) : ORIGIN = 0x20000000, LENGTH = 128K
}

SECTIONS {
    .text : { *(.text) } > FLASH
    .data : { *(.data) } > RAM AT > FLASH
    .bss  : { *(.bss)  } > RAM
}
```

`MEMORY` fiziksel bölgeleri tanımlar. `SECTIONS` hangi kod/verinin nereye yerleşeceğini belirler.
