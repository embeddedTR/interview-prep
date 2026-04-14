# Bitwise İşlemler

## 🟢 Giriş Seviyesi

**S: Temel bitwise operatörler nelerdir?**

| Operatör | Anlamı | Örnek |
|---|---|---|
| `&` | AND | `0b1100 & 0b1010 = 0b1000` |
| `\|` | OR | `0b1100 \| 0b1010 = 0b1110` |
| `^` | XOR | `0b1100 ^ 0b1010 = 0b0110` |
| `~` | NOT | `~0b1100 = 0b0011` |
| `<<` | Sola kaydır | `0b0001 << 3 = 0b1000` |
| `>>` | Sağa kaydır | `0b1000 >> 3 = 0b0001` |

---

**S: Bir biti nasıl set edersin?**

```c
REG |= (1 << BIT_POS);
```

---

**S: Bir biti nasıl temizlersin (clear)?**

```c
REG &= ~(1 << BIT_POS);
```

---

**S: Bir bitin durumunu nasıl kontrol edersin?**

```c
if (REG & (1 << BIT_POS)) {
    // bit set
}
```

---

## 🟡 Orta Seviye

**S: Bir biti nasıl toggle edersin?**

```c
REG ^= (1 << BIT_POS);
```

---

**S: Birden fazla biti aynı anda set/clear etmek?**

```c
// 2. ve 4. bitleri set et
REG |= (1 << 2) | (1 << 4);

// 2. ve 4. bitleri temizle
REG &= ~((1 << 2) | (1 << 4));
```

---

**S: Mask nedir, neden kullanılır?**

Belirli bitleri korurken diğerlerini işlemek için kullanılır.

```c
#define MODE_MASK  0x03   // son 2 bit
#define MODE_SHIFT 4

// 4. ve 5. bitleri yaz
REG = (REG & ~(MODE_MASK << MODE_SHIFT)) | (value << MODE_SHIFT);
```

---

## 🔴 İleri Seviye

**S: `x & (x - 1)` ne yapar?**

x'in en düşük set bitini temizler. x'in 2'nin kuvveti olup olmadığını kontrol etmek için kullanılır.

```c
int is_power_of_two(uint32_t x) {
    return (x != 0) && ((x & (x - 1)) == 0);
}
```

---

**S: Sağa kaydırma işaretsiz mi, işaretli mi?**

- `unsigned` tiplerde `>>` her zaman sıfır doldurur (logical shift)
- `signed` tiplerde derleyiciye bağlıdır — taşınabilir kod için `unsigned` kullan

```c
int8_t a = -8;   // 0b11111000
a >> 1;          // undefined behavior (implementation-defined)

uint8_t b = 0b11111000;
b >> 1;          // 0b01111100 — güvenli
```

---

**S: Bir register'ın belirli bit alanını (bit field) okumak/yazmak?**

```c
#define CR_PRESC_MASK  0x07
#define CR_PRESC_SHIFT 8

// Okuma
uint32_t presc = (CR & (CR_PRESC_MASK << CR_PRESC_SHIFT)) >> CR_PRESC_SHIFT;

// Yazma
CR = (CR & ~(CR_PRESC_MASK << CR_PRESC_SHIFT)) | ((value & CR_PRESC_MASK) << CR_PRESC_SHIFT);
```
