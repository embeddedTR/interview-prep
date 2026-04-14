# Volatile ve Const

## 🟢 Giriş Seviyesi

**S: `volatile` ne işe yarar?**

Derleyiciye bu değişkeni optimize etmemesini söyler. Değer her an donanım, interrupt veya başka bir thread tarafından değişebilir.

```c
volatile uint8_t flag = 0;

// Interrupt içinde:
void EXTI0_IRQHandler(void) {
    flag = 1;
}

// Main loop içinde:
while (!flag);  // volatile olmasaydı derleyici bunu sonsuz döngüye optimize edebilirdi
```

---

**S: `const` ne işe yarar?**

Değişkenin değerinin değiştirilemeyeceğini belirtir. Derleyici bunu ROM'a alabileceği için RAM tasarrufu sağlar.

```c
const uint8_t MAX_RETRY = 3;
```

---

## 🟡 Orta Seviye

**S: `volatile` olmayan bir değişkene neden derleyici yanlış optimizasyon yapabilir?**

Derleyici "bu değişken döngü içinde değişmiyor" diye düşünüp değeri register'da saklayabilir. Donanım register'ı veya interrupt'tan gelen değişikliği göremez.

```c
// volatile olmadan:
uint8_t status = PERIPH->SR;
while (status == 0);  // derleyici status'u bir kez okur, sonsuza döner

// volatile ile:
volatile uint8_t status;
while ((status = PERIPH->SR) == 0);  // her iterasyonda yeniden okunur
```

---

**S: Bir değişken hem `volatile` hem `const` olabilir mi?**

Evet. Sadece okunabilen ama her an donanım tarafından değişebilen register'lar için kullanılır.

```c
const volatile uint32_t *RO_REG = (const volatile uint32_t *)0x40000000;
// Yazma yapılamaz, ama her okumada güncel değer alınır
```

---

## 🔴 İleri Seviye

**S: `volatile` bir değişkende atomik erişim garanti edilir mi?**

Hayır. `volatile` sadece optimizasyonu engeller, atomikliği garanti etmez. 32-bit olmayan erişimlerde veya multi-core sistemlerde ek önlem gerekir.

```c
volatile uint64_t timestamp;  // 32-bit CPU'da iki ayrı bus erişimi olabilir
// Interrupt tam ortada gelirse tutarsız değer okunur
// Çözüm: kritik bölge veya atomik tip kullanımı
```

---

**S: `volatile` ile `__IO` arasındaki fark nedir?**

`__IO`, CMSIS'in `volatile` için tanımladığı macro'dur. İkisi aynı anlama gelir.

```c
#define __IO volatile
__IO uint32_t *reg = (uint32_t *)0x40021000;
```
