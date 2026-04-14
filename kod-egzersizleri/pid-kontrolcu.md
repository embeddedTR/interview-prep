# PID Kontrolcü

## Problem

Motor hız kontrolü veya sıcaklık regülasyonu gibi uygulamalarda kullanılmak üzere basit bir PID kontrolcü implementasyonu yaz.

Aşağıdaki gereksinimleri karşılamalı:
- Periyodik olarak çağrılmalı (sabit örnekleme süresi)
- Integral windup koruması olmalı
- Çıkış sınırlandırılabilmeli

---

## Çözüm

```c
#include <stdint.h>

typedef struct {
    float kp;           // oransal kazanç
    float ki;           // integral kazanç
    float kd;           // türevsel kazanç
    float dt;           // örnekleme süresi (saniye)
    float out_min;      // çıkış alt sınırı
    float out_max;      // çıkış üst sınırı
    float integral;     // birikmiş integral
    float prev_error;   // önceki hata (türev için)
} PID;

void pid_init(PID *pid, float kp, float ki, float kd,
              float dt, float out_min, float out_max) {
    pid->kp         = kp;
    pid->ki         = ki;
    pid->kd         = kd;
    pid->dt         = dt;
    pid->out_min    = out_min;
    pid->out_max    = out_max;
    pid->integral   = 0.0f;
    pid->prev_error = 0.0f;
}

void pid_reset(PID *pid) {
    pid->integral   = 0.0f;
    pid->prev_error = 0.0f;
}

float pid_update(PID *pid, float setpoint, float measurement) {
    float error      = setpoint - measurement;
    float derivative = (error - pid->prev_error) / pid->dt;

    // integral hesapla
    pid->integral += error * pid->dt;

    // çıkışı hesapla
    float output = (pid->kp * error)
                 + (pid->ki * pid->integral)
                 + (pid->kd * derivative);

    // çıkışı sınırla
    if (output > pid->out_max) {
        output = pid->out_max;
        // integral windup koruması: sınırı aştıysa integral'i geri al
        pid->integral -= error * pid->dt;
    } else if (output < pid->out_min) {
        output = pid->out_min;
        pid->integral -= error * pid->dt;
    }

    pid->prev_error = error;
    return output;
}
```

---

## Kullanım

```c
PID motor_pid;

void app_init(void) {
    // Kp=1.0, Ki=0.1, Kd=0.05, dt=10ms, çıkış: 0–100 (PWM duty cycle)
    pid_init(&motor_pid, 1.0f, 0.1f, 0.05f, 0.01f, 0.0f, 100.0f);
}

// TIM interrupt — her 10ms çağrılır
void control_loop(void) {
    float setpoint    = 1500.0f;           // hedef RPM
    float measurement = encoder_get_rpm(); // ölçülen RPM
    float duty        = pid_update(&motor_pid, setpoint, measurement);
    pwm_set_duty(duty);
}
```

---

## Mülakatta Sorulabilecek Ek Sorular

1. Integral windup nedir, neden sorun çıkarır?
2. `dt` sabit olmak zorunda mı? Değilse ne değişmeli?
3. Bu PID'i sıcaklık kontrolü için nasıl ayarlarsın (tune)?
4. Derivative kick nedir, nasıl önlenir?

---

## Cevaplar

**1.** Sistem setpoint'e ulaşamadığında hata birikiyor ve integral çok büyüyor. Sistem setpoint'e geldiğinde bile yüksek integral nedeniyle aşım (overshoot) oluşur. Çıkış sınırına ulaşıldığında integral'i dondurarak önlenir.

**2.** Değil. Değişken `dt` için her çağrıda geçen süre ölçülerek `dt` güncellenebilir. Ancak sabit örnekleme çok daha öngörülebilir davranış sağlar — timer interrupt tercih edilmeli.

**3.** Ziegler-Nichols yöntemi: önce Ki=Kd=0 ile Kp artırılır, sistem salınıma girdiğinde kritik Kp ve periyot ölçülür, buradan Ki ve Kd hesaplanır.

**4.** Setpoint ani değiştiğinde `error` büyük sıçrama yapar, türev terimi çok yüksek çıkış üretir. Çözüm: türevi `error` yerine `measurement` üzerinden al — `derivative = -(measurement - prev_measurement) / dt`.
