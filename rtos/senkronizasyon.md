# Senkronizasyon

## 🟢 Giriş Seviyesi

**S: Neden senkronizasyona ihtiyaç duyulur?**

Birden fazla task veya interrupt aynı kaynağa (değişken, buffer, peripheral) erişiyorsa race condition oluşabilir. Senkronizasyon bu erişimi düzenler.

---

**S: Semaphore nedir?**

Sayaç tabanlı bir sinyalleşme mekanizmasıdır.

- **Binary semaphore:** 0 veya 1. Bir task sinyal bekler, diğeri sinyal verir.
- **Counting semaphore:** N'e kadar sayar. Kaynak havuzu yönetiminde kullanılır.

```c
// ISR → Task haberleşmesi
void UART_IRQHandler(void) {
    xSemaphoreGiveFromISR(xSem, &xHigherPriorityTaskWoken);
}

void vTask(void *p) {
    for (;;) {
        xSemaphoreTake(xSem, portMAX_DELAY);
        // veriyi işle
    }
}
```

---

## 🟡 Orta Seviye

**S: Mutex ile binary semaphore farkı nedir?**

| | Binary Semaphore | Mutex |
|---|---|---|
| Sahiplik | Yok | Var (alan task bırakmalı) |
| Priority inheritance | Yok | Var (FreeRTOS'ta) |
| Kullanım | Task/ISR sinyalleşme | Mutual exclusion |

Mutex'i **yalnızca** kaynak korumak için, semaphore'u **sinyalleşme** için kullan.

---

**S: Deadlock nedir, nasıl oluşur?**

İki task birbirinin tuttuğu kaynağı beklediğinde oluşur, ikisi de ilerleyemez.

```
Task A → Mutex1 aldı → Mutex2 bekliyor
Task B → Mutex2 aldı → Mutex1 bekliyor
→ DEADLOCK
```

**Önlem:**
- Kaynakları her zaman aynı sırada al
- Timeout kullan (`xSemaphoreTake(xMutex, timeout)`)
- Kaynak sayısını minimumda tut

---

**S: Priority inversion nedir?**

Düşük öncelikli task bir mutex tutarken, yüksek öncelikli task o mutex'i bekliyorsa, orta öncelikli task CPU'yu alır. Yüksek öncelikli task fiilen en son çalışır.

**Çözüm:** Priority inheritance — mutex tutan task, bekleyen en yüksek önceliğe geçici olarak yükseltilir.

---

## 🔴 İleri Seviye

**S: Queue ile semaphore arasındaki farkı ne zaman önemlidir?**

Queue hem veri taşır hem senkronizasyon sağlar. Semaphore sadece sinyal verir, veri taşımaz.

```c
// Semaphore: "bir şey oldu" sinyali
xSemaphoreGive(xSem);

// Queue: "şu veriyi işle" mesajı
uint8_t data = 42;
xQueueSend(xQueue, &data, 0);
```

ISR'dan task'a veri gönderirken queue tercih edilmeli.

---

**S: ISR içinde `FromISR` varyantları neden kullanılmalıdır?**

Normal FreeRTOS API'leri scheduler'ı çağırabilir veya task context varsayar. ISR context'te bu güvenli değildir. `FromISR` varyantları ISR-safe'tir ve `xHigherPriorityTaskWoken` parametresiyle context switch tetikleyebilir.

```c
void TIM2_IRQHandler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    xSemaphoreGiveFromISR(xSem, &xHigherPriorityTaskWoken);
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```
