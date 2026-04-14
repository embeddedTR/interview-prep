# Pointerlar

## 🟢 Giriş Seviyesi

**S: Pointer nedir?**

Bir değişkenin bellek adresini tutan değişkendir.

```c
int x = 10;
int *ptr = &x;  // ptr, x'in adresini tutar
printf("%d", *ptr);  // 10
```

---

**S: `*ptr` ile `ptr` arasındaki fark nedir?**

- `ptr` — adresin kendisi
- `*ptr` — o adresteki değer (dereference)

---

**S: NULL pointer nedir, neden önemlidir?**

Hiçbir geçerli adresi göstermeyen pointer'dır. Embedded sistemlerde NULL pointer dereference tanımsız davranışa (undefined behavior) yol açar ve genellikle hard fault üretir.

```c
int *ptr = NULL;
*ptr = 5;  // HATALI — hard fault
```

---

## 🟡 Orta Seviye

**S: `const int *ptr` ile `int * const ptr` arasındaki fark nedir?**

```c
const int *ptr;      // ptr'nin gösterdiği değer değiştirilemez, ptr değişebilir
int * const ptr;     // ptr değiştirilemez, gösterdiği değer değişebilir
const int * const ptr; // ikisi de değiştirilemez
```

Embedded'da register adresleri için `volatile uint32_t * const` yaygın kullanılır.

---

**S: Pointer aritmetiği nasıl çalışır?**

Pointer bir artırıldığında, gösterdiği tipin boyutu kadar ilerler.

```c
int arr[3] = {10, 20, 30};
int *ptr = arr;
ptr++;          // 4 byte ileri (int boyutu)
printf("%d", *ptr);  // 20
```

---

**S: Fonksiyon pointer'ı nedir? Nerede kullanılır?**

Bir fonksiyonun adresini tutan pointer'dır. Embedded'da callback mekanizması ve jump table için yaygın kullanılır.

```c
void led_on(void)  { /* ... */ }
void led_off(void) { /* ... */ }

void (*led_func)(void) = led_on;
led_func();  // led_on çağrılır
```

---

## 🔴 İleri Seviye

**S: Volatile pointer ile normal pointer farkı nedir?**

`volatile` derleyiciye "bu değer her an dışarıdan değişebilir, optimize etme" der. Donanım register'larına erişimde zorunludur.

```c
volatile uint32_t *reg = (volatile uint32_t *)0x40021000;
*reg = 0x01;  // derleyici bu satırı optimize edemez
```

---

**S: Pointer'ı cast ederken dikkat edilmesi gerekenler nelerdir?**

- Alignment: bazı mimarilerde hizasız erişim hard fault üretir
- Strict aliasing: farklı tiplere cast `undefined behavior` olabilir
- `char *` veya `uint8_t *` ile byte bazlı erişim güvenlidir

```c
uint32_t val = 0x12345678;
uint8_t *bytes = (uint8_t *)&val;  // güvenli
```
