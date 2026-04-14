# Hareketli Ortalama Filtresi

## Problem

ADC'den gelen gürültülü sensör verisini yumuşatmak için sabit boyutlu bir pencere kullanan hareketli ortalama (moving average) filtresi yaz.

Gereksinimler:
- Dinamik bellek (`malloc`) kullanma
- Her yeni ölçümde O(1) karmaşıklıkta çalışsın
- Farklı pencere boyutları için kullanılabilsin

---

## Çözüm

```c
#include <stdint.h>
#include <string.h>

#define MA_MAX_WINDOW 32

typedef struct {
    float    buf[MA_MAX_WINDOW];
    uint8_t  size;      // pencere boyutu
    uint8_t  index;     // sonraki yazılacak konum
    uint8_t  count;     // doldurulan eleman sayısı
    float    sum;       // toplam (her seferinde yeniden hesaplamamak için)
} MovingAverage;

void ma_init(MovingAverage *ma, uint8_t window_size) {
    if (window_size > MA_MAX_WINDOW) window_size = MA_MAX_WINDOW;
    ma->size  = window_size;
    ma->index = 0;
    ma->count = 0;
    ma->sum   = 0.0f;
    memset(ma->buf, 0, sizeof(ma->buf));
}

float ma_update(MovingAverage *ma, float new_sample) {
    // çıkan değeri toplamdan çıkar
    ma->sum -= ma->buf[ma->index];

    // yeni değeri yaz
    ma->buf[ma->index] = new_sample;
    ma->sum += new_sample;

    // indeksi ilerlet (dairesel)
    ma->index = (ma->index + 1) % ma->size;

    // pencere dolana kadar gerçek eleman sayısını kullan
    if (ma->count < ma->size) ma->count++;

    return ma->sum / ma->count;
}
```

---

## Kullanım

```c
MovingAverage temp_filter;

void app_init(void) {
    ma_init(&temp_filter, 16);  // 16 örneklik pencere
}

// Her ADC ölçümünde:
void adc_callback(uint16_t raw) {
    float voltage    = raw * (3.3f / 4095.0f);
    float filtered   = ma_update(&temp_filter, voltage);
    display_voltage(filtered);
}
```

---

## Pencere Boyutunun Etkisi

```
Küçük pencere (4):  Hızlı tepki, az gürültü azaltma
Büyük pencere (32): Yavaş tepki, çok gürültü azaltma

Ham veri:     25.1, 24.8, 25.9, 24.5, 25.2, 25.0 ...
4 pencereli:  25.1, 24.9, 25.3, 25.1, 25.1, 25.1 ...
16 pencereli: 25.1, 25.0, 25.0, 25.0, 25.0, 25.0 ...
```

---

## Mülakatta Sorulabilecek Ek Sorular

1. Bu implementasyonun zayıf noktası nedir?
2. Hareketli ortalama yerine ne zaman başka bir filtre tercih edilir?
3. `sum` değişkeni neden tutuldu, her seferinde yeniden hesaplayabilirdin?

---

## Cevaplar

**1.** Kayan nokta (float) toplama işlemleri zamanla birikimli hata (floating point drift) oluşturabilir. Uzun süreli çalışan sistemlerde `sum` periyodik olarak yeniden hesaplanabilir. Ayrıca `MA_MAX_WINDOW` sabit — çalışma zamanında boyut değiştirilemiyor.

**2.** Ani geçişlerin korunması gerekiyorsa (örn. darbe tespiti) hareketli ortalama sinyali gereğinden fazla yumuşatır. Bu durumda medyan filtresi veya üstel hareketli ortalama (EMA) daha uygun olabilir.

**3.** Her `ma_update` çağrısında tüm pencereyi toplayarak ortalama almak O(N) karmaşıklık gerektirir. `sum` tutulduğunda sadece iki toplama işlemi yapılır — O(1) karmaşıklık.
