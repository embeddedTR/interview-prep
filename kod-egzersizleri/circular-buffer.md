# Circular Buffer (Halka Tamponu)

## Soru

Interrupt-safe, sabit boyutlu bir circular buffer yaz. Aşağıdaki operasyonları desteklemeli:

- `cb_push()` — veri ekle
- `cb_pop()` — veri al
- `cb_is_empty()` — boş mu?
- `cb_is_full()` — dolu mu?

---

## Çözüm

```c
#include <stdint.h>
#include <stdbool.h>

#define CB_SIZE 16  // 2'nin kuvveti olması önerilir

typedef struct {
    uint8_t  buf[CB_SIZE];
    uint16_t head;
    uint16_t tail;
    uint16_t count;
} CircularBuffer;

void cb_init(CircularBuffer *cb) {
    cb->head  = 0;
    cb->tail  = 0;
    cb->count = 0;
}

bool cb_is_empty(const CircularBuffer *cb) {
    return cb->count == 0;
}

bool cb_is_full(const CircularBuffer *cb) {
    return cb->count == CB_SIZE;
}

bool cb_push(CircularBuffer *cb, uint8_t data) {
    if (cb_is_full(cb)) return false;

    cb->buf[cb->head] = data;
    cb->head = (cb->head + 1) % CB_SIZE;
    cb->count++;
    return true;
}

bool cb_pop(CircularBuffer *cb, uint8_t *data) {
    if (cb_is_empty(cb)) return false;

    *data = cb->buf[cb->tail];
    cb->tail = (cb->tail + 1) % CB_SIZE;
    cb->count--;
    return true;
}
```

---

## Kullanım

```c
CircularBuffer uart_rx;
cb_init(&uart_rx);

// ISR içinde:
void UART_IRQHandler(void) {
    uint8_t byte = UART->DR;
    cb_push(&uart_rx, byte);
}

// Main loop içinde:
uint8_t data;
if (cb_pop(&uart_rx, &data)) {
    process(data);
}
```

---

## Mülakatta Sorulabilecek Ek Sorular

1. Bu implementasyon interrupt-safe mi? Neden, neden değil?
2. `count` yerine sadece `head == tail` kontrolü neden yetmez?
3. Boyutu 2'nin kuvveti yapmanın `%` operatörü yerine ne avantajı var?
4. Bu yapıyı birden fazla producer/consumer için nasıl güvenli hale getirirsin?

---

## Cevaplar

**1.** Hayır, tam anlamıyla değil. `cb_push` ve `cb_pop` atomik değil. `count`, `head` veya `tail` güncellenirken interrupt gelebilir. Güvenli hale getirmek için kritik bölge gerekli (`__disable_irq()` / `__enable_irq()` veya RTOS mutex).

**2.** `head == tail` hem boş hem dolu durumunu temsil eder. Ayırt etmek için ya `count` tutulur ya da bir slot boş bırakılır.

**3.** `% CB_SIZE` yerine `& (CB_SIZE - 1)` kullanılabilir — tek instruction, daha hızlı. Örn: `head = (head + 1) & (CB_SIZE - 1);`

**4.** Single-producer single-consumer (SPSC) durumunda memory barrier + atomic operasyonlarla lock-free yapılabilir. Çoklu üretici/tüketici için mutex zorunlu.
