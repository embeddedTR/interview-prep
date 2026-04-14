# RTOS Temel Kavramlar

## 🟢 Giriş Seviyesi

**S: RTOS nedir, bare-metal'dan farkı nedir?**

RTOS (Real-Time Operating System), görevlerin belirlenmiş süre kısıtları içinde çalışmasını garanti eden işletim sistemidir.

| | Bare-metal | RTOS |
|---|---|---|
| Görev yönetimi | Super-loop | Preemptive/cooperative scheduler |
| Zamanlama | Manuel | Otomatik |
| Karmaşıklık | Düşük | Yüksek |
| Deterministik mi? | Zor | Evet |

---

**S: Task (görev) nedir?**

Bağımsız çalışan, kendi stack'i olan fonksiyondur. RTOS her task'ı sanki tek başına çalışıyormuş gibi görür.

```c
void vMyTask(void *pvParameters) {
    for (;;) {
        // görev kodu
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

---

**S: Task önceliği (priority) ne anlama gelir?**

Hangi task'ın önce çalışacağını belirler. Yüksek öncelikli task, düşük öncelikli task'ı preempt eder.

---

## 🟡 Orta Seviye

**S: Preemptive ve cooperative scheduling farkı nedir?**

- **Preemptive:** Scheduler istediği zaman task'ı durdurup başkasını çalıştırabilir
- **Cooperative:** Task kendisi `yield` çağırıncaya kadar CPU'yu bırakmaz

FreeRTOS varsayılan olarak preemptive'dir.

---

**S: Tick nedir?**

RTOS'un periyodik zamanlayıcı interrupt'ıdır. Her tick'te scheduler hangi task'ın çalışacağına karar verir. FreeRTOS'ta varsayılan 1ms (1000 Hz).

---

**S: `vTaskDelay` ile `vTaskDelayUntil` farkı nedir?**

- `vTaskDelay`: Çağrıldığı andan itibaren bekler — sürüklenme (drift) oluşabilir
- `vTaskDelayUntil`: Belirtilen mutlak zamana kadar bekler — periyodik görevler için doğru seçim

```c
// Periyodik görev için doğru kullanım:
TickType_t xLastWakeTime = xTaskGetTickCount();
for (;;) {
    vTaskDelayUntil(&xLastWakeTime, pdMS_TO_TICKS(10));
    // görev kodu
}
```

---

## 🔴 İleri Seviye

**S: Stack boyutunu nasıl belirlersin?**

1. `uxTaskGetStackHighWaterMark()` ile çalışma zamanında izle
2. En kötü durumu simüle et (tüm fonksiyon derinlikleri, interrupt preemption)
3. Güvenlik payı ekle (genellikle %20-30)

---

**S: RTOS ne zaman kullanılmamalıdır?**

- Çok sıkı real-time gereksinimleri olan ve deterministik interrupt latency gereken sistemlerde (bare-metal daha öngörülebilir olabilir)
- Flash/RAM çok kısıtlıysa (FreeRTOS ~5-10KB Flash gerektirir)
- Tek bir görev varsa — RTOS overhead gereksizdir
