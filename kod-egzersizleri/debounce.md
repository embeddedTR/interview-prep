# Debounce (Titreşim Giderme)

## Problem

Mekanik buton/anahtar basıldığında kontaklar kısa süre titreşir ve MCU bunu çok sayıda basış olarak algılar. Bu titreşimi yazılımla filtrelemek gerekir.

---

## Soru

Timer interrupt'ı kullanan bir software debounce implementasyonu yaz. Her 10ms'de bir çağrılacak şekilde tasarla.

---

## Çözüm 1 — Basit Gecikme Tabanlı

```c
#define DEBOUNCE_MS     50
#define SAMPLE_PERIOD   10   // her 10ms çağrılır

typedef struct {
    uint8_t  pin_state;
    uint8_t  stable_state;
    uint16_t counter;
} Button;

void button_update(Button *btn, uint8_t current_pin) {
    if (current_pin != btn->pin_state) {
        btn->pin_state = current_pin;
        btn->counter = 0;
    } else {
        if (btn->counter < (DEBOUNCE_MS / SAMPLE_PERIOD)) {
            btn->counter++;
        } else {
            btn->stable_state = btn->pin_state;
        }
    }
}
```

---

## Çözüm 2 — Integrator Tabanlı (Daha Robust)

```c
#define DEBOUNCE_MAX 5

typedef struct {
    uint8_t integrator;
    uint8_t output;
} Button;

void button_update(Button *btn, uint8_t raw_input) {
    if (raw_input == 0) {
        if (btn->integrator > 0)
            btn->integrator--;
    } else {
        if (btn->integrator < DEBOUNCE_MAX)
            btn->integrator++;
    }

    if (btn->integrator == 0)
        btn->output = 0;
    else if (btn->integrator == DEBOUNCE_MAX)
        btn->output = 1;
    // arada: önceki stabil değerde kal
}
```

---

## Kullanım

```c
Button btn = {0};

// TIM6 ISR — 10ms periyodik
void TIM6_IRQHandler(void) {
    uint8_t raw = HAL_GPIO_ReadPin(BTN_GPIO_Port, BTN_Pin);
    button_update(&btn, raw);
}

// Main loop
if (btn.output == 1) {
    // buton basılı
}
```

---

## Mülakatta Sorulabilecek Ek Sorular

1. Bu iki yöntem arasındaki pratik fark nedir?
2. Hardware debounce ile yazılım debounce'u ne zaman tercih edersin?
3. RTOS'ta bu tasarım nasıl değişir?

---

## Cevaplar

**1.** Basit gecikme tabanlı yöntem "N ms boyunca aynı kaldıysa stabil" der. Integrator yöntemi ise gürültüye karşı daha toleranslı — kısa ani değişimleri filtreler, ancak daha uzun kararlı değişiklikleri kabul eder.

**2.** Yüksek hız gereken veya çok sayıda buton varsa RC filtre + Schmitt trigger (hardware) daha temiz. Yazılım debounce ek devre gerektirmez, parametresi runtime değiştirilebilir.

**3.** RTOS'ta timer callback veya ayrı düşük öncelikli task kullanılır. Stabil durum değişiminde event group veya queue ile ilgili task uyandırılır.
